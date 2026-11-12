# Prompts d'Optimisation pour MPD-Client-v2

## 🎯 Prompt Principal - Optimisation Globale

```
Tu es un expert Rust senior spécialisé dans l'optimisation de performance.
Ton objectif est d'optimiser AU MAXIMUM la vitesse du projet MPD-Client-v2.

CONTEXTE:
- Projet: Client MPD pour GNOME en Rust/GTK4/libadwaita
- Cible: Performance maximale, latence minimale, fluidité UI
- Contraintes: Pas de régression fonctionnelle, maintenir la lisibilité

RÈGLES ABSOLUES:
1. TOUJOURS mesurer avant et après chaque changement
2. TOUJOURS prioriser la vitesse sur la beauté du code (sans sacrifier la maintenabilité)
3. TOUJOURS utiliser les dernières optimisations Rust (edition 2024)
4. TOUJOURS considérer l'impact mémoire ET CPU
5. TOUJOURS tester sur du matériel réel (pas seulement en debug)

PRINCIPES D'OPTIMISATION:
- Éviter les allocations inutiles (String, Vec, Box)
- Minimiser les copies de données
- Maximiser le cache CPU (localité des données)
- Utiliser l'asynchrone pour les opérations bloquantes
- Batcher les opérations similaires
- Lazy loading partout où possible
- Utiliser les structures de données les plus efficaces

OUTILS À UTILISER:
- cargo clippy --all-targets -- -D warnings
- cargo bloat --release
- cargo udeps
- perf record
- flamegraph
- criterion (benchmarks)

NE JAMAIS:
- Introduire des unsafe sans justification de performance
- Sacrifier la sécurité pour la vitesse
- Casser la compatibilité GTK4
- Ignorer les warnings du compilateur
```

---

## 🔥 Prompts Spécifiques par Catégorie

### 1. Optimisation des Structs de Données

**Prompt: Optimiser les structs Chanson, Album, Flux**
```
Analyse les structs suivantes dans commun/src/modele.rs:

```rust
#[derive(Debug, Clone)]
pub struct Chanson {
    pub titre: Option<String>,
    pub artiste: Option<String>,
    pub album_artiste: Option<String>,
    pub compositeur: Option<String>,
    pub album: Option<String>,
    pub genre: Option<String>,
    pub commentaire: Option<String>,
    pub chemin: String,
    pub extension: String,
    pub id_chanson: String,
    pub piste: i32,
    pub total_pistes: i32,
    pub disque: i32,
    pub duree: u32,
    pub annee: i32,
    pub bits: i32,
}

#[derive(Debug, Clone)]
pub struct Album {
    pub artiste: Option<String>,
    pub nom: Option<String>,
    pub chemin: String,
    pub genre: Option<String>,
    pub id_album: String,
    pub annee: i32,
    pub chemin_piste: String,
}
```

OBJECTIF: Réduire les allocations et les copies

TACHES:
1. Remplacer Option<String> par Option<&'a str> avec lifetimes appropriés
2. Remplacer String par Cow<'a, str> où possible
3. Utiliser Rc<str> ou Arc<str> pour les champs partagés
4. Implémenter #[derive(Default)] pour les champs optionnels
5. Ajouter des méthodes de construction optimisées (builder pattern)
6. Implémenter PartialEq, Eq, Hash pour le caching
7. Utiliser #[repr(C)] pour les structs FFI

CONTRAINTES:
- Maintenir la compatibilité avec le code existant
- Ne pas casser les serializations/deserializations
- Garder le code lisible

EXEMPLE DE SORTIE ATTENDUE:
```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
pub struct Chanson<'a> {
    pub titre: Option<Cow<'a, str>>,
    pub artiste: Option<Cow<'a, str>>,
    pub album_artiste: Option<Cow<'a, str>>,
    pub compositeur: Option<Cow<'a, str>>,
    pub album: Option<Cow<'a, str>>,
    pub genre: Option<Cow<'a, str>>,
    pub commentaire: Option<Cow<'a, str>>,
    pub chemin: Cow<'a, str>,
    pub extension: Cow<'a, str>,
    pub id_chanson: Cow<'a, str>,
    pub piste: u16,
    pub total_pistes: u16,
    pub disque: u16,
    pub duree: u32,
    pub annee: i16,
    pub bits: u16,
}

