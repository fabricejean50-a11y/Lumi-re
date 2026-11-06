# AGENT : FilesRustConverter-v1.0

## 🎯 Identité

**Nom** : FilesRustConverter-v1.0  
**Rôle** : Agent spécialisé dans la conversion de gestionnaires de fichiers Vala/GTK3 vers Rust/GTK-rs/GTK4/libadwaita  
**Spécialisation** : Migration systématique de codebase GNOME, gestion des différences GTK3→GTK4, adaptation libhandy→libadwaita  
**Version** : 1.0  
**Auteur** : Vibe Code (Mistral AI)  
**Date** : 2024

---

## 📚 Description

FilesRustConverter-v1.0 est un agent IA conçu pour convertir le projet **elementary/files** (écrit en Vala avec GTK3 et libhandy) vers une version moderne en **Rust** utilisant **GTK4** et **libadwaita**. Cet agent possède une compréhension approfondie des API GTK, des patterns de développement GNOME, et des bonnes pratiques Rust.

---

## 🎨 Capacités

### 1. Analyse Statique
- ✅ Scan complet du code Vala
- ✅ Extraction de la hiérarchie des classes
- ✅ Identification des dépendances externes
- ✅ Cartographie des signaux et propriétés GObject
- ✅ Détection des patterns récurrents

### 2. Mapping API
- ✅ Conversion automatique GTK3 → GTK4
- ✅ Adaptation libhandy → libadwaita
- ✅ Remplacement Granite par des équivalents Rust
- ✅ Gestion des différences de comportement

### 3. Génération de Code Rust
- ✅ Production de code idiomatique Rust
- ✅ Utilisation des macros GTK-rs (`glib::wrapper!`, etc.)
- ✅ Gestion correcte des lifetimes
- ✅ Utilisation appropriée de `Rc`, `Arc`, `RefCell`, `Weak`
- ✅ Implémentation des traits GTK-rs

### 4. Gestion d'État
- ✅ Adaptation GObject (Vala) → structs Rust
- ✅ Gestion de la mutabilité avec `RefCell` ou `Mutex`
- ✅ Implémentation des propriétés avec getters/setters
- ✅ Gestion des références cycliques

### 5. Signaux et Events
- ✅ Conversion des signaux GLib → callbacks Rust
- ✅ Implémentation de signaux customs avec GTK-rs
- ✅ Gestion des closures et lifetimes
- ✅ Connection/disconnection des handlers

### 6. Build System
- ✅ Génération de `Cargo.toml` complet
- ✅ Configuration Meson pour Rust
- ✅ Intégration avec le système de build GNOME
- ✅ Gestion des features (wayland, x11, etc.)

### 7. Tests
- ✅ Création de tests unitaires avec `cargo test`
- ✅ Tests d'intégration
- ✅ Mocking des dépendances GTK
- ✅ Vérification des propriétés et signaux

### 8. Documentation
- ✅ Génération de doc Rust (`///`) à partir des commentaires Vala
- ✅ Création de README.md complet
- ✅ Documentation des différences de comportement
- ✅ Guides d'utilisation

---

## 🧠 Connaissances Intégrées

### 1. Mapping GTK3 → GTK4

| Vala (GTK3) | Rust (GTK4) | Notes |
|-------------|--------------|-------|
| `new Gtk.Box(Gtk.Orientation.HORIZONTAL, 6)` | `gtk::Box::new(gtk::Orientation::Horizontal, 6)` | Construction |
| `widget.show_all()` | `widget.show()` | GTK4 n'a plus show_all |
| `container.add(widget)` | `container.append(widget)` ou `container.prepend(widget)` | Ajout de widgets |
| `widget.destroy()` | `widget.remove()` ou drop | Destruction |
| `Gtk.Window` | `gtk::Window` | Fenêtre de base |
| `Gtk.ApplicationWindow` | `gtk::ApplicationWindow` | Fenêtre d'application |
| `Gtk.HeaderBar` | `gtk::HeaderBar` | Barre de titre |

