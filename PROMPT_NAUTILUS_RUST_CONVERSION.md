# 🎯 PROMPT DE CONVERSION : Nautilus My Computer (Python → Rust)

## 📋 Contexte

**Projet Source** : [nautilus-my-computer](https://github.com/yannmasoch/nautilus-my-computer) par Yann Masoch

**Objectif** : Convertir cette extension Nautilus populaire de **Python** vers **Rust** tout en conservant 100% des fonctionnalités.

**Technologies** :
- **Source** : Python 3 + PyGObject + GTK4 + LibAdwaita + Nautilus API
- **Cible** : Rust 2021 + gtk4-rs + libadwaita-rs + nautilus-rs (à créer)

---

## 🎨 Description du Projet

### Qu'est-ce que "My Computer for Nautilus" ?

Une extension pour Nautilus (GNOME Files) qui ajoute un **panneau "My Computer"** moderne, remplaçant la vue "Other Locations" supprimée par GNOME.

**Fonctionnalités principales** :
- Vue disque complète avec indicateurs d'utilisation
- Navigation intuitive vers tous les points de montage
- Vue colonne alternative avec tri personnalisable
- Surveillance en temps réel des montages/démontages
- Préférences complètes via GSettings
- Support réseau (GVfs, SSHFS, NFS, CIFS, WebDAV)
- Gestion des signets et dossiers préférés
- Filtre de localisation dans la barre d'adresse
- Menus contextuels avancés

---

## 📦 Architecture Technique

### Structure Actuelle (Python)

```
nautilus-my-computer/
├── nautilus-my-computer.py          # Point d'entrée
├── nautilus_my_computer/
│   ├── main.py                     # MyComputerExtension (130KB)
│   ├── my_computer_view.py         # Vue panneau disques (130KB)
│   ├── column_view.py              # Vue colonne (181KB)
│   ├── common.py                   # Utilitaires (39KB)
│   ├── context_menu.py             # Menus contextuels
│   ├── bookmarks.py                # Gestion signets
│   ├── nautilus_prefs.py           # Préférences Nautilus
│   ├── location_filter.py          # Filtre localisation
│   └── widgets.py                  # Widgets personnalisés (161KB)
└── io.github.yannmasoch.nautilus-my-computer.gschema.xml
```

### Classe Principale : MyComputerExtension

```python
class MyComputerExtension(GObject.GObject, Nautilus.MenuProvider):
    def __init__(self):
        # Initialisation
        # - Détection version Nautilus
        # - Initialisation VolumeMonitor
        # - Chargement GSettings
        # - Connexion aux signaux
        
    def menu_provider_get_background_items(self, window):
        # Items de menu de fond
        
    def menu_provider_get_context_items(self, window, files):
        # Items de menu contextuel
        
    def _on_window_added(self, app, window):
        # Gestion nouvelles fenêtres
        
    def _inject_separate_computer_row(self, nautilus_win):
        # Injection ligne Computer dans sidebar
```

---

## 🎯 Spécifications de Conversion

### 1. Structure du Projet Rust

```
nautilus-my-computer-rs/
├── Cargo.toml
├── build.rs
├── src/
│   ├── lib.rs                    # Points d'entrée Nautilus
│   ├── extension.rs              # MyComputerExtension
│   ├── models/
│   │   ├── disk.rs               # Modèle Disk
│   │   ├── volume.rs             # Modèle Volume
│   │   └── place.rs              # Modèle Place
│   ├── services/
│   │   ├── monitor.rs            # VolumeMonitor
│   │   └── settings.rs           # SettingsManager
│   ├── views/
│   │   ├── my_computer.rs        # MyComputerPanel
│   │   └── column.rs             # ColumnView
│   └── widgets/
│       └── card.rs               # DiskCard
├── data/
│   ├── nautilus-my-computer-rs.plugin
│   └── io.github.yannmasoch.nautilus-my-computer.gschema.xml
└── assets/
    └── icons/
```

### 2. Points d'Entrée Nautilus

```rust
// src/lib.rs
use gtk4::glib;
use std::panic::catch_unwind;

#[no_mangle]
pub extern "C" fn nautilus_module_initialize(
    module: *mut glib::ffi::GTypeModule,
) -> *mut glib::gobject_ffi::GType {
    catch_unwind(|| {
        let _ = module;
        MyComputerExtension::static_type().to_glib()
    }).unwrap_or_else(|_| std::ptr::null_mut())
}

#[no_mangle]
pub extern "C" fn nautilus_module_shutdown() {
    // Nettoyage
}

#[no_mangle]
pub extern "C" fn nautilus_module_list_types() -> *mut glib::ffi::GList {
    use std::ffi::CString;
    let type_ = MyComputerExtension::static_type();
    unsafe {
        glib::ffi::g_list_append(
            std::ptr::null_mut(),
            glib::ffi::g_type_name(type_.to_glib()) as *mut _
        )
    }
}
```

### 3. MyComputerExtension en Rust

```rust
// src/extension.rs
use gtk4::glib::{self, subclass::prelude::*, Properties};
use nautilus_rs::prelude::*;

mod imp {
    use super::*;
    
    #[derive(Properties, Default)]
    #[properties(wrapper_type = super::MyComputerExtension)]
    pub struct MyComputerExtension {
        pub start_on_disks: glib::Cell<bool>,
        pub windows: glib::RefCell<Vec<glib::WeakRef<gtk4::Window>>>,
    }
    
    #[glib::object_subclass]
    impl ObjectSubclass for MyComputerExtension {
        const NAME: &'static str = "MyComputerExtension";
        type Type = super::MyComputerExtension;
        type ParentType = nautilus_rs::MenuProvider;
    }
    
    impl ObjectImpl for MyComputerExtension {}
    
    impl MenuProviderImpl for MyComputerExtension {
        fn get_background_items(&self, window: &nautilus_rs::Window) -> Vec<nautilus_rs::MenuItem> {
            vec![]
        }
        
        fn get_context_items(&self, window: &nautilus_rs::Window, files: &[nautilus_rs::FileInfo]) -> Vec<nautilus_rs::MenuItem> {
            vec![]
        }
    }
}

glib::wrapper! {
    pub struct MyComputerExtension(
        ObjectSubclass<imp::MyComputerExtension>
    ) @extends nautilus_rs::MenuProvider;
}
```

### 4. Modèles de Données

**Disk** :
```rust
// src/models/disk.rs
#[derive(Debug, Clone)]
pub struct Disk {
    pub name: String,
    pub device: String,
    pub mount_point: std::path::PathBuf,
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
        if self.total_size == 0 { 0.0 } else {
            (self.used_size as f64 / self.total_size as f64) * 100.0
        }
    }
}
```

**PlaceEntry** :
```rust
// src/models/place.rs
#[derive(Debug, Clone)]
pub struct PlaceEntry {
    pub name: String,
    pub position: i32,
    pub label: String,
    pub icon: String,
    pub uri: String,
    pub visible: bool,
}

impl PlaceEntry {
    pub fn computer() -> Self {
        Self {
            name: "my_computer".to_string(),
            position: 0,
            label: "Computer".to_string(),
            icon: "computer-symbolic".to_string(),
            uri: "computer:///".to_string(),
            visible: true,
        }
    }
}
```

### 5. VolumeMonitor

```rust
// src/services/monitor.rs
use std::sync::{Arc, Mutex};
use gtk4::glib::{self, Sender};
use gio::prelude::*;
use crate::models::Disk;

pub struct VolumeMonitor {
    volume_monitor: gio::VolumeMonitor,
    disks: Arc<Mutex<Vec<Disk>>>,
}

impl VolumeMonitor {
    pub fn new() -> Self {
        Self {
            volume_monitor: gio::VolumeMonitor::get(),
            disks: Arc::new(Mutex::new(Vec::new())),
        }
    }
    
    pub fn start(&self, sender: Sender<()>) {
        let monitor = &self.volume_monitor;
        let disks = self.disks.clone();
        
        monitor.connect_volume_added(move |_, _| {
            let _ = sender.send(());
        });
        
        monitor.connect_mount_added(move |_, _| {
            let _ = sender.send(());
        });
        
        self.refresh();
    }
    
    pub fn refresh(&self) {
        let mut disks = self.disks.lock().unwrap();
        disks.clear();
        
        for mount in self.volume_monitor.mounts() {
            if let Some(root) = mount.root() {
                if let Some(uri) = root.uri() {
                    if let Some(disk) = self.extract_disk_info(&mount, &uri) {
                        disks.push(disk);
                    }
                }
            }
        }
    }
    
    pub fn get_disks(&self) -> Vec<Disk> {
        self.disks.lock().unwrap().clone()
    }
}
```

### 6. Vue My Computer

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
        name_label.set_halign(gtk4::Align::Center);
        
        let bar = ProgressBar::new();
        bar.set_fraction(disk.usage_percentage() / 100.0);
        
        let info_label = Label::new(Some(&format!(
            "{} / {}",
            format_size(disk.used_size),
            format_size(disk.total_size)
        )));
        
        vbox.append(&icon);
        vbox.append(&name_label);
        vbox.append(&bar);
        vbox.append(&info_label);
        
        let child = FlowBoxChild::new();
        child.set_child(Some(&vbox));
        child
    }
}