impl<'a> Default for Chanson<'a> {
    fn default() -> Self {
        Self {
            titre: None,
            artiste: None,
            album_artiste: None,
            compositeur: None,
            album: None,
            genre: None,
            commentaire: None,
            chemin: Cow::Borrowed(""),
            extension: Cow::Borrowed(""),
            id_chanson: Cow::Borrowed(""),
            piste: 0,
            total_pistes: 0,
            disque: 0,
            duree: 0,
            annee: 0,
            bits: 0,
        }
    }
}

impl<'a> Chanson<'a> {
    pub fn into_owned(self) -> Chanson<'static> {
        Chanson {
            titre: self.titre.map(|c| Cow::Owned(c.into_owned())),
            artiste: self.artiste.map(|c| Cow::Owned(c.into_owned())),
            // ... etc
        }
    }
}
```

MÉTRIQUES À MESURER:
- Taille mémoire par instance (std::mem::size_of)
- Temps de clônage
- Temps de comparaison
- Impact sur le temps de chargement
```

---

### 2. Optimisation des Appels MPD

**Prompt: Optimiser la communication avec libmpdclient**
```
Analyse le module commun/src/mpd/commande.rs et connexion.rs

OBJECTIF: Réduire la latence des appels MPD et le nombre de requêtes

TACHES:
1. Implémenter le batching des commandes MPD:
   - Utiliser mpd_command_list_begin() / mpd_command_list_end()
   - Grouper les commandes similaires
   - Implémenter un buffer de commandes

2. Implémenter le cache des réponses:
   - Cache des métadonnées des chansons
   - Cache des informations d'album
   - Cache de l'état de lecture
   - Utiliser dashmap ou RwLock<HashMap> pour le thread-safety

3. Optimiser les appels FFI:
   - Minimiser les conversions String <-> CString
   - Utiliser des buffers réutilisables
   - Éviter les allocations dans les callbacks
   - Utiliser #[repr(C)] sur les structs

4. Implémenter l'asynchrone:
   - Remplacer les appels bloquants par des appels async
   - Utiliser tokio::spawn_blocking pour les opérations FFI
   - Implémenter un système de futures pour les requêtes

EXEMPLE DE SORTIE ATTENDUE:
```rust
pub struct MpdCommandBatcher {
    connection: Rc<RefCell<Connexion>>,
    batch: RefCell<Vec<MpdCommand>>,
    cache: Arc<dashmap::DashMap<String, MpdResponse>>,
}

impl MpdCommandBatcher {
    pub fn new(connection: Rc<RefCell<Connexion>>) -> Self {
        Self {
            connection,
            batch: RefCell::new(Vec::with_capacity(100)),
            cache: Arc::new(dashmap::DashMap::new()),
        }
    }

    pub fn queue_command(&self, command: MpdCommand) {
        self.batch.borrow_mut().push(command);
        
        // Flush si trop de commandes en attente
        if self.batch.borrow().len() >= 50 {
            self.flush();
        }
    }

    pub fn flush(&self) {
        let mut batch = self.batch.borrow_mut();
        if batch.is_empty() {
            return;
        }

        let mut conn = self.connection.borrow_mut();
        unsafe {
            mpd_command_list_begin(&mut *conn.ptr, false);
            for cmd in batch.drain(..) {
                cmd.execute(&mut *conn.ptr);
            }
            mpd_command_list_end(&mut *conn.ptr);
        }
    }

    pub fn get_cached<T: Serialize + DeserializeOwned>(&self, key: &str) -> Option<T> {
        self.cache.get(key)
            .and_then(|entry| serde_json::from_slice(&entry).ok())
    }

    pub fn set_cache<T: Serialize>(&self, key: String, value: &T) {
        if let Ok(data) = serde_json::to_vec(value) {
            self.cache.insert(key, data);
        }
    }
}
```

MÉTRIQUES À MESURER:
- Nombre de requêtes MPD par opération
- Temps de réponse moyen
- Temps de chargement de la bibliothèque
- Consommation mémoire du cache
```

---

### 3. Optimisation du Chargement des Pochettes