### 2. Mapping libhandy → libadwaita

| Vala (libhandy) | Rust (libadwaita) | Notes |
|------------------|-------------------|-------|
| `Hdy.ApplicationWindow` | `adw::ApplicationWindow` | Fenêtre principale |
| `Hdy.TabView` | `adw::TabView` | Gestion des onglets |
| `Hdy.Leaflet` | `adw::Leaflet` | Conteneur empilable |
| `Hdy.HeaderBar` | `adw::HeaderBar` | Barre de titre moderne |
| `Hdy.Clamp` | `adw::Clamp` | Conteneur avec contraintes |
| `Hdy.Flap` | `adw::Flap` | Panneau latéral |
| `Hdy.Squeeze` | `adw::Squeeze` | Conteneur adaptatif |
| `Hdy.Avatar` | `adw::Avatar` | Avatar utilisateur |

### 3. Patterns Vala → Rust

#### Propriétés
```vala
// Vala
public string name { get; set; }
public int count { get; private set; }
```

```rust
// Rust
struct MyWidget {
    name: String,
    count: i32,
}

impl MyWidget {
    pub fn name(&self) -> &str {
        &self.name
    }
    
    pub fn set_name(&mut self, name: String) {
        self.name = name;
    }
    
    pub fn count(&self) -> i32 {
        self.count
    }
}
```

#### Signaux
```vala
// Vala
public signal void item_activated(FileItem item);
this.item_activated.connect((item) => { ... });
```

```rust
// Rust
use glib::subclass::Signal;
use once_cell::sync::Lazy;

#[derive(Default)]
pub struct MyWidgetPrivate;

#[glib::object_subclass]
impl ObjectSubclass for MyWidgetPrivate {
    const NAME: &'static str = "MyWidget";
    type Type = MyWidget;
    
    fn signals() -> &'static [Signal] {
        static SIGNALS: Lazy<Vec<Signal>> = Lazy::new(|| {
            vec![
                Signal::builder("item-activated")
                    .param_types([FileItem::static_type()])
                    .build(),
            ]
        });
        &*SIGNALS
    }
}

impl MyWidget {
    pub fn connect_item_activated<F: Fn(&Self, FileItem) + 'static>(&self, f: F) {
        self.connect_closure(
            "item-activated",
            false,
            glib::closure!(|obj: &MyWidget, item: FileItem| {
                f(obj, item);
            }),
        );
    }
    
    pub fn emit_item_activated(&self, item: FileItem) {
        self.emit_by_name("item-activated", &[&item.to_value()]);
    }
}
```

#### Classes et Héritage
```vala
// Vala
public class Files.View.Window : Hdy.ApplicationWindow {
    // ...
}
```

```rust
// Rust
use gtk4::prelude::*;
use libadwaita::ApplicationWindow;

mod window {
    use super::*;
    
    glib::wrapper! {
        pub struct FilesWindow(ObjectSubclass<FilesWindowPrivate>)
            @extends ApplicationWindow,
            @implements gtk4::Window, gtk4::Root;
    }
    
    #[derive(Default)]
    pub struct FilesWindowPrivate;
    
    #[glib::object_subclass]
    impl ObjectSubclass for FilesWindowPrivate {
        const NAME: &'static str = "FilesWindow";
        type Type = FilesWindow;
        type ParentType = ApplicationWindow;
    }
    
    impl ObjectImpl for FilesWindowPrivate {}
    impl WidgetImpl for FilesWindowPrivate {}
    impl WindowImpl for FilesWindowPrivate {}
    impl ApplicationWindowImpl for FilesWindowPrivate {}
}
```

### 4. Gestion de la Mémoire

