# 🦀 Nautilus My Computer - Conversion Rust

[![Status](https://img.shields.io/badge/Status-En%20Cours-yellow)]
[![License](https://img.shields.io/badge/License-MIT-blue)]
[![Rust](https://img.shields.io/badge/Rust-2021-orange)]

> **Projet** : Conversion de l'extension Nautilus [nautilus-my-computer](https://github.com/yannmasoch/nautilus-my-computer) de Python vers Rust

---

## 📚 Documentation

Ce dépôt contient la documentation complète pour convertir l'extension **My Computer for Nautilus** de Python vers Rust.

### 📖 Documents Disponibles

| Document | Description | Taille |
|----------|-------------|--------|
| **[AGENT_NAUTILUS_RUST_CONVERSION.md](./AGENT_NAUTILUS_RUST_CONVERSION.md) | Agent spécialisé avec instructions détaillées | 37 KB |
| **[PROMPT_NAUTILUS_RUST_CONVERSION.md](./PROMPT_NAUTILUS_RUST_CONVERSION.md) | Prompt technique pour la conversion | 13 KB |
| **[RESUME_CONVERSION.md](./RESUME_CONVERSION.md) | Résumé et plan de conversion | 4 KB |

---

## 🎯 À Propos du Projet

### **My Computer for Nautilus** (Original)

Une extension **indispensable** pour Nautilus (GNOME Files) qui ajoute :

- ✨ Un panneau "My Computer" moderne
- 📁 Accès rapide à tous les disques et volumes
- 🌐 Support des montages réseau (SSHFS, NFS, CIFS, etc.)
- 📊 Indicateurs d'utilisation disque en temps réel
- ⚙️ Vue colonne alternative avec tri personnalisable
- 🎨 Intégration parfaite avec GNOME 42+

**Auteur** : [Yann Masoch](https://github.com/yannmasoch)
**Licence** : MIT
**Étoiles** : ⭐ (à vérifier)

### **Pourquoi une version Rust ?**

| Critère | Python | Rust | Gain |
|---------|--------|------|------|
| **Performance** | ✅ Bonne | ⚡ Excellente | +50-100% |
| **Stabilité** | ✅ Bonne | 🛡️ Excellente | Moins de crashes |
| **Mémoire** | ❌ GC | ✅ Pas de GC | Moins de pauses |
| **Sécurité** | ✅ Bonne | 🔒 Excellente | Moins de bugs |
| **Maintenance** | ✅ Facile | ✅ Facile | Meilleure typage |
| **Intégration** | ✅ Bonne | ✅ Excellente | Bindings natifs |

---

## 🚀 Structure du Projet Cible

```
nautilus-my-computer-rs/
├── Cargo.toml                    # Configuration Rust
├── build.rs                      # Build script
├── src/
│   ├── lib.rs                    # Points d'entrée Nautilus
│   │                               # - nautilus_module_initialize
│   │                               # - nautilus_module_shutdown
│   │                               # - nautilus_module_list_types
│   │
│   ├── extension.rs              # MyComputerExtension
│   │                               # - Structure principale
│   │                               # - Implémentation MenuProvider
│   │                               # - Gestion des fenêtres
│   │
│   ├── models/
│   │   ├── mod.rs                # Ré-exports
│   │   ├── disk.rs               # Modèle Disk
│   │   ├── volume.rs             # Modèle Volume
│   │   ├── mount.rs              # Modèle Mount
│   │   ├── place.rs              # Modèle Place (sidebar)
│   │   ├── bookmark.rs           # Modèle Bookmark
│   │   └── folder.rs             # Modèle PreferredFolder
│   │
│   ├── services/
│   │   ├── mod.rs                # Ré-exports
│   │   ├── monitor.rs            # VolumeMonitor (surveillance volumes)
│   │   ├── settings.rs           # SettingsManager (GSettings)
│   │   ├── usage.rs              # UsageMonitor (surveillance utilisation)
│   │   └── bookmarks.rs          # BookmarksManager
│   │
│   ├── views/
│   │   ├── mod.rs                # Ré-exports
│   │   ├── my_computer.rs        # MyComputerPanel (panneau principal)
│   │   ├── column.rs             # ColumnViewManager (vue colonne)
│   │   └── utils.rs              # Utilitaires vues
│   │
│   ├── widgets/
│   │   ├── mod.rs                # Ré-exports
│   │   ├── card.rs               # DiskCard (carte de disque)
│   │   ├── bar.rs                # UsageBar (barre d'utilisation)
│   │   ├── context_menu.rs       # ContextMenu
│   │   └── styles.rs             # Styles CSS
│   │
│   └── utils/
│       ├── mod.rs                # Ré-exports
│       ├── uri.rs                # Gestion URI
│       ├── format.rs             # Formatage (tailles, dates)
│       ├── native.rs             # Intégration native
│       └── navigation.rs         # Gestion navigation
│
├── data/
│   ├── io.github.yannmasoch.nautilus-my-computer.gschema.xml
│   └── nautilus-my-computer-rs.plugin
│
├── assets/
│   └── icons/                    # Icônes personnalisées
│
└── po/                           # Traductions (gettext)
```

---

## 📦 Dépendances

### **Cargo.toml**

```toml
[package]
name = "nautilus-my-computer-rs"
version = "0.13.4"
authors = ["Votre Nom <votre@email.com>"]
license = "MIT"
readme = "README.md"
homepage = "https://github.com/votre-utilisateur/nautilus-my-computer-rs"
repository = "https://github.com/votre-utilisateur/nautilus-my-computer-rs"
keywords = ["nautilus", "gnome", "files", "gtk4", "extension"]
categories = ["gui", "filesystem"]

[lib]
name = "nautilus_my_computer"
crate-type = ["cdylib"]

[dependencies]
# GTK4 et LibAdwaita
gtk4 = { version = "0.8", features = ["v4_10"] }
gtk4-layer-shell = "0.2"
libadwaita = "0.5"

# GLib, GIO
glib = "0.18"
gio = "0.18"

# GObject
gobject-subclass = "0.1"

# Utilitaires
once_cell = "1.18"
lazy_static = "1.4"
regex = "1.10"

# Gestion d'erreur
anyhow = "1.0"

# Logging
log = "0.4"
env_logger = "0.10"

# Bindings Nautilus (à créer)
nautilus-rs = { path = "../nautilus-rs" }

[build-dependencies]
glib-build-tools = "0.18"

[features]
default = []
# Pour le développement
debug = []
```

### **nautilus-rs** (Bindings Nautilus)

Un crate séparé pour les bindings Nautilus :

```
nautilus-rs/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── menu_provider.rs
│   ├── window.rs
│   ├── window_slot.rs
│   ├── file_info.rs
│   └── operation_result.rs
└── build.rs
```

---

## 🔧 Points d'Entrée Nautilus

Nautilus attend 3 fonctions exportées en C :

### **1. Initialisation**

```rust
// src/lib.rs
use gtk4::glib;
use std::panic::catch_unwind;

/// Nautilus appelle cette fonction pour initialiser le module
/// Doit retourner le GType de l'extension
#[no_mangle]
pub extern "C" fn nautilus_module_initialize(
    module: *mut glib::ffi::GTypeModule,
) -> *mut glib::gobject_ffi::GType {
    catch_unwind(|| {
        let _ = module; // Module non utilisé mais requis
        MyComputerExtension::static_type().to_glib()
    }).unwrap_or_else(|_| {
        eprintln!("Failed to initialize nautilus-my-computer-rs");
        std::ptr::null_mut()
    })
}
```

### **2. Nettoyage**

```rust
/// Nautilus appelle cette fonction lors du déchargement
/// Nettoyage des ressources si nécessaire
#[no_mangle]
pub extern "C" fn nautilus_module_shutdown() {
    unsafe {
        // Réinitialiser les caches globaux si nécessaire
    }
}
```

### **3. Liste des Types**

```rust
/// Nautilus appelle cette fonction pour lister les types exportés
/// Doit retourner une GList de noms de types
#[no_mangle]
pub extern "C" fn nautilus_module_list_types() -> *mut glib::ffi::GList {
    use std::ffi::CString;
    
    let type_ = MyComputerExtension::static_type();
    unsafe {
        let name = CString::new("MyComputerExtension").unwrap();
        glib::ffi::g_list_append(
            std::ptr::null_mut(),
            glib::ffi::g_type_name(type_.to_glib()) as *mut _
        )
    }
}
```

---

## 📊 Mappings Python → Rust

| **Concept Python** | **Rust** | **Notes** |
|-------------------|----------|-----------|
| `GObject.GObject` | `gtk4::glib::Object` | Héritage via `#[glib::object_subclass]` |
| `Nautilus.MenuProvider` | `nautilus_rs::MenuProvider` | Bindings personnalisés |
| `Gtk.Window` | `gtk4::Window` | Direct |
| `Gtk.ListBox` | `gtk4::ListBox` | Direct |
| `Gtk.FlowBox` | `gtk4::FlowBox` | Direct |
| `Gio.Settings` | `gio::Settings` | Direct |
| `Gio.VolumeMonitor` | `gio::VolumeMonitor` | Direct |
| `GLib.idle_add` | `glib::idle_add` | Direct |
| `GObject.Property` | `glib::Properties` | Macro `#[properties]` |
| `threading.Thread` | `std::thread::spawn` | + `glib::MainContext` |
| `weakref.WeakSet` | `glib::WeakRef` | Ou `std::sync::Weak` |
| `dataclasses` | `#[derive(Default)]` | Structures Rust |
| `frozenset` | `HashSet` | `std::collections` |

---

## 🎯 Implémentation de MyComputerExtension

### **Structure**

```rust
// src/extension.rs
use gtk4::glib::{self, subclass::prelude::*, Properties};
use nautilus_rs::prelude::*;
use std::cell::{Cell, RefCell};
use once_cell::sync::OnceCell;

mod imp {
    use super::*;
    
    #[derive(Properties, Default)]
    #[properties(wrapper_type = super::MyComputerExtension)]
    pub struct MyComputerExtension {
        // Préférences
        #[property(get, set)]
        pub start_on_disks: Cell<bool>,
        
        #[property(get, set)]
        pub default_view: RefCell<String>,
        
        // État
        pub windows: RefCell<Vec<glib::WeakRef<gtk4::Window>>>,
        pub panel_slots: RefCell<Vec<glib::WeakRef<gtk4::Widget>>>,
        
        // Services (lazy initialization)
        pub volume_monitor: OnceCell<crate::services::VolumeMonitor>,
        pub settings_manager: OnceCell<crate::services::SettingsManager>,
        
        // Flags
        pub polling_started: Cell<bool>,
        pub refresh_pending: Cell<bool>,
    }
    
    #[glib::object_subclass]
    impl ObjectSubclass for MyComputerExtension {
        const NAME: &'static str = "MyComputerExtension";
        type Type = super::MyComputerExtension;
        type ParentType = nautilus_rs::MenuProvider;
        type Interfaces = (nautilus_rs::MenuProviderInterface,);
    }
    
    impl ObjectImpl for MyComputerExtension {
        fn constructed(&self, obj: &glib::subclass::InitializingObject<Self>) {
            self.parent_constructed(obj);
            obj.init().expect("Failed to initialize MyComputerExtension");
        }
    }
    
    impl MenuProviderImpl for MyComputerExtension {
        fn get_background_items(&self, window: &nautilus_rs::Window) -> Vec<nautilus_rs::MenuItem> {
            self.parent_get_background_items(window)
        }
        
        fn get_context_items(&self, window: &nautilus_rs::Window, files: &[nautilus_rs::FileInfo]) -> Vec<nautilus_rs::MenuItem> {
            self.parent_get_context_items(window, files)
        }
    }
}

glib::wrapper! {
    pub struct MyComputerExtension(
        ObjectSubclass<imp::MyComputerExtension>
    ) @extends nautilus_rs::MenuProvider, @implements nautilus_rs::MenuProviderInterface;
}

impl MyComputerExtension {
    pub fn new() -> Self {
        glib::Object::builder()
            .property("start-on-disks", true)
            .property("default-view", "native")
            .build()
    }
    
    fn init(&self) -> anyhow::Result<()> {
        // 1. Détecter la version de Nautilus
        let nautilus_version = self.detect_nautilus_version();
        log::info!("Nautilus version: {:?}", nautilus_version);
        
        // 2. Initialiser VolumeMonitor
        let monitor = crate::services::VolumeMonitor::new();
        self.imp().volume_monitor.set(monitor).unwrap();
        
        // 3. Initialiser SettingsManager
        let settings = crate::services::SettingsManager::new();
        self.imp().settings_manager.set(settings).unwrap();
        
        // 4. Charger les préférences
        self.load_settings();
        
        // 5. Démarrer la surveillance
        self.start_monitoring();
        
        // 6. Se connecter aux signaux de l'application
        self.connect_application_signals();
        
        Ok(())
    }
}
```

---

## 📚 Exemples de Code

### **Modèle Disk**

```rust
// src/models/disk.rs
use std::path::PathBuf;

#[derive(Debug, Clone)]
pub struct Disk {
    pub name: String,
    pub device: String,
    pub mount_point: PathBuf,
    pub uri: String,
    pub icon_name: String,
    pub total_size: u64,
    pub used_size: u64,
    pub filesystem_type: String,
    pub is_removable: bool,
    pub is_system: bool,
    pub is_network: bool,
}

impl Disk {
    pub fn usage_percentage(&self) -> f64 {
        if self.total_size == 0 {
            0.0
        } else {
            (self.used_size as f64 / self.total_size as f64) * 100.0
        }
    }
    
    pub fn is_system_fstype(fstype: &str) -> bool {
        ["ext4", "ext3", "xfs", "btrfs", "ntfs", "vfat"].contains(&fstype)
    }
    
    pub fn is_network_fstype(fstype: &str) -> bool {
        ["nfs", "cifs", "smb", "sshfs", "fuse", "gvfsd-fuse"].contains(&fstype)
    }
}
```

### **Vue MyComputerPanel**

```rust
// src/views/my_computer.rs
use gtk4::{prelude::*, FlowBox, FlowBoxChild, Picture, ProgressBar, Label, Box};
use crate::models::Disk;

pub struct MyComputerPanel {
    flow_box: FlowBox,
    disks: Vec<Disk>,
}

impl MyComputerPanel {
    pub fn new() -> Self {
        let flow_box = FlowBox::builder()
            .valign(gtk4::Align::Start)
            .halign(gtk4::Align::Fill)
            .selection_mode(gtk4::SelectionMode::Single)
            .build();
        
        Self {
            flow_box,
            disks: Vec::new(),
        }
    }
    
    pub fn populate(&mut self, disks: Vec<Disk>) {
        self.disks = disks;
        self.flow_box.remove_all();
        
        for disk in &self.disks {
            let card = self.create_disk_card(disk);
            self.flow_box.append(&card);
        }
    }
    
    fn create_disk_card(&self, disk: &Disk) -> FlowBoxChild {
        let vbox = Box::new(gtk4::Orientation::Vertical, 6);
        
        let icon = Picture::new();
        icon.set_icon_name(Some(&disk.icon_name));
        icon.set_pixel_size(48);
        
        let name_label = Label::new(Some(&disk.name));
        
        let bar = ProgressBar::new();
        bar.set_fraction(disk.usage_percentage() / 100.0);
        
        vbox.append(&icon);
        vbox.append(&name_label);
        vbox.append(&bar);
        
        let child = FlowBoxChild::new();
        child.set_child(Some(&vbox));
        child
    }
}
```

---

## 📊 Checklist de Conversion

### ✅ Phase 1 : Infrastructure (2 jours)
- [ ] Créer le projet Rust
- [ ] Configurer Cargo.toml
- [ ] Créer build.rs
- [ ] Implémenter les points d'entrée Nautilus
- [ ] Créer MyComputerExtension
- [ ] Tester le chargement dans Nautilus

### ✅ Phase 2 : Modèles (3 jours)
- [ ] Disk
- [ ] Volume
- [ ] Mount
- [ ] PlaceEntry
- [ ] Bookmark
- [ ] PreferredFolder

### ✅ Phase 3 : Services (4 jours)
- [ ] VolumeMonitor
- [ ] SettingsManager
- [ ] UsageMonitor
- [ ] BookmarksManager

### ✅ Phase 4 : Vues (5 jours)
- [ ] MyComputerPanel
- [ ] ColumnViewManager
- [ ] Vue grille (optionnelle)
- [ ] Vue liste (optionnelle)

### ✅ Phase 5 : Bindings (3 jours)
- [ ] Créer nautilus-rs
- [ ] MenuProvider
- [ ] Window
- [ ] WindowSlot
- [ ] FileInfo

### ✅ Phase 6 : Intégration (5 jours)
- [ ] Injection sidebar
- [ ] Gestion des menus
- [ ] Navigation
- [ ] Gestion des onglets
- [ ] Préférences

### ✅ Phase 7 : Fonctionnalités (4 jours)
- [ ] Filtre de localisation
- [ ] Menus contextuels
- [ ] Drag and drop
- [ ] Tooltips

### ✅ Phase 8 : Tests (3 jours)
- [ ] Tests unitaires
- [ ] Tests d'intégration
- [ ] Validation performances
- [ ] Validation compatibilité

### ✅ Phase 9 : Documentation (2 jours)
- [ ] README.md
- [ ] Documentation code
- [ ] Guide installation
- [ ] Guide développement

### ✅ Phase 10 : Packaging (2 jours)
- [ ] Fichier .plugin
- [ ] Schéma GSettings
- [ ] Makefile
- [ ] Script installation
- [ ] Packages distributions

---

## 🚀 Pour Commencer

### 1. **Lire la Documentation**

```bash
# Lire le guide complet
cat AGENT_NAUTILUS_RUST_CONVERSION.md

# Lire le prompt technique
cat PROMPT_NAUTILUS_RUST_CONVERSION.md

# Lire le résumé
cat RESUME_CONVERSION.md
```

### 2. **Explorer le Code Source**

```bash
cd /workspace/nautilus-my-computer

# Structure du projet
ls -la
ls -la nautilus_my_computer/

# Lire les fichiers principaux
hhead -100 nautilus_my_computer/main.py
head -100 nautilus_my_computer/my_computer_view.py
```

### 3. **Créer le Projet Rust**

```bash
# Créer le projet
cargo new nautilus-my-computer-rs --lib
cd nautilus-my-computer-rs

# Configurer Cargo.toml
# (copier depuis PROMPT_NAUTILUS_RUST_CONVERSION.md)

# Créer la structure de fichiers
mkdir -p src/models src/services src/views src/widgets src/utils
mkdir -p data assets/icons
```

### 4. **Implémenter les Points d'Entrée**

```bash
# Créer src/lib.rs avec les fonctions nautilus_module_*
# (voir les exemples dans ce document)

# Builder
cargo build
```

### 5. **Tester dans Nautilus**

```bash
# Copier la bibliothèque
cp target/debug/libnautilus_my_computer.so ~/.local/lib/nautilus/extensions-4/

# Créer le fichier plugin
cat > ~/.local/lib/nautilus/extensions-4/nautilus-my-computer-rs.plugin << EOF
[X-Nautilus]
Extensions=nautilus-my-computer-rs

[nautilus-my-computer-rs]
Module=nautilus-my-computer-rs
IAge=43
EOF

# Redémarrer Nautilus
nautilus -q
nautilus

# Vérifier les logs
RUST_LOG=debug nautilus
```

---

## 💡 Conseils de Développement

### 1. **Développer Incrementalement**

```text
Étape 1 : Extension qui se charge → OK
Étape 2 : Injection sidebar → OK
Étape 3 : Panneau My Computer → OK
Étape 4 : Surveillance volumes → OK
Étape 5 : Autres fonctionnalités → OK
```

### 2. **Tester Fréquemment**

```bash
# Après chaque changement majeur
cargo build --release
cp target/release/libnautilus_my_computer.so ~/.local/lib/nautilus/extensions-4/
nautilus -q
nautilus
```

### 3. **Utiliser les Logs**

```rust
// Dans votre code
use log::{debug, info, warn, error};

fn ma_fonction() {
    debug!("Début de ma_fonction");
    info!("Info importante");
    warn!("Avertissement");
    error!("Erreur critique");
}
```

```bash
# Pour activer les logs
RUST_LOG=debug nautilus
RUST_LOG=info nautilus
```

### 4. **Gérer les Erreurs**

```rust
use anyhow::{Context, Result};

fn charger_config() -> Result<Config> {
    let contenu = std::fs::read("config.toml")
        .context("Impossible de lire le fichier config")?;
    
    toml::from_str(&contenu)
        .context("Impossible de parser le config")
}
```

### 5. **Respecter les Conventions Rust**

```bash
# Formater le code
cargo fmt

# Vérifier avec clippy
cargo clippy

# Vérifier les warnings
cargo build --warnings-as-errors
```

---

## 📞 Support et Communauté

### **Ressources Officielles**

- 📖 [GTK-rs Documentation](https://gtk-rs.org/docs/)
- 📖 [GTK4 API Reference](https://docs.gtk.org/gtk4/)
- 📖 [GIR - Binding Generator](https://github.com/gtk-rs/gir)
- 📖 [Rust FFI Guide](https://doc.rust-lang.org/nomicon/ffi.html)

### **Communauté**

- 💬 [Matrix - GTK-rs](https://matrix.to/#/#gtk-rs:gnome.org)
- 💬 [Discourse - GNOME](https://discourse.gnome.org/)
- 🐙 [GitHub - GTK-rs](https://github.com/gtk-rs)

### **Projets Similaires**

- 🔗 [nautilus-python](https://python-nautilus.readthedocs.io/) - Bindings Python officiels
- 🔗 [gtk4-rs](https://github.com/gtk-rs/gtk4-rs) - Bindings GTK4 pour Rust
- 🔗 [libadwaita-rs](https://github.com/gtk-rs/libadwaita-rs) - Bindings LibAdwaita

---

## 🎉 Contribuer

Les contributions sont les bienvenues ! Voici comment aider :

1. **Forker** le dépôt
2. **Créer une branche** (`git checkout -b feature/ma-fonctionnalite`)
3. **Commiter** vos changements (`git commit -m "Ajout de ma fonctionnalite"`)
4. **Pousser** (`git push origin feature/ma-fonctionnalite`)
5. **Ouvrir une Pull Request**

---

## 📜 Licence

Ce projet est sous licence **MIT** - voir le fichier [LICENSE](LICENSE) pour plus de détails.

---

## 🙏 Remerciements

- **Yann Masoch** - Auteur de l'extension originale [nautilus-my-computer](https://github.com/yannmasoch/nautilus-my-computer)
- **GTK-rs Team** - Pour les excellents bindings Rust
- **GNOME Community** - Pour Nautilus et GTK4
- **Tous les contributeurs** - Pour leurs contributions

---

*Prêt à convertir nautilus-my-computer en Rust ?* **🚀**

*Commencez par lire* **AGENT_NAUTILUS_RUST_CONVERSION.md** *pour le guide complet !*