**Prompt: Optimiser le chargement et l'affichage des pochettes d'album**
```
Analyse le module commun/src/pochettes.rs et mpdc/src/pochette.rs

OBJECTIF: Réduire le temps de chargement et l'utilisation mémoire

TACHES:
1. Implémenter le lazy loading:
   - Ne charger que les pochettes visibles
   - Utiliser gtk::ListBox avec set_placeholder()
   - Implémenter un observer pour le scroll

2. Implémenter le cache:
   - Cache mémoire (LRU) pour les pochettes récemment utilisées
   - Cache disque pour les pochettes déjà téléchargées
   - Utiliser sha2 pour les clés de cache

3. Optimiser le téléchargement:
   - Remplacer reqwest::blocking par reqwest async
   - Implémenter le streaming avec reqwest::Response::chunk()
   - Downsampler les images avant affichage
   - Utiliser gdk::Texture avec le bon scale_factor

4. Optimiser le rendu:
   - Utiliser gdk::pixbuf::Pixbuf avec le bon colorspace
   - Implémenter un placeholder système
   - Utiliser gtk::Image avec set_pixel_size()

EXEMPLE DE SORTIE ATTENDUE:
```rust
pub struct PochetteCache {
    mem_cache: lru::LruCache<String, Arc<gdk::Texture>>,
    disk_cache: DiskCache,
    client: reqwest::Client,
}

impl PochetteCache {
    pub fn new(capacity: usize) -> Self {
        Self {
            mem_cache: lru::LruCache::new(capacity),
            disk_cache: DiskCache::new("~/.cache/mpdc/pochettes"),
            client: reqwest::Client::new(),
        }
    }

    pub async fn get_pochette(
        &self,
        url: &str,
        size: (i32, i32),
    ) -> Option<Arc<gdk::Texture>> {
        // 1. Vérifier le cache mémoire
        if let Some(texture) = self.mem_cache.get(url) {
            return Some(texture.clone());
        }

        // 2. Vérifier le cache disque
        if let Some(path) = self.disk_cache.get(url) {
            if let Ok(texture) = gdk::Texture::from_file(&path).await {
                let texture = Arc::new(texture);
                self.mem_cache.put(url.to_string(), texture.clone());
                return Some(texture);
            }
        }

        // 3. Télécharger et traiter
        self.download_and_cache(url, size).await
    }

    async fn download_and_cache(
        &self,
        url: &str,
        size: (i32, i32),
    ) -> Option<Arc<gdk::Texture>> {
        let response = self.client.get(url).send().await.ok()?;
        let bytes = response.bytes().await.ok()?;
        
        // Downsample si nécessaire
        let pixbuf = gdk::pixbuf::Pixbuf::from_bytes(&bytes)
            .and_then(|p| p.scale_simple(size.0, size.1, gdk::pixbuf::InterpType::Bilinear));
        
        let texture = gdk::Texture::for_pixbuf(&pixbuf);
        let texture = Arc::new(texture);

        // Sauvegarder dans les caches
        self.disk_cache.set(url, &bytes).await;
        self.mem_cache.put(url.to_string(), texture.clone());

        Some(texture)
    }
}

// Utilisation dans GTK
pub fn setup_lazy_loading(list_box: &gtk::ListBox, cache: Arc<PochetteCache>) {
    list_box.connect_row_activated(move |_, row| {
        let index = row.index();
        let url = get_pochette_url_for_index(index);
        
        // Utiliser spawn_blocking pour ne pas bloquer le thread UI
        glib::spawn_future_local(async move {
            if let Some(texture) = cache.get_pochette(&url, (128, 128)).await {
                // Mettre à jour l'UI
                glib::idle_add(move || {
                    if let Some(row) = list_box.row_at_index(index) {
                        if let Some(image) = row.downcast_ref::<gtk::Image>() {
                            image.set_paintable(Some(&texture));
                        }
                    }
                    glib::ControlFlow::Continue
                });
            }
        });
    });
}
```

MÉTRIQUES À MESURER:
- Temps de chargement de la première pochette
- Temps de scroll dans une liste de 100 albums
- Consommation mémoire avec 100 pochettes chargées
- Taille du cache disque
```

---

### 4. Optimisation de l'Interface Utilisateur