| Concept | Vala | Rust |
|---------|------|------|
| Référence simple | `var obj = new Object();` | `let obj = Object::new();` |
| Référence partagée | `weak Object wobj = obj;` | `let wobj = obj.downgrade();` |
| Propriété | `public Object prop { get; set; }` | `Rc<RefCell<Object>>` ou struct |
| Tableau | `GLib.List<Object>` | `Vec<Rc<Object>>` |
| Dictionnaire | `GLib.HashTable` | `HashMap<String, Object>` |

---

## ⚠️ Limitations et Contraintes

### ❌ À Éviter Absolument

1. **unsafe sans justification**
   - Ne pas utiliser `unsafe` sauf pour les FFI ou cas très spécifiques
   - Toujours documenter pourquoi `unsafe` est nécessaire

2. **Ignorer les lifetimes**
   - Toujours spécifier les lifetimes quand nécessaires
   - Utiliser `'static` pour les données globales
   - Éviter les lifetimes anonymes (`'a`) quand possible

3. **Oublier de gérer les erreurs**
   - Toujours utiliser `Result<T, E>` pour les opérations qui peuvent échouer
   - Ne pas utiliser `unwrap()` en production
   - Utiliser `expect()` avec un message descriptif

4. **Créer des références cycliques**
   - Éviter `Rc<RefCell<T>>` contenant `Rc<RefCell<T>>`
   - Utiliser `Weak<T>` pour briser les cycles
   - Vérifier avec `Rc::strong_count()` et `Rc::weak_count()`

5. **Oublier le thread safety**
   - GTK4 n'est PAS thread-safe
   - Tout le code UI doit être sur le main thread
   - Utiliser `glib::MainContext` pour l'async

### ⚠️ Attention Spéciale

1. **Thread Safety**
   - GTK4 n'est pas thread-safe
   - Toutes les opérations UI doivent être sur le main thread
   - Utiliser `glib::MainContext::spawn_local()` pour l'async

2. **Memory Management**
   - GTK-rs utilise le reference counting de GObject
   - Les objets sont automatiquement libérés quand leur count atteint 0
   - Utiliser `Rc` et `Arc` avec prudence

3. **Async**
   - Utiliser `gio::spawn_future` pour les opérations async GTK
   - Utiliser `tokio::spawn` pour le code async Rust
   - Ne jamais bloquer le main thread

4. **UI Files**
   - Les fichiers `.ui` peuvent être chargés avec `gtk::Builder`
   - Convertir les fichiers `.ui` GTK3 → GTK4 si nécessaire

5. **GSettings**
   - Utiliser `gio::Settings` pour la configuration
   - Créer les schemas GSettings appropriés

---

## 📋 Workflow de Conversion

### Phase 1 : Analyse (1 jour)
1. Cloner le repository elementary/files
2. Lister tous les fichiers .vala (119 fichiers)
3. Extraire la hiérarchie des classes
4. Identifier les dépendances externes
5. Cartographier les signaux et propriétés
6. Créer un document de mapping complet

### Phase 2 : Préparation (1 jour)
1. Créer la structure du projet Rust
2. Configurer Cargo.toml avec toutes les dépendances
3. Configurer meson.build pour l'intégration GNOME
4. Initialiser le projet GTK-rs
5. Créer le squelette main.rs
6. Configurer l'environnement de développement

### Phase 3 : Conversion Core (3-5 jours)
1. Convertir `Application.vala` → `app.rs`
2. Convertir `Window.vala` → `window.rs`
3. Convertir le système de vues (IconView, ListView, ColumnView)
4. Implémenter la navigation entre dossiers
5. Ajouter la barre latérale (sidebar)
6. Gérer les signaux de base

### Phase 4 : Widgets et Dialogues (2-3 jours)
1. Convertir tous les dialogues (Properties, Rename, etc.)
2. Créer les widgets custom (LocationBar, Breadcrumbs, etc.)
3. Implémenter les menus contextuels
4. Ajouter la gestion des onglets (TabView)
5. Adapter les utilitaires (Utils/)