fn format_size(bytes: u64) -> String {
    const KB: u64 = 1024;
    const MB: u64 = KB * 1024;
    const GB: u64 = MB * 1024;
    
    if bytes >= GB {
        format!("{:.1} GB", bytes as f64 / GB as f64)
    } else if bytes >= MB {
        format!("{:.1} MB", bytes as f64 / MB as f64)
    } else if bytes >= KB {
        format!("{:.1} KB", bytes as f64 / KB as f64)
    } else {
        format!("{} B", bytes)
    }
}
```

---

## 📊 Checklist de Conversion

### Phase 1 : Infrastructure
- [ ] Créer le projet Rust
- [ ] Configurer Cargo.toml
- [ ] Créer build.rs
- [ ] Implémenter les points d'entrée Nautilus
- [ ] Créer MyComputerExtension
- [ ] Tester le chargement dans Nautilus

### Phase 2 : Modèles
- [ ] Disk
- [ ] Volume
- [ ] PlaceEntry
- [ ] Bookmark

### Phase 3 : Services
- [ ] VolumeMonitor
- [ ] SettingsManager
- [ ] UsageMonitor

### Phase 4 : Vues
- [ ] MyComputerPanel
- [ ] ColumnViewManager

### Phase 5 : Bindings Nautilus
- [ ] Créer nautilus-rs
- [ ] MenuProvider
- [ ] Window
- [ ] FileInfo

### Phase 6 : Intégration
- [ ] Injection sidebar
- [ ] Gestion des menus
- [ ] Navigation
- [ ] Préférences

---

## 📚 Cargo.toml

```toml
[package]
name = "nautilus-my-computer-rs"
version = "0.13.4"
authors = ["Votre Nom"]
license = "MIT"