**Prompt: Optimiser le rendu et la réactivité de l'UI GTK4**
```
Analyse les modules mpdc/src/vue_playlist.rs, mpdc/src/liste_pistes.rs, mpdc/src/albums.rs

OBJECTIF: Atteindre 60 FPS et réduire la latence des interactions

TACHES:
1. Optimiser les listes:
   - Implémenter le lazy loading avec gtk::ListBox
   - Utiliser set_placeholder() pour les éléments non chargés
   - Désactiver les animations inutiles
   - Utiliser gtk_widget_set_can_focus(false) sur les widgets non focusables

2. Optimiser les événements:
   - Batcher les mises à jour UI avec gtk_widget_queue_draw()
   - Utiliser glib::idle_add au lieu de timers GTK
   - Désabonner les signaux inutiles
   - Implémenter le debounce pour les événements scroll et resize

3. Optimiser le rendu:
   - Utiliser gtk::Snapshot pour le rendu custom
   - Minimiser les appels à queue_draw()
   - Utiliser gtk::Gesture au lieu de button-press-event
   - Désactiver le sensitive des widgets non utilisables

4. Optimiser les CSS:
   - Simplifier les sélecteurs CSS
   - Éviter les propriétés coûteuses (box-shadow, blur)
   - Utiliser des classes au lieu de sélecteurs complexes

EXEMPLE DE SORTIE ATTENDUE:
```rust
pub fn optimiser_listbox(list_box: &gtk::ListBox) {
    // Désactiver les animations
    list_box.set_transition_duration(0);
    
    // Désactiver le focus sur les rows
    list_box.set_selection_mode(gtk::SelectionMode::None);
    
    // Configurer le placeholder
    let placeholder = gtk::Label::new(Some("Chargement..."));
    placeholder.set_halign(gtk::Align::Center);
    placeholder.set_valign(gtk::Align::Center);
    list_box.set_placeholder(Some(&placeholder));
    
    // Désactiver le focus sur les enfants
    list_box.connect_setup_row(move |_, row| {
        row.set_can_focus(false);
    });
}

pub fn optimiser_scroll(event_controller: &gtk::EventControllerScroll) {
    let mut last_scroll_time = std::time::Instant::now();
    let mut scroll_events = Vec::new();
    
    event_controller.connect_scroll(move |_, _, dx, dy| {
        let now = std::time::Instant::now();
        
        // Debounce: ne traiter que le dernier événement dans les 16ms
        if now.duration_since(last_scroll_time) < std::time::Duration::from_millis(16) {
            scroll_events.push((dx, dy));
            return glib::ControlFlow::Continue;
        }
        
        last_scroll_time = now;
        
        // Traiter tous les événements accumulés
        let (total_dx, total_dy) = scroll_events.drain(..).fold((0.0, 0.0), |acc, (x, y)| {
            (acc.0 + x, acc.1 + y)
        });
        
        // Appliquer le scroll batché
        // ...
        
        glib::ControlFlow::Continue
    });
}

// Optimisation des mises à jour UI
pub fn batch_ui_updates<F: FnOnce()>(callback: F) {
    glib::idle_add_once(move || {
        callback();
        glib::ControlFlow::Break
    });
}
```

MÉTRIQUES À MESURER:
- FPS lors du scroll rapide
- Latence entre le clic et l'action
- Temps de rendu de la fenêtre principale
- Consommation CPU lors de l'interaction
```

---

### 5. Optimisation Async et Concurrence

**Prompt: Implémenter l'asynchrone pour les opérations bloquantes**
```
Analyse l'utilisation de reqwest::blocking dans le projet

OBJECTIF: Éviter de bloquer le thread UI

TACHES:
1. Remplacer reqwest::blocking par reqwest async:
   - Créer un runtime tokio pour l'application
   - Utiliser glib::MainContext::spawn_local pour intégrer avec GTK
   - Implémenter un wrapper async pour les appels bloquants

2. Optimiser les opérations FFI:
   - Utiliser tokio::spawn_blocking pour les appels libmpdclient
   - Implémenter un pool de threads pour les opérations CPU-intensive
   - Utiliser std::sync::mpsc ou tokio::sync::mpsc pour la communication

3. Implémenter le batching async:
   - Grouper les requêtes HTTP
   - Implémenter un système de priorité
   - Utiliser futures::stream::FuturesUnordered

EXEMPLE DE SORTIE ATTENDUE:
```rust
pub struct AsyncRuntime {
    tokio_runtime: tokio::runtime::Runtime,
    glib_context: glib::MainContext,
}