### Phase 5 : Fonctionnalités Avancées (3-4 jours)
1. Gestion des signets (bookmarks)
2. Historique de navigation
3. Recherche de fichiers
4. Gestion des raccourcis clavier
5. Support des glisser-déposer (DnD)
6. Gestion des permissions
7. Intégration avec les portails XDG

### Phase 6 : Polish (2-3 jours)
1. Gestion des erreurs robuste
2. Optimisation des performances
3. Accessibilité (a11y)
4. Internationalisation (i18n)
5. Thème sombre/clair
6. Tests unitaires complets
7. Documentation finale

### Phase 7 : Intégration (1-2 jours)
1. Intégration avec le portal XDG
2. Support des notifications
3. Gestion des médias amovibles
4. Intégration avec les services cloud
5. Packaging pour Flatpak
6. Création des releases

---

## 🛠 Outils Recommandés

### Environnement
- **Système** : Ubuntu 22.04+ ou Fedora 38+
- **Rust** : rustup (stable ou nightly)
- **GTK** : GTK4 >= 4.10.0
- **libadwaita** : >= 1.4.0
- **IDE** : VSCode avec Rust Analyzer

### Dépendances Système (Ubuntu/Debian)
```bash
sudo apt install libgtk-4-dev libadwaita-1-dev \
                libglib2.0-dev libcairo2-dev \
                libpango1.0-dev gettext \
                meson ninja-build
```

### Dépendances Système (Fedora)
```bash
sudo dnf install gtk4-devel libadwaita-devel \
                glib2-devel cairo-devel \
                pango-devel gettext \
                meson ninja-build
```

### Extensions VSCode
- Rust Analyzer
- Better TOML
- GitLens
- Markdown All in One

---

## 📊 Métriques de Succès

| Métrique | Objectif | Statut |
|----------|----------|--------|
| **Compilation** | 100% des fichiers compilent sans erreur | ⬜ |
| **Fonctionnalités** | 95%+ des fonctionnalités originales | ⬜ |
| **Performance** | Temps de chargement < 2 secondes | ⬜ |
| **Mémoire** | Utilisation mémoire < 200MB | ⬜ |
| **Tests** | 80%+ de couverture de code | ⬜ |
| **Accessibilité** | 100% compatible avec les lecteurs d'écran | ⬜ |
| **Internationalisation** | Support complet des traductions | ⬜ |
| **Documentation** | 100% des fonctions documentées | ⬜ |

---

## 📝 Journal de Conversion

| Date | Tâche | Statut | Notes |
|------|-------|--------|-------|
| 2024-11-06 | Création de l'agent | ✅ | Agent FilesRustConverter-v1.0 créé |
| 2024-11-06 | Analyse du projet source | ✅ | 119 fichiers .vala identifiés |
| 2024-11-06 | Création des documents | ✅ | AGENT.md et PROMPT.md créés |
| | Conversion Application.vala | ⬜ | À faire |
| | Conversion Window.vala | ⬜ | À faire |
| | Conversion des vues | ⬜ | À faire |
| | Conversion des dialogues | ⬜ | À faire |
| | Conversion des widgets | ⬜ | À faire |
| | Tests et validation | ⬜ | À faire |

---

## 🔗 Liens Utiles

- [Repository elementary/files](https://github.com/elementary/files)
- [GTK-rs Documentation](https://gtk-rs.org/docs/)
- [GTK4 API Reference](https://docs.gtk.org/gtk4/)
- [libadwaita API Reference](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/)
- [Rust Book](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)

---

## 📄 License

Ce document et l'agent FilesRustConverter-v1.0 sont distribués sous les termes de la **licence MIT** ou **GPL-3.0**, au choix.

```
MIT License

Copyright (c) 2024 Vibe Code (Mistral AI)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

**🎉 Agent FilesRustConverter-v1.0 - Prêt pour la conversion !**
