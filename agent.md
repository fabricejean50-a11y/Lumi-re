# Agent d'Optimisation de Performance - MPD Client v2

## 🎯 Mission

Optimiser **au maximum** la vitesse et la réactivité du client MPD pour GNOME (MPD-Client-v2), un projet Rust/GTK4/libadwaita.

## 📊 Contexte du Projet

- **Langage**: Rust (Edition 2024)
- **UI Framework**: GTK4 + libadwaita
- **Backend**: libmpdclient (C) via FFI
- **Architecture**: Modulaire (crate `commun` + crate `mpdc`)
- **Cible**: Application desktop GNOME

## 🚀 Stratégies d'Optimisation Prioritaires

### 1. Optimisations Rust (Niveau Code)

#### 🔥 Critique - Impact Élevé
- **[P1]** Remplacer les `String` par `&str` ou `Arc<str>` dans les structs de données (Chanson, Album, etc.)
- **[P1]** Utiliser `Rc<str>` ou `Arc<str>` pour les champs partagés entre plusieurs objets
- **[P1]** Appliquer `#[derive(Default)]` et initialisation lazy où possible
- **[P1]** Remplacer les `Option<String>` par `Option<&str>` avec lifetimes appropriés
- **[P1]** Utiliser `Cow<'_, str>` pour éviter les allocations inutiles
- **[P1]** Optimiser les patterns de matching avec `matches!` macro
- **[P1]** Remplacer les boucles `for` par des itérateurs `.iter()` avec chaînage
- **[P2]** Utiliser `std::mem::take` ou `std::mem::replace` pour vider les collections
- **[P2]** Appliquer `#[cold]` aux fonctions d'erreur rarement appelées
- **[P2]** Utiliser `#[inline]` pour les petites fonctions appelées fréquemment
- **[P2]** Remplacer `Vec::new()` + `push` par `Vec::with_capacity()`
- **[P2]** Utiliser `extend_from_slice` au lieu de boucles de push
- **[P3]** Appliquer `#[repr(C)]` ou `#[repr(transparent)]` aux structs FFI
- **[P3]** Utiliser `MaybeUninit` pour les allocations non initialisées