impl AsyncRuntime {
    pub fn new() -> Self {
        let tokio_runtime = tokio::runtime::Builder::new_multi_thread()
            .worker_threads(4)
            .enable_all()
            .build()
            .expect("Failed to create tokio runtime");
        
        let glib_context = glib::MainContext::default();
        
        Self {
            tokio_runtime,
            glib_context,
        }
    }

    pub fn spawn<F>(&self, future: F) 
    where
        F: std::future::Future<Output = ()> + 'static,
    {
        self.tokio_runtime.spawn(future);
    }

    pub fn spawn_local<F>(&self, future: F) 
    where
        F: std::future::Future<Output = ()> + 'static,
    {
        self.glib_context.spawn_local(future);
    }

    pub fn spawn_blocking<F, R>(&self, func: F) -> tokio::task::JoinHandle<R>
    where
        F: FnOnce() -> R + Send + 'static,
        R: Send + 'static,
    {
        self.tokio_runtime.spawn_blocking(func)
    }
}

// Wrapper pour les appels MPD async
pub async fn async_mpd_command(
    conn: Arc<tokio::sync::Mutex<Connexion>>,
    command: impl FnOnce(&mut mpd::Mpd) + Send + 'static,
) -> Result<(), MpdError> {
    tokio::task::spawn_blocking(move || {
        let mut conn = conn.blocking_lock();
        command(&mut *conn.ptr);
        Ok(())
    }).await??
}

// Utilisation dans le code GTK
pub fn load_data_async(window: &adw::ApplicationWindow, runtime: Arc<AsyncRuntime>) {
    let runtime_clone = runtime.clone();
    
    runtime.spawn_local(async move {
        let data = runtime_clone.tokio_runtime
            .spawn_blocking(|| fetch_data_from_mpd())
            .await
            .expect("Failed to fetch data");
        
        // Mettre à jour l'UI sur le thread principal
        glib::idle_add_once(move || {
            update_ui(&window, &data);
        });
    });
}
```

MÉTRIQUES À MESURER:
- Temps d'exécution des opérations async
- Utilisation CPU par thread
- Latence des opérations bloquantes
- Réactivité de l'UI pendant les opérations
```

---

### 6. Optimisation du Build et des Dépendances

**Prompt: Optimiser la configuration Cargo et les dépendances**
```
Analyse les fichiers Cargo.toml du workspace

OBJECTIF: Réduire la taille du binaire et le temps de compilation

TACHES:
1. Optimiser Cargo.toml:
   - Configurer [profile.release] avec lto = "fat"
   - Utiliser codegen-units = 1
   - Activer strip = true
   - Désactiver overflow-checks si sûr
   - Configurer panic = "abort"

2. Optimiser les dépendances:
   - Utiliser les features minimales pour chaque crate
   - Supprimer les dépendances inutilisées (cargo udeps)
   - Utiliser des versions plus récentes si plus optimisées
   - Éviter les features "full"

3. Optimiser le build:
   - Utiliser RUSTFLAGS="-C target-cpu=native"
   - Configurer les dépendances build avec opt-level = 3
   - Utiliser cargo +nightly pour les optimisations expérimentales

EXEMPLE DE SORTIE ATTENDUE:
```toml
# Cargo.toml (workspace)
[workspace]
resolver = "2"
members = ["commun", "mpdc"]

[workspace.package]
version = "2.0.6"
edition = "2024"
license = "GPL-3.0-or-later"

[workspace.dependencies]
# Optimiser les features
reqwest = { version = "0.13.5", features = ["rustls-tls-native-roots", "json", "stream"] }
tokio = { version = "1.0", features = ["rt-multi-thread", "macros", "sync", "time", "fs"] }
libadwaita = { version = "0.9.2", features = ["v1_4"] }
gtk4 = { version = "0.11.5", features = ["v4_6"] }