[lib]
name = "nautilus_my_computer"
crate-type = ["cdylib"]

[dependencies]
gtk4 = { version = "0.8", features = ["v4_10"] }
libadwaita = "0.5"
glib = "0.18"
gio = "0.18"
gobject-subclass = "0.1"
once_cell = "1.18"
lazy_static = "1.4"
anyhow = "1.0"
log = "0.4"
env_logger = "0.10"
nautilus-rs = { path = "../nautilus-rs" }

[build-dependencies]
glib-build-tools = "0.18"
```

---

## 🚀 Commandes Utiles

### Build
```bash
cargo build --release
```

### Installation
```bash
cp target/release/libnautilus_my_computer.so ~/.local/lib/nautilus/extensions-4/
cp data/nautilus-my-computer-rs.plugin ~/.local/lib/nautilus/extensions-4/
cp data/*.gschema.xml ~/.local/share/glib-2.0/schemas/
glib-compile-schemas ~/.local/share/glib-2.0/schemas/
nautilus -q
```

### Debug
```bash
RUST_LOG=debug nautilus
RUST_BACKTRACE=1 nautilus
```

---

## 💡 Conseils

1. **Commencez petit** : Extension minimale qui se charge
2. **Testez fréquemment** : Vérifiez après chaque changement
3. **Utilisez les logs** : `log::debug!`, `log::error!`
4. **Gérez les erreurs** : Utilisez `anyhow::Result`
5. **Respectez les conventions** : `cargo fmt`, `cargo clippy`

---

*Bon courage avec la conversion ! 🦀*