#### 💡 Optimisations Mémoire
- **[P1]** Implémenter `Pool` ou `Arena` allocator pour les objets fréquemment alloués
- **[P1]** Utiliser `Box<[T]>` au lieu de `Vec<T>` pour les collections statiques
- **[P2]** Appliquer le pattern `Flyweight` pour les données partagées (noms d'artistes, albums)
- **[P2]** Utiliser `smallvec::SmallVec` pour les petites collections
- **[P3]** Implémenter un cache LRU pour les pochettes d'albums
- **[P3]** Utiliser `dashmap` ou `RwLock<HashMap>` pour le cache thread-safe

#### ⚡ Optimisations Async/IO
- **[P1]** Remplacer `reqwest::blocking` par `reqwest` async + tokio
- **[P1]** Implémenter le streaming des pochettes au lieu de chargement complet
- **[P1]** Utiliser `tokio::spawn_blocking` pour les opérations CPU-intensive
- **[P2]** Implémenter le batching des requêtes MPD
- **[P2]** Utiliser `futures::stream::FuturesUnordered` pour les requêtes parallèles
- **[P2]** Appliquer le pattern `debounce` pour les recherches utilisateur
- **[P3]** Implémenter un système de priorité pour les requêtes
- **[P3]** Utiliser `tokio::sync::mpsc` avec buffer limité pour éviter la surcharge

### 2. Optimisations GTK4/libadwaita (Niveau UI)

#### 🎨 Rendu et Performance UI
- **[P1]** Désactiver les animations inutiles avec `gtk_widget_set_can_focus(false)`
- **[P1]** Utiliser `gtk::Snapshot` pour le rendu custom optimisé
- **[P1]** Implémenter le lazy loading des listes (virtualisation)
- **[P2]** Utiliser `gtk::ListBox` avec `set_activate_on_single_click(true)`
- **[P2]** Appliquer `gtk_widget_set_hexpand(false)` et `set_vexpand(false)`
- **[P2]** Utiliser `gtk::Gesture` au lieu de `button-press-event` pour la gestion tactile
- **[P2]** Désactiver le `sensitive` des widgets non utilisables au lieu de les cacher
- **[P3]** Implémenter un système de cache pour les cellules de liste
- **[P3]** Utiliser `gtk::Native` pour les dialogues natifs plus rapides
- **[P3]** Optimiser les CSS avec des sélecteurs moins spécifiques

#### 🖼️ Gestion des Images
- **[P1]** Implémenter le downsampling des pochettes avant affichage
- **[P1]** Utiliser `gdk::Texture` avec `set_scale_factor()` pour le HiDPI
- **[P1]** Charger les images en arrière-plan avec `gdk::Texture::from_file()` async
- **[P2]** Implémenter un placeholder système pour les images manquantes
- **[P2]** Utiliser `gdk::pixbuf::Pixbuf` avec le bon `colorspace`
- **[P3]** Implémenter le cache disque des pochettes avec `std::time::SystemTime`
- **[P3]** Utiliser `image-rs` pour le traitement d'image optimisé

#### 📡 Gestion des Événements
- **[P1]** Utiliser `glib::idle_add` ou `glib::timeout_add` au lieu de timers GTK
- **[P1]** Batcher les mises à jour UI avec `gtk_widget_queue_draw()`
- **[P2]** Désabonner les signaux inutiles avec `disconnect()`
- **[P2]** Utiliser `g_object_weak_ref` pour éviter les memory leaks
- **[P3]** Implémenter un système de throttling pour les événements scroll
- **[P3]** Utiliser `gtk::EventController` pour une gestion optimisée

### 3. Optimisations libmpdclient (Niveau Backend)

#### 🔌 Communication MPD
- **[P1]** Implémenter la connexion persistante au lieu de reconnexion
- **[P1]** Utiliser le protocol binaire MPD au lieu du protocol texte
- **[P1]** Batcher les commandes MPD (`command_list_begin`/`command_list_end`)
- **[P2]** Implémenter le cache des métadonnées localement
- **[P2]** Utiliser `mpd_recv_pair_nb()` pour les réponses non-bloquantes
- **[P2]** Implémenter un timeout adaptatif pour les connexions
- **[P3]** Utiliser `mpd_async` pour les opérations asynchrones
- **[P3]** Implémenter la compression des réponses

#### 🎵 Gestion de la Playlist
- **[P1]** Implémenter le lazy loading de la playlist
- **[P1]** Utiliser `mpd_send_queue_changes_meta()` pour les mises à jour incrémentielles
- **[P2]** Implémenter un cache des positions de lecture
- **[P2]** Utiliser `mpd_playlist` pour la gestion optimisée
- **[P3]** Implémenter la prédiction de lecture (prefetch)

#### 📊 Gestion de la Bibliothèque
- **[P1]** Implémenter l'indexation locale des métadonnées
- **[P1]** Utiliser `mpd_search` avec des filtres optimisés
- **[P2]** Implémenter le cache des résultats de recherche
- **[P2]** Utiliser `mpd_database` pour les requêtes batch
- **[P3]** Implémenter un index inversé pour la recherche rapide

### 4. Optimisations Système

#### ⚙️ Configuration Cargo
```toml
[profile.release]
opt-level = 3
lto = "fat"  # ou "thin" pour un compromis
codegen-units = 1
panic = "abort"
strip = true
overflow-checks = false  # Si sûr du code

[profile.release.build-override]
opt-level = 3

[dependencies]
# Utiliser les features minimales
reqwest = { version = "0.13", features = ["rustls-tls", "json"] }
```

#### 🏗️ Optimisations Build
- **[P1]** Utiliser `cargo +nightly` pour les optimisations expérimentales
- **[P1]** Appliquer `RUSTFLAGS="-C target-cpu=native -C target-feature=+crt-static"`
- **[P2]** Utiliser `cargo-bloat` pour identifier le code mort
- **[P2]** Appliquer `cargo-udeps` pour supprimer les dépendances inutilisées
- **[P3]** Utiliser `cargo-pgo` pour l'optimisation guidée par profil
- **[P3]** Appliquer `cargo-audit` pour la sécurité

#### 📈 Profiling et Benchmarking
- **[P1]** Intégrer `tracing` + `tracing-subscriber` pour le profiling
- **[P1]** Utiliser `criterion` pour les benchmarks
- **[P1]** Intégrer `pprof` pour le profiling CPU/mémoire
- **[P2]** Utiliser `flamegraph` pour visualiser les appels
- **[P2]** Appliquer `perf` ou `vtune` pour l'analyse native
- **[P3]** Intégrer `metrics` pour le monitoring en production

## 📋 Checklist d'Optimisation

### Phase 1: Audit Initial (1-2 jours)
- [ ] Analyser le code avec `cargo clippy --all-targets --all-features -- -D warnings`
- [ ] Exécuter `cargo audit` pour les vulnérabilités
- [ ] Exécuter `cargo bloat --release` pour identifier le code mort
- [ ] Exécuter `cargo udeps` pour les dépendances inutilisées
- [ ] Créer un benchmark de base avec `criterion`
- [ ] Profiler l'application avec `perf` ou `vtune`
- [ ] Identifier les 3-5 goulots d'étranglement principaux

### Phase 2: Optimisations Critiques (3-5 jours)
- [ ] Optimiser les structs de données (String -> &str/Arc<str>)
- [ ] Remplacer reqwest::blocking par reqwest async
- [ ] Implémenter le lazy loading des listes
- [ ] Implémenter le batching des requêtes MPD
- [ ] Optimiser le rendu UI (désactiver animations, lazy loading)
- [ ] Implémenter le cache des pochettes
- [ ] Optimiser la gestion des événements

### Phase 3: Optimisations Avancées (5-10 jours)
- [ ] Implémenter l'indexation locale des métadonnées
- [ ] Appliquer le pattern Flyweight pour les données partagées
- [ ] Implémenter le cache LRU pour les données fréquentes
- [ ] Optimiser les appels FFI avec libmpdclient
- [ ] Implémenter le streaming des pochettes
- [ ] Appliquer le debounce pour les recherches
- [ ] Optimiser les CSS et le rendu

### Phase 4: Optimisations Système (2-3 jours)
- [ ] Configurer `lto = "fat"` dans Cargo.toml
- [ ] Appliquer `RUSTFLAGS="-C target-cpu=native"`
- [ ] Utiliser `panic = "abort"` pour la release
- [ ] Optimiser les dépendances (features minimales)
- [ ] Supprimer le code mort
- [ ] Appliquer `strip = true`

### Phase 5: Validation (1-2 jours)
- [ ] Exécuter tous les benchmarks
- [ ] Comparer avec la version originale
- [ ] Vérifier la consommation mémoire
- [ ] Tester sur différents matériels
- [ ] Vérifier la stabilité
- [ ] Documenter les gains de performance

## 🎯 Objectifs de Performance

| Métrique | Cible | Actuel (estimé) | Status |
|----------|-------|------------------|--------|
| Temps de démarrage | < 500ms | ~1-2s | ❌ |
| Temps de chargement bibliothèque (10k chansons) | < 1s | ~3-5s | ❌ |
| Temps de recherche | < 100ms | ~200-500ms | ❌ |
| Temps de changement de piste | < 50ms | ~100-200ms | ❌ |
| Consommation mémoire | < 100MB | ~150-200MB | ❌ |
| FPS UI (scrolling) | 60 FPS | ~30-45 FPS | ❌ |
| Latence clic | < 16ms | ~30-50ms | ❌ |

## 🛠️ Outils Recommandés

### Analyse Statique
- `cargo clippy --all-targets -- -D warnings`
- `cargo check --release`
- `cargo doc --no-deps --open`
- `cargo metadata --format-version 1`

### Profiling
- `perf record --call-graph dwarf ./target/release/mpdc`
- `flamegraph` (via `cargo flamegraph`)
- `tracing` + `tracing-subscriber`
- `pprof` (Google Performance Tools)
- `heaptrack` pour la mémoire
- `valgrind --tool=massif`

### Benchmarking
- `criterion` pour les micro-benchmarks
- `iai` pour les benchmarks IA-aware
- `gtk-rs` benchmarks internes
- Tests manuels avec `time`

### Optimisation
- `cargo bloat --release`
- `cargo udeps`
- `cargo tree` pour les dépendances
- `cargo pgo` pour l'optimisation guidée

## 📚 Documentation de Référence

- [Rust Performance Patterns](https://nnethercote.github.io/perf-book/)
- [GTK4 Performance Guide](https://docs.gtk.org/gtk4/perf.html)
- [libmpdclient Documentation](https://www.musicpd.org/doc/libmpdclient/)
- [The Rust Performance Book](https://nnethercote.github.io/perf-book/)
- [Rust Async Book](https://rust-lang.github.io/async-book/)
- [Tokio Tutorial](https://tokio.rs/tokio/tutorial)

## 💬 Communication

- **Format des commits**: `perf: optimiser le chargement des pochettes (#123)`
- **Tags**: Utiliser `perf`, `optimization`, `speed`
- **Branches**: `perf/nom-de-l-optimisation`
- **Reviews**: Priorité aux PRs avec des benchmarks

## ⚠️ Contraintes et Limites

- **Compatibilité**: Maintenir la compatibilité avec GTK4.6+
- **Stabilité**: Aucune régression fonctionnelle acceptable
- **Maintenabilité**: Le code doit rester lisible et documenté
- **Portabilité**: Fonctionner sur Linux (GNOME) principalement
- **Licence**: Respecter la licence GPL-3.0-or-later

## ✅ Critères d'Acceptation

1. **Mesurable**: Chaque optimisation doit avoir une métrique avant/après
2. **Testée**: Aucune régression fonctionnelle
3. **Documentée**: Les changements doivent être documentés
4. **Reproductible**: Les benchmarks doivent être reproductibles
5. **Progressive**: Les optimisations doivent être incrémentielles

---

*Document généré pour l'optimisation maximale de MPD-Client-v2*
*Dernière mise à jour: 2024*