[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
strip = true
overflow-checks = false
incremental = false

[profile.release.build-override]
opt-level = 3
codegen-units = 4

# mpdc/Cargo.toml
[dependencies]
commun = { path = "../commun" }
reqwest = { workspace = true }
tokio = { workspace = true }
# ... autres dépendances avec features minimales

[build-dependencies]
glib-build-tools = { workspace = true }

# commun/Cargo.toml
[dependencies]
sha2 = { version = "0.11.1", features = ["std"] }
# ... autres dépendances

[features]
default = []
# Pas de features par défaut
```

MÉTRIQUES À MESURER:
- Taille du binaire release (strip)
- Temps de compilation
- Temps de linkage
- Consommation mémoire du binaire
```

---

### 7. Prompt de Benchmarking

**Prompt: Créer des benchmarks pour mesurer les performances**
```
Créer des benchmarks avec criterion pour mesurer l'impact des optimisations

OBJECTIF: Avoir des métriques objectives pour chaque optimisation

TACHES:
1. Créer un module benchmarks/ dans le workspace
2. Implémenter des benchmarks pour:
   - Chargement des structs de données
   - Clônage des objets
   - Comparaison des objets
   - Appels MPD (simulés)
   - Chargement des pochettes
   - Rendu UI (simulé)
   - Traitement des événements

3. Configurer Cargo.toml pour criterion:
   ```toml
   [dev-dependencies]
   criterion = "0.5"
   
   [[bench]]
   name = "donnees"
   harness = false
   
   [[bench]]
   name = "mpd"
   harness = false
   
   [[bench]]
   name = "ui"
   harness = false
   ```

EXEMPLE DE SORTIE ATTENDUE:
```rust
// benchmarks/donnees.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion, Throughput};
use mpdc::commun::modele::Chanson;

fn bench_chanson_creation(c: &mut Criterion) {
    let mut group = c.benchmark_group("Chanson");
    
    group.throughput(Throughput::Elements(1));
    
    group.bench_function("create", |b| {
        b.iter(|| {
            black_box(Chanson {
                titre: Some(String::from("Test Song")),
                artiste: Some(String::from("Test Artist")),
                chemin: String::from("/path/to/song.mp3"),
                id_chanson: String::from("12345"),
                duree: 180,
                // ... autres champs
            });
        })
    });
    
    group.bench_function("clone", |b| {
        let chanson = Chanson {
            titre: Some(String::from("Test Song")),
            artiste: Some(String::from("Test Artist")),
            chemin: String::from("/path/to/song.mp3"),
            id_chanson: String::from("12345"),
            duree: 180,
            // ...
        };
        
        b.iter(|| {
            black_box(chanson.clone());
        })
    });
    
    group.finish();
}

fn bench_chanson_comparaison(c: &mut Criterion) {
    let chanson1 = Chanson {
        titre: Some(String::from("Test Song")),
        artiste: Some(String::from("Test Artist")),
        chemin: String::from("/path/to/song.mp3"),
        id_chanson: String::from("12345"),
        duree: 180,
        // ...
    };
    
    let chanson2 = chanson1.clone();
    
    c.bench_function("partial_eq", |b| {
        b.iter(|| {
            black_box(chanson1.eq(&chanson2));
        })
    });
}

criterion_group!(benches, bench_chanson_creation, bench_chanson_comparaison);
criterion_main!(benches);
```

COMMANDES POUR EXÉCUTER:
```bash
# Benchmark complet
cargo bench --all-targets

# Benchmark spécifique
cargo bench --bench donnees

# Avec flamegraph
cargo bench --bench donnees -- --profile-time 1

# Comparaison avant/après
cargo bench --bench donnees -- --baseline previous
```
```

---

### 8. Prompt de Profiling

**Prompt: Profiler l'application pour identifier les goulots d'étranglement**
```
Utiliser perf, flamegraph et d'autres outils pour analyser les performances

OBJECTIF: Identifier les 3-5 principaux goulots d'étranglement

TACHES:
1. Profiler avec perf:
   ```bash
   # Compiler avec les symboles de debug
   RUSTFLAGS="-C debug-info=2 -C strip=none" cargo build --release
   
   # Enregistrer le profil
   perf record --call-graph dwarf ./target/release/mpdc
   
   # Générer le rapport
   perf report
   
   # Générer un flamegraph
   perf script | FlameGraph/flamegraph.pl > profile.svg
   ```

2. Profiler avec tracing:
   ```rust
   // Ajouter dans main.rs
   use tracing_subscriber::{fmt, prelude::*, EnvFilter};
   
   fn main() {
       fmt()
           .with_env_filter(EnvFilter::from_default_env())
           .with_target(false)
           .with_thread_ids(true)
           .with_thread_names(true)
           .init();
       
       // ... reste du code
   }
   
   // Utiliser dans le code
   use tracing::{info, instrument, span, Level};
   
   #[instrument]
   fn charge_bibliotheque() {
       info!("Chargement de la bibliothèque");
       // ...
   }
   ```

3. Profiler la mémoire avec heaptrack:
   ```bash
   heaptrack ./target/release/mpdc
   heaptrack_gui heaptrack.mpdc.12345.heaptrack
   ```

4. Profiler avec valgrind:
   ```bash
   valgrind --tool=massif --massif-out-file=massif.out ./target/release/mpdc
   ms_print massif.out
   ```

EXEMPLE DE SORTIE ATTENDUE:
```
# Rapport d'analyse perf

## Top 5 des fonctions les plus coûteuses:

1. **mpd_send_command** (25% du temps CPU)
   - Problème: Appels bloquants synchrones
   - Solution: Implémenter async avec tokio::spawn_blocking
   - Impact estimé: -20% temps CPU

2. **Chanson::clone** (15% du temps CPU)
   - Problème: Clônage profond de toutes les String
   - Solution: Utiliser Arc<str> ou Cow<'_, str>
   - Impact estimé: -12% temps CPU, -30% mémoire

3. **gtk::ListBox::insert** (12% du temps CPU)
   - Problème: Insertion synchrone de tous les éléments
   - Solution: Lazy loading avec placeholder
   - Impact estimé: -10% temps CPU, meilleur FPS

4. **reqwest::blocking::get** (10% du temps CPU)
   - Problème: Appels HTTP bloquants sur le thread UI
   - Solution: Remplacer par reqwest async
   - Impact estimé: -8% temps CPU, UI plus réactive

5. **gdk::Texture::from_file** (8% du temps CPU)
   - Problème: Chargement synchrone des images
   - Solution: Chargement async avec cache
   - Impact estimé: -6% temps CPU, -40% temps de scroll

## Recommandations par priorité:

### P1 (Impact Élevé, Effort Faible):
- [ ] Remplacer reqwest::blocking par reqwest async
- [ ] Implémenter le lazy loading des listes
- [ ] Désactiver les animations GTK

### P2 (Impact Élevé, Effort Moyen):
- [ ] Optimiser les structs de données (String -> Cow/Arc)
- [ ] Implémenter le batching des commandes MPD
- [ ] Implémenter le cache des pochettes

### P3 (Impact Moyen, Effort Élevé):
- [ ] Implémenter l'indexation locale des métadonnées
- [ ] Optimiser les appels FFI avec libmpdclient
- [ ] Implémenter le streaming des pochettes
```

---

## 📝 Template de Commit pour les Optimisations

```
perf: [domaine] description de l'optimisation

## Contexte
- Problème: [description du problème de performance]
- Impact: [impact mesuré avant l'optimisation]

## Solution
- [description technique de la solution]

## Résultats
| Métrique | Avant | Après | Gain |
|----------|-------|-------|------|
| [Métrique 1] | X | Y | Z% |
| [Métrique 2] | A | B | C% |

## Benchmarks
```
[sortie des benchmarks]
```

## Notes
- [notes supplémentaires]
- [limitations connues]
- [travail futur]

Closes #[numéro si applicable]
```

---

## 🎯 Résumé des Commandes Utiles

```bash
# Build release optimisé
RUSTFLAGS="-C target-cpu=native -C lto=fat -C codegen-units=1" cargo build --release

# Check les warnings
cargo clippy --all-targets --all-features -- -D warnings

# Vérifier les dépendances inutilisées
cargo udeps

# Vérifier la taille du binaire
cargo bloat --release

# Exécuter les benchmarks
cargo bench --all-targets

# Profiler avec perf
perf record --call-graph dwarf ./target/release/mpdc
perf report

# Profiler avec flamegraph
cargo flamegraph --bench nom_du_benchmark

# Analyser la mémoire
heaptrack ./target/release/mpdc

# Vérifier les fuites mémoire
valgrind --leak-check=full ./target/release/mpdc
```

---

*Document de prompts d'optimisation pour MPD-Client-v2*
*Généré pour une optimisation maximale de la vitesse*
*Dernière mise à jour: 2024*
