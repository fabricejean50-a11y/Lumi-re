# Agent de Conversion : Nautilus My Computer → Rust

## 🎯 Mission

Convertir l'extension Nautilus [nautilus-my-computer](https://github.com/yannmasoch/nautilus-my-computer) de **Python** vers **Rust**, en préservant toutes les fonctionnalités :

- ✅ Panneau "My Computer" avec visualisation des disques
- ✅ Intégration sidebar (barre latérale)
- ✅ Vue colonne alternative
- ✅ Système de préférences GSettings
- ✅ Menus contextuels
- ✅ Filtres de localisation
- ✅ Gestion des signets
- ✅ Détection automatique des montages/volumes
- ✅ Affichage de l'utilisation disque
- ✅ Prise en charge réseau (GVfs, SSHFS, etc.)

---

## 📦 Contexte Technique

### Architecture Actuelle (Python)

```
nautilus-my-computer/
├── nautilus-my-computer.py          # Point d'entrée (module hyphené)
├── nautilus_my_computer/
│   ├── __init__.py                 # Version
│   ├── main.py                     # MyComputerExtension (classe principale)
│   ├── my_computer_view.py         # Vue panneau disques
│   ├── column_view.py              # Vue colonne
│   ├── common.py                   # Utilitaires partagés
│   ├── context_menu.py             # Menus contextuels
│   ├── bookmarks.py                # Gestion signets
│   ├── preferred_folders.py        # Dossiers préférés
│   ├── nautilus_prefs.py           # Préférences Nautilus
│   ├── location_filter.py          # Filtre barre d'adresse
│   ├── widgets.py                  # Widgets personnalisés
│   ├── selection.py                # Gestion sélection
│   ├── native_selection.py         # Sélection native
│   ├── components.py               # Composants UI
│   ├── column_state.py             # État vue colonne
│   └── uris.py                     # Utilitaires URI
├── io.github.yannmasoch.nautilus-my-computer.gschema.xml  # Schéma GSettings
└── assets/                         # Icônes, images
```

### Stack Technique

| Composant | Python | Rust (cible) |
|-----------|--------|--------------|
| **GTK** | PyGObject (gi.repository) | `gtk4` crate |
| **LibAdwaita** | PyGObject | `libadwaita` crate |
| **Nautilus** | python-nautilus | `nautilus-rs` (à créer) |
| **GObject** | PyGObject | `glib` crate |
| **GIO** | PyGObject | `gio` crate |
| **GSettings** | PyGObject | `gio` crate |
| **Async** | GLib.idle_add | `glib::idle_add` |
| **Threads** | threading | `std::thread` + `glib::MainContext` |

---

## 🔧 Prérequis

### 1. Dépendances Rust

```toml
# Cargo.toml
[package]
name = "nautilus-my-computer-rs"
version = "0.13.4"
edition = "2021"

[dependencies]
gtk4 = { version = "0.8", features = ["v4_10"] }
gtk4-layer-shell = "0.2"
libadwaita = "0.5"
glib = "0.18"
gio = "0.18"
gobject-subclass = "0.1"
once_cell = "1.18"
lazy_static = "1.4"
regex = "1.10"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
anyhow = "1.0"
log = "0.4"
env_logger = "0.10"
rayon = "1.8"
```

### 2. Bindings Nautilus

Créer un crate `nautilus-rs` avec les bindings pour :
- `NautilusMenuProvider`
- `NautilusMenuItem`
- `NautilusWindow`
- `NautilusWindowSlot`
- `NautilusFileInfo`
- `NautilusOperationResult`

**Approche recommandée** : Utiliser `gir` (GitHub: https://github.com/gtk-rs/gir) pour générer automatiquement les bindings depuis le fichier `.gir` de Nautilus.

```bash
# Génération des bindings
cargo install gir
gir -o nautilus-sys src/nautilus.gir
```

### 3. Structure du Projet Rust

```
nautilus-my-computer-rs/
├── Cargo.toml
├── build.rs                    # Génération ressources
├── src/
│   ├── main.rs                 # Point d'entrée
│   ├── lib/
│   │   ├── mod.rs              # Ré-exports
│   │   ├── extension.rs        # MyComputerExtension
│   │   ├── views/
│   │   │   ├── mod.rs
│   │   │   ├── my_computer.rs  # Vue panneau
│   │   │   ├── column.rs       # Vue colonne
│   │   │   └── utils.rs        # Utilitaires vues
│   │   ├── models/
│   │   │   ├── mod.rs
│   │   │   ├── disk.rs         # Modèle disque
│   │   │   ├── volume.rs       # Modèle volume
│   │   │   ├── mount.rs        # Modèle montage
│   │   │   └── place.rs        # Modèle place
│   │   ├── services/
│   │   │   ├── mod.rs
│   │   │   ├── monitor.rs      # Surveillance volumes
│   │   │   ├── settings.rs     # Gestion GSettings
│   │   │   └── usage.rs        # Surveillance utilisation
│   │   ├── widgets/
│   │   │   ├── mod.rs
│   │   │   ├── card.rs         # Carte disque
│   │   │   ├── bar.rs          # Barre utilisation
│   │   │   └── context_menu.rs # Menu contextuel
│   │   └── utils/
│   │       ├── mod.rs
│   │       ├── uri.rs          # Gestion URI
│   │       ├── format.rs       # Formatage
│   │       └── native.rs       # Intégration native
│   └── resources/
│       └── ui/                 # Fichiers .ui
├── data/
│   ├── io.github.yannmasoch.nautilus-my-computer.gschema.xml
│   └── nautilus-my-computer-rs.plugin
├── assets/
│   └── icons/
└── po/                         # Traductions
```

---

## 🚀 Étapes de Conversion

### Phase 1 : Infrastructure de Base (1-2 jours)

#### 1.1 Créer le squelette du projet

```bash
cargo new nautilus-my-computer-rs --lib
cd nautilus-my-computer-rs
```

#### 1.2 Configurer Cargo.toml

```toml
[lib]
name = "nautilus_my_computer"
crate-type = ["cdylib"]

[build-dependencies]
glib-build-tools = "0.18"
```

#### 1.3 Créer le point d'entrée

**src/lib.rs** :
```rust
mod extension;
mod models;
mod services;
mod utils;
mod views;
mod widgets;

use extension::MyComputerExtension;
use gtk4::glib;

// Export pour Nautilus
#[no_mangle]
pub extern "C" fn nautilus_module_initialize(
    module: *mut glib::ff::GTypeModule,
) -> *mut glib::gobject_ffi::GType {
    use std::panic::catch_unwind;
    
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
    let cstring = CString::new("MyComputerExtension").unwrap();
    unsafe {
        glib::ffi::g_list_append(
            std::ptr::null_mut(),
            glib::ffi::g_type_name(type_.to_glib()) as *mut _
        )
    }
}
```

#### 1.4 Créer la classe principale

**src/extension/mod.rs** :
```rust
use gtk4::{
    glib::{self, subclass::prelude::*, Properties},
    prelude::*,
    Application, Window,
};
use nautilus_rs::prelude::*;
use std::cell::{Cell, RefCell};

mod imp {
    use super::*;
    
    #[derive(Default, Properties)]
    #[properties(wrapper_type = super::MyComputerExtension)]
    pub struct MyComputerExtension {
        #[property(get, set)]
        pub start_on_disks: Cell<bool>,
        
        pub windows: RefCell<Vec<Window>>,
        pub settings: RefCell<Option<gio::Settings>>,
    }
    
    #[glib::object_subclass]
    impl ObjectSubclass for MyComputerExtension {
        const NAME: &'static str = "MyComputerExtension";
        type Type = super::MyComputerExtension;
        type ParentType = nautilus_rs::MenuProvider;
    }
    
    impl ObjectImpl for MyComputerExtension {}
    impl MenuProviderImpl for MyComputerExtension {}
}

glib::wrapper! {
    pub struct MyComputerExtension(
        ObjectSubclass<imp::MyComputerExtension>
    ) @extends nautilus_rs::MenuProvider;
}

impl MyComputerExtension {
    pub fn new() -> Self {
        glib::Object::builder()
            .property("start-on-disks", true)
            .build()
    }
}
```

### Phase 2 : Modèles de Données (2-3 jours)

#### 2.1 Modèle Disque

**src/models/disk.rs** :
```rust
use std::path::PathBuf;
use gtk4::gdk;
use gio::prelude::*;

#[derive(Debug, Clone)]
pub struct Disk {
    pub name: String,
    pub device: String,
    pub mount_point: PathBuf,
    pub uri: String,
    pub icon_name: String,
    pub total_size: u64,
    pub used_size: u64,
    pub is_removable: bool,
    pub is_system: bool,
    pub filesystem_type: String,
    pub label: String,
    pub uuid: Option<String>,
}

impl Disk {
    pub fn usage_percentage(&self) -> f64 {
        if self.total_size == 0 {
            0.0
        } else {
            (self.used_size as f64 / self.total_size as f64) * 100.0
        }
    }
    
    pub fn is_network(&self) -> bool {
        let network_fstypes = [
            "nfs", "nfs4", "cifs", "smb", "smb2", "smbfs",
            "fuse", "fuse.sshfs", "fuse.rclone", "fuse.s3fs",
            "sshfs", "ftpfs", "gvfsd-fuse", "davfs"
        ];
        network_fstypes.contains(&self.filesystem_type.as_str())
    }
}
```

#### 2.2 Modèle Volume

**src/models/volume.rs** :
```rust
use std::path::PathBuf;
use gio::Volume;

#[derive(Debug, Clone)]
pub struct AppVolume {
    pub native: Volume,
    pub name: String,
    pub icon: String,
    pub mount: Option<gio::Mount>,
    pub is_removable: bool,
    pub is_media_removable: bool,
    pub is_media_check_automatic: bool,
    pub can_eject: bool,
    pub can_mount: bool,
    pub can_unmount: bool,
}

impl From<&gio::Volume> for AppVolume {
    fn from(volume: &gio::Volume) -> Self {
        Self {
            native: volume.clone(),
            name: volume.name().unwrap_or_default(),
            icon: volume.icon().map(|i| i.to_string()).unwrap_or_default(),
            mount: volume.mount(),
            is_removable: volume.is_removable(),
            is_media_removable: volume.is_media_removable(),
            is_media_check_automatic: volume.is_media_check_automatic(),
            can_eject: volume.can_eject(),
            can_mount: volume.can_mount(),
            can_unmount: volume.can_unmount(),
        }
    }
}
```

#### 2.3 Modèle Place (Sidebar)

**src/models/place.rs** :
```rust
use gtk4::gdk;

#[derive(Debug, Clone)]
pub struct PlaceEntry {
    pub name: String,           // Identifiant interne
    pub position: i32,          // Ordre visuel
    pub label: String,          // Libellé affiché
    pub icon: String,           // Nom de l'icône
    pub uri: String,            // URI de la location
    pub visible: bool,          // Visible dans sidebar
    pub tooltip: String,        // Info-bulle
    pub order_index: i32,       // Index de tri
    pub droppable: bool,        // Accepte le drop de fichiers
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
            tooltip: "Open My Computer".to_string(),
            order_index: 0,
            droppable: false,
        }
    }
    
    pub fn home() -> Self {
        let home = glib::home_dir().unwrap_or_default();
        let uri = glib::filename_to_uri(&home, None);
        Self {
            name: "home".to_string(),
            position: 1,
            label: "Home".to_string(),
            icon: "user-home-symbolic".to_string(),
            uri,
            visible: true,
            tooltip: "".to_string(),
            order_index: 1,
            droppable: true,
        }
    }
}
```

### Phase 3 : Services (3-4 jours)

#### 3.1 Surveillance des Volumes

**src/services/monitor.rs** :
```rust
use std::sync::{Arc, Mutex};
use gtk4::glib::{self, MainContext};
use gio::prelude::*;
use crate::models::{Disk, AppVolume};

pub struct VolumeMonitor {
    volume_monitor: gio::VolumeMonitor,
    disks: Arc<Mutex<Vec<Disk>>>,
    volumes: Arc<Mutex<Vec<AppVolume>>>,
}

impl VolumeMonitor {
    pub fn new() -> Self {
        let volume_monitor = gio::VolumeMonitor::get();
        
        Self {
            volume_monitor,
            disks: Arc::new(Mutex::new(Vec::new())),
            volumes: Arc::new(Mutex::new(Vec::new())),
        }
    }
    
    pub fn start(&self, sender: glib::Sender<()>) {
        let monitor = self.volume_monitor.clone();
        let disks = self.disks.clone();
        let volumes = self.volumes.clone();
        
        // Connecter les signaux
        monitor.connect_volume_added(move |_, volume| {
            let volume = AppVolume::from(&volume);
            volumes.lock().unwrap().push(volume);
            sender.send(()).unwrap();
        });
        
        monitor.connect_volume_removed(move |_, volume| {
            let mut volumes = volumes.lock().unwrap();
            volumes.retain(|v| v.native != volume);
            sender.send(()).unwrap();
        });
        
        monitor.connect_mount_added(move |_, mount| {
            sender.send(()).unwrap();
        });
        
        monitor.connect_mount_removed(move |_, mount| {
            sender.send(()).unwrap();
        });
        
        // Chargement initial
        self.refresh();
    }
    
    pub fn refresh(&self) {
        let mut disks = self.disks.lock().unwrap();
        let mut volumes = self.volumes.lock().unwrap();
        
        disks.clear();
        volumes.clear();
        
        // Charger les volumes
        for volume in self.volume_monitor.volumes() {
            volumes.push(AppVolume::from(&volume));
        }
        
        // Charger les montages
        for mount in self.volume_monitor.mounts() {
            if let Some(root) = mount.root() {
                if let Some(uri) = root.uri() {
                    // Extraire les infos du disque
                    if let Some(disk) = self.extract_disk_info(&mount, &uri) {
                        disks.push(disk);
                    }
                }
            }
        }
    }
    
    fn extract_disk_info(&self, mount: &gio::Mount, uri: &str) -> Option<Disk> {
        // Implémentation de l'extraction des infos disque
        // ...
        unimplemented!()
    }
    
    pub fn get_disks(&self) -> Vec<Disk> {
        self.disks.lock().unwrap().clone()
    }
    
    pub fn get_volumes(&self) -> Vec<AppVolume> {
        self.volumes.lock().unwrap().clone()
    }
}
```

#### 3.2 Gestion des Paramètres

**src/services/settings.rs** :
```rust
use gio::prelude::*;
use gtk4::glib;

const SCHEMA_ID: &str = "io.github.yannmasoch.nautilus-my-computer";

#[derive(Debug, Clone)]
pub struct AppSettings {
    pub start_on_disks: bool,
    pub default_view: String,
    pub column_view_sort_by: String,
    pub column_view_sort_reversed: bool,
    pub show_system_partitions: bool,
    pub visibility_system: i32,
    pub visibility_removable: i32,
    pub visibility_disc: i32,
    pub visibility_network: i32,
    pub sidebar_show_home: bool,
    pub sidebar_show_recent: bool,
    pub sidebar_show_starred: bool,
    pub sidebar_show_network: bool,
    pub sidebar_show_trash: bool,
    pub color_mode: String,
}

impl Default for AppSettings {
    fn default() -> Self {
        Self {
            start_on_disks: true,
            default_view: "native".to_string(),
            column_view_sort_by: "name".to_string(),
            column_view_sort_reversed: false,
            show_system_partitions: false,
            visibility_system: 0,
            visibility_removable: 0,
            visibility_disc: 0,
            visibility_network: 0,
            sidebar_show_home: true,
            sidebar_show_recent: true,
            sidebar_show_starred: true,
            sidebar_show_network: true,
            sidebar_show_trash: true,
            color_mode: "system".to_string(),
        }
    }
}

pub struct SettingsManager {
    settings: gio::Settings,
}

impl SettingsManager {
    pub fn new() -> Option<Self> {
        match gio::Settings::new(SCHEMA_ID) {
            Ok(settings) => Some(Self { settings }),
            Err(_) => None,
        }
    }
    
    pub fn load(&self) -> AppSettings {
        let mut settings = AppSettings::default();
        
        settings.start_on_disks = self.settings.get::<bool>("start-on-disks");
        settings.default_view = self.settings.get::<String>("default-view");
        settings.column_view_sort_by = self.settings.get::<String>("column-view-sort-by");
        settings.column_view_sort_reversed = self.settings.get::<bool>("column-view-sort-reversed");
        settings.show_system_partitions = self.settings.get::<bool>("show-system-partitions");
        settings.visibility_system = self.settings.get::<i32>("visibility-system");
        settings.visibility_removable = self.settings.get::<i32>("visibility-removable");
        settings.visibility_disc = self.settings.get::<i32>("visibility-disc");
        settings.visibility_network = self.settings.get::<i32>("visibility-network");
        settings.sidebar_show_home = self.settings.get::<bool>("sidebar-show-home");
        settings.sidebar_show_recent = self.settings.get::<bool>("sidebar-show-recent");
        settings.sidebar_show_starred = self.settings.get::<bool>("sidebar-show-starred");
        settings.sidebar_show_network = self.settings.get::<bool>("sidebar-show-network");
        settings.sidebar_show_trash = self.settings.get::<bool>("sidebar-show-trash");
        settings.color_mode = self.settings.get::<String>("color-mode");
        
        settings
    }
    
    pub fn connect_changed<F>(&self, callback: F) 
    where
        F: Fn(&str) + 'static,
    {
        self.settings.connect_changed(move |settings, key| {
            callback(key);
        });
    }
}
```

### Phase 4 : Vues (4-5 jours)

#### 4.1 Vue Panneau My Computer

**src/views/my_computer.rs** :
```rust
use gtk4::{
    prelude::*,
    Box, FlowBox, FlowBoxChild, GestureClick, Label, 
    Picture, ProgressBar, ScrolledWindow, Stack, Widget,
};
use libadwaita::prelude::*;
use crate::models::Disk;

pub struct MyComputerPanel {
    pub container: Stack,
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
        
        let scrolled = ScrolledWindow::builder()
            .child(&flow_box)
            .hscrollbar_policy(gtk4::PolicyType::Automatic)
            .vscrollbar_policy(gtk4::PolicyType::Automatic)
            .build();
        
        let container = Stack::builder()
            .build();
        
        Self {
            container,
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
        
        self.flow_box.show();
    }
    
    fn create_disk_card(&self, disk: &Disk) -> FlowBoxChild {
        let vbox = Box::new(gtk4::Orientation::Vertical, 6);
        
        // Icône
        let icon = Picture::new();
        icon.set_icon_name(Some(&disk.icon_name));
        icon.set_pixel_size(48);
        
        // Nom
        let name_label = Label::new(Some(&disk.name));
        name_label.set_halign(gtk4::Align::Center);
        name_label.set_ellipsize(gtk4::pango::EllipsizeMode::End);
        
        // Barre d'utilisation
        let bar = ProgressBar::new();
        bar.set_fraction(disk.usage_percentage() / 100.0);
        bar.set_show_text(true);
        bar.set_text(Some(&format!("{}%", disk.usage_percentage().round())));
        
        // Info
        let info_label = Label::new(Some(&format!(
            "{} / {}",
            format_size(disk.used_size),
            format_size(disk.total_size)
        )));
        info_label.set_halign(gtk4::Align::Center);
        info_label.set_css_classes(&["dim-label"]);
        
        vbox.append(&icon);
        vbox.append(&name_label);
        vbox.append(&bar);
        vbox.append(&info_label);
        
        let child = FlowBoxChild::new();
        child.set_child(Some(&vbox));
        child.set_activatable(true);
        
        child
    }
    
    pub fn connect_activation<F>(&self, callback: F)
    where
        F: Fn(&Disk) + 'static,
    {
        self.flow_box.connect_child_activated(move |_, child| {
            if let Some(index) = child.index() {
                // Trouver le disque correspondant
                // ...
            }
        });
    }
}

fn format_size(bytes: u64) -> String {
    const KB: u64 = 1024;
    const MB: u64 = KB * 1024;
    const GB: u64 = MB * 1024;
    const TB: u64 = GB * 1024;
    
    if bytes >= TB {
        format!("{:.1} TB", bytes as f64 / TB as f64)
    } else if bytes >= GB {
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

#### 4.2 Vue Colonne

**src/views/column.rs** :
```rust
use gtk4::{prelude::*, ColumnView, ListBox, SingleSelection};
use gio::ListStore;
use crate::models::{Disk, AppVolume};

pub struct ColumnViewManager {
    column_view: ColumnView,
    model: ListStore,
    selection: SingleSelection,
}

impl ColumnViewManager {
    pub fn new() -> Self {
        let model = ListStore::new(Disk::static_type());
        let selection = SingleSelection::new(Some(&model));
        
        let column_view = ColumnView::new(Some(&selection));
        
        Self {
            column_view,
            model,
            selection,
        }
    }
    
    pub fn populate(&mut self, disks: Vec<Disk>) {
        self.model.remove_all();
        
        for disk in disks {
            self.model.append(&disk);
        }
    }
    
    pub fn set_sort(&self, sort_by: &str, reversed: bool) {
        // Implémenter le tri
    }
}
```

### Phase 5 : Intégration Nautilus (2-3 jours)

#### 5.1 Bindings Nautilus

Créer **nautilus-rs** :

**nautilus-rs/Cargo.toml** :
```toml
[package]
name = "nautilus-rs"
version = "0.1.0"
edition = "2021"

[dependencies]
gtk4 = "0.8"
glib = "0.18"
gio = "0.18"
```

**nautilus-rs/src/lib.rs** :
```rust
#![allow(unused_imports)]

pub mod prelude;
pub mod menu_provider;
pub mod window;
pub mod file_info;

pub use menu_provider::MenuProvider;
pub use window::Window;
pub use file_info::FileInfo;
```

**nautilus-rs/src/menu_provider.rs** :
```rust
use gtk4::glib::{self, subclass::prelude::*, Properties};
use gtk4::prelude::*;

mod imp {
    use super::*;
    
    #[derive(Default)]
    pub struct MenuProvider;
    
    #[glib::object_subclass]
    impl ObjectSubclass for MenuProvider {
        const NAME: &'static str = "NautilusMenuProvider";
        type Type = super::MenuProvider;
    }
    
    impl ObjectImpl for MenuProvider {}
}

glib::wrapper! {
    pub struct MenuProvider(
        ObjectSubclass<imp::MenuProvider>
    ) @extends gtk4::Widget;
}

pub trait MenuProviderImpl: ObjectSubclass {
    fn menu_provider_get_background_items(&self, window: &super::Window) -> Vec<MenuItem> {
        Vec::new()
    }
    
    fn menu_provider_get_context_items(&self, window: &super::Window, files: &[FileInfo]) -> Vec<MenuItem> {
        Vec::new()
    }
}
```

#### 5.2 Implémentation MenuProvider

**src/extension/menu.rs** :
```rust
use gtk4::glib::{self, subclass::prelude::*};
use gtk4::prelude::*;
use nautilus_rs::prelude::*;
use crate::extension::MyComputerExtension;

#[gtk4::template_callback]
fn on_menu_item_activated(_item: &nautilus_rs::MenuItem, _ext: &MyComputerExtension) {
    // Gérer l'activation du menu
}

impl MenuProviderImpl for MyComputerExtension {
    fn menu_provider_get_background_items(&self, window: &nautilus_rs::Window) -> Vec<nautilus_rs::MenuItem> {
        let mut items = Vec::new();
        
        // Ajouter l'item "My Computer Settings"
        if let Some(menu_item) = self.create_settings_menu_item(window) {
            items.push(menu_item);
        }
        
        items
    }
    
    fn menu_provider_get_context_items(&self, window: &nautilus_rs::Window, files: &[nautilus_rs::FileInfo]) -> Vec<nautilus_rs::MenuItem> {
        let mut items = Vec::new();
        
        // Ajouter des items contextuels si nécessaire
        
        items
    }
}
```

### Phase 6 : Intégration Sidebar (2 jours)

**src/extension/sidebar.rs** :
```rust
use gtk4::{prelude::*, ListBox, ListBoxRow, Image, Label};
use nautilus_rs::Window;
use crate::models::PlaceEntry;

pub struct SidebarManager {
    listbox: ListBox,
    places: Vec<PlaceEntry>,
}

impl SidebarManager {
    pub fn new() -> Self {
        let listbox = ListBox::new();
        listbox.set_selection_mode(gtk4::SelectionMode::Single);
        
        Self {
            listbox,
            places: Vec::new(),
        }
    }
    
    pub fn inject_computer_row(&mut self, window: &Window) {
        // Injecter la ligne "Computer" dans la sidebar
        // Cela nécessite d'accéder au widget tree de Nautilus
        
        // 1. Trouver le sidebar container
        // 2. Créer notre ListBox avec le Computer row
        // 3. L'insérer au bon endroit
        
        let computer_row = self.create_place_row(&PlaceEntry::computer());
        self.listbox.append(&computer_row);
    }
    
    fn create_place_row(&self, place: &PlaceEntry) -> ListBoxRow {
        let hbox = gtk4::Box::new(gtk4::Orientation::Horizontal, 6);
        
        let icon = Image::new();
        icon.set_icon_name(Some(&place.icon));
        icon.set_pixel_size(16);
        
        let label = Label::new(Some(&place.label));
        label.set_halign(gtk4::Align::Start);
        
        hbox.append(&icon);
        hbox.append(&label);
        
        let row = ListBoxRow::new();
        row.set_child(Some(&hbox));
        row.set_activatable(true);
        row.set_selectable(true);
        
        row
    }
    
    pub fn connect_activation<F>(&self, callback: F)
    where
        F: Fn(&PlaceEntry) + 'static,
    {
        self.listbox.connect_row_activated(move |_, row| {
            if let Some(child) = row.child() {
                // Extraire le PlaceEntry depuis la row
                // ...
            }
        });
    }
}
```

### Phase 7 : Gestion des Événements (2 jours)

**src/extension/events.rs** :
```rust
use gtk4::glib::{self, MainContext, Receiver, Sender};
use gtk4::prelude::*;
use std::sync::mpsc;
use crate::{services::VolumeMonitor, extension::MyComputerExtension};

impl MyComputerExtension {
    pub fn setup_event_handlers(&self) {
        // Canal pour les mises à jour
        let (sender, receiver) = MainContext::channel(glib::PRIORITY_DEFAULT);
        
        // Démarrer le monitor
        let monitor = VolumeMonitor::new();
        monitor.start(sender);
        
        // Recevoir les mises à jour dans le main thread
        receiver.attach(
            None,
            glib::clone!(@weak self as ext => move |_| {
                ext.on_volume_change();
                glib::ControlFlow::Continue
            })
        );
        
        // Connecter les signaux Nautilus
        self.connect_nautilus_signals();
    }
    
    fn on_volume_change(&self) {
        // Rafraîchir les vues
        self.refresh_all_views();
    }
    
    fn connect_nautilus_signals(&self) {
        // Se connecter aux signaux de la fenêtre Nautilus
        // - window-added
        // - active-window-changed
        // - etc.
    }
    
    pub fn refresh_all_views(&self) {
        // Mettre à jour toutes les vues ouvertes
        // ...
    }
}
```

### Phase 8 : Navigation et Ouverture (1-2 jours)

**src/extension/navigation.rs** :
```rust
use gtk4::glib;
use nautilus_rs::{Window, FileInfo};
use gio::File;

impl MyComputerExtension {
    pub fn open_location(&self, uri: &str, window: Option<&Window>) {
        let file = File::for_uri(uri);
        
        if let Some(win) = window {
            // Ouvrir dans la fenêtre existante
            self.open_in_window(&file, win);
        } else {
            // Ouvrir dans une nouvelle fenêtre
            self.open_in_new_window(&file);
        }
    }
    
    pub fn open_in_new_tab(&self, uri: &str, window: &Window) {
        let file = File::for_uri(uri);
        // Ouvrir dans un nouvel onglet
    }
    
    pub fn open_in_new_window(&self, uri: &str) {
        let file = File::for_uri(uri);
        
        if let Some(app) = gtk4::Application::default() {
            app.activate();
            
            // Attendre que la fenêtre soit prête
            glib::idle_add_once(glib::clone!(@strong file => move || {
                // Naviguer vers l'URI
            }));
        }
    }
    
    pub fn open_computer_view(&self, window: &Window) {
        self.open_location("computer:///", Some(window));
    }
}
```

### Phase 9 : Préférences (1-2 jours)

**src/extension/preferences.rs** :
```rust
use gtk4::{prelude::*, Window, Dialog};
use libadwaita::{prelude::*, PreferencesWindow};
use crate::services::SettingsManager;

pub struct PreferencesDialog {
    window: PreferencesWindow,
    settings: SettingsManager,
}

impl PreferencesDialog {
    pub fn new(parent: &Window) -> Self {
        let window = PreferencesWindow::new();
        window.set_transient_for(Some(parent));
        window.set_modal(true);
        window.set_title(Some("My Computer Settings"));
        
        let settings = SettingsManager::new().unwrap();
        
        // Créer les pages de préférences
        let general_page = Self::create_general_page(&settings);
        let sidebar_page = Self::create_sidebar_page(&settings);
        let view_page = Self::create_view_page(&settings);
        
        window.add(&general_page);
        window.add(&sidebar_page);
        window.add(&view_page);
        
        Self { window, settings }
    }
    
    fn create_general_page(settings: &SettingsManager) -> libadwaita::PreferencesPage {
        let page = libadwaita::PreferencesPage::new();
        page.set_title(Some("General"));
        
        // Start on Computer view
        let start_switch = gtk4::Switch::new();
        start_switch.set_active(settings.load().start_on_disks);
        start_switch.connect_state_notify(glib::clone!(@weak settings => move |switch| {
            settings.settings.set::<bool>("start-on-disks", switch.is_active());
        }));
        
        let start_row = libadwaita::ActionRow::new();
        start_row.set_title(Some("Start on Computer view"));
        start_row.add_suffix(&start_switch);
        
        page.add(&start_row);
        
        page
    }
    
    pub fn show(&self) {
        self.window.present();
    }
}
```

### Phase 10 : Finalisation (2-3 jours)

#### 10.1 Packaging

**Cargo.toml** :
```toml
[package]
name = "nautilus-my-computer-rs"
version = "0.13.4"
authors = ["Yann Masoch"]
license = "MIT"

[lib]
name = "nautilus_my_computer"
crate-type = ["cdylib"]

[dependencies]
gtk4 = { version = "0.8", features = ["v4_10"] }
gtk4-layer-shell = "0.2"
libadwaita = "0.5"
glib = "0.18"
gio = "0.18"
gobject-subclass = "0.1"
once_cell = "1.18"
lazy_static = "1.4"
regex = "1.10"
nautilus-rs = { path = "../nautilus-rs" }

[build-dependencies]
glib-build-tools = "0.18"
```

#### 10.2 Fichier .plugin

**data/nautilus-my-computer-rs.plugin** :
```ini
[X-Nautilus]
Extensions=nautilus-my-computer-rs

[nautilus-my-computer-rs]
Module=nautilus-my-computer-rs
IAge=43
```

#### 10.3 Build et Installation

**build.rs** :
```rust
use glib_build_tools::compile_resources;

fn main() {
    // Compiler les ressources
    compile_resources(
        "data",
        "data/resources.gresource",
        "nautilus-my-computer-rs",
    );
    
    println!("cargo:rerun-if-changed=data/");
}
```

**Makefile** :
```makefile
.PHONY: all build install uninstall clean

all: build

build:
	cargo build --release

install: build
	mkdir -p ~/.local/lib/nautilus/extensions-4/
	cp target/release/libnautilus_my_computer.so ~/.local/lib/nautilus/extensions-4/
	cp data/nautilus-my-computer-rs.plugin ~/.local/lib/nautilus/extensions-4/
	cp data/io.github.yannmasoch.nautilus-my-computer.gschema.xml ~/.local/share/glib-2.0/schemas/
	glib-compile-schemas ~/.local/share/glib-2.0/schemas/
	nautilus -q

uninstall:
	rm -f ~/.local/lib/nautilus/extensions-4/libnautilus_my_computer.so
	rm -f ~/.local/lib/nautilus/extensions-4/nautilus-my-computer-rs.plugin
	rm -f ~/.local/share/glib-2.0/schemas/io.github.yannmasoch.nautilus-my-computer.gschema.xml
	glib-compile-schemas ~/.local/share/glib-2.0/schemas/

clean:
	cargo clean
```

---

## 📊 Planification et Estimation

| Phase | Tâches | Durée | Priorité |
|-------|--------|-------|----------|
| 1 | Infrastructure Rust | 2 jours | ⭐⭐⭐⭐⭐ |
| 2 | Modèles de données | 3 jours | ⭐⭐⭐⭐⭐ |
| 3 | Services (monitor, settings) | 4 jours | ⭐⭐⭐⭐⭐ |
| 4 | Vues (panneau, colonne) | 5 jours | ⭐⭐⭐⭐⭐ |
| 5 | Bindings Nautilus | 3 jours | ⭐⭐⭐⭐⭐ |
| 6 | Intégration sidebar | 2 jours | ⭐⭐⭐⭐ |
| 7 | Gestion événements | 2 jours | ⭐⭐⭐⭐ |
| 8 | Navigation | 2 jours | ⭐⭐⭐⭐ |
| 9 | Préférences | 2 jours | ⭐⭐⭐ |
| 10 | Finalisation | 3 jours | ⭐⭐⭐ |
| **Total** | | **~28 jours** | |

---

## ⚠️ Défis et Solutions

### 1. Bindings Nautilus Manquants

**Problème** : Il n'existe pas de bindings Rust officiels pour Nautilus.

**Solution** : 
- Créer un crate `nautilus-rs` avec `gir`
- Ou utiliser `gtk4::glib::Object` avec des calls FFI manuels
- Ou contribuer aux bindings officiels gtk-rs

### 2. Gestion de la Mémoire

**Problème** : Python utilise la gestion mémoire automatique, Rust nécessite une gestion explicite.

**Solution** :
- Utiliser `Rc<RefCell<T>>` pour les objets partagés
- Utiliser `glib::Object` qui gère automatiquement les refcounts
- Utiliser `WeakRef` pour éviter les cycles

### 3. Intégration avec le Widget Tree de Nautilus

**Problème** : Accéder aux widgets internes de Nautilus nécessite des hacks.

**Solution** :
- Utiliser `gtk4::Widget::parent()` pour naviguer dans la hiérarchie
- Utiliser les noms de classes CSS pour identifier les widgets
- Se baser sur l'API publique de Nautilus

### 4. Gestion Asynchrone

**Problème** : Python utilise GLib.idle_add, Rust a son propre système async.

**Solution** :
- Utiliser `glib::idle_add` et `glib::timeout_add`
- Ou utiliser `tokio` avec `glib::MainContext`
- Éviter `async/await` pour l'UI GTK

### 5. Compatibilité des Versions

**Problème** : Nautilus évolue rapidement, les API changent.

**Solution** :
- Détecter la version de Nautilus à l'exécution
- Utiliser des features flags dans Cargo.toml
- Fournir des fallbacks pour les anciennes versions

---

## 🎯 Livrables

1. ✅ **Code source Rust** : Projet complet `nautilus-my-computer-rs`
2. ✅ **Bindings Nautilus** : Crate `nautilus-rs`
3. ✅ **Documentation** : README, commentaires dans le code
4. ✅ **Packaging** : Makefile, .plugin, .gschema.xml
5. ✅ **Tests** : Tests unitaires et d'intégration
6. ✅ **Traductions** : Système de traduction (gettext)

---

## 📚 Ressources Utiles

- [GTK-rs Book](https://gtk-rs.org/docs/gtk4-rs/stable/latest/book/)
- [GTK4 API Reference](https://docs.gtk.org/gtk4/)
- [Nautilus Python API](https://python-nautilus.readthedocs.io/)
- [GIR (Binding Generator)](https://github.com/gtk-rs/gir)
- [Rust FFI Guide](https://doc.rust-lang.org/nomicon/ffi.html)
- [Glib-rs Documentation](https://gtk-rs.org/docs/glib/stable/latest/glib/)

---

## 💡 Conseils

1. **Commencez petit** : Implémentez d'abord une version minimale qui affiche juste "Hello World" dans Nautilus
2. **Testez fréquemment** : Vérifiez que l'extension se charge correctement dans Nautilus
3. **Utilisez les logs** : `log::debug!`, `log::error!` pour le débogage
4. **Documentez** : Commentez le code Rust pour expliquer la logique
5. **Respectez les conventions** : Suivez les conventions Rust (rustfmt, clippy)
6. **Gérez les erreurs** : Utilisez `anyhow::Result` et `anyhow::Context`
7. **Optimisez plus tard** : D'abord faire fonctionner, puis optimiser

---

## 🚀 Prochaines Étapes

1. **Cloner le dépôt original** : `git clone https://github.com/yannmasoch/nautilus-my-computer`
2. **Étudier le code Python** : Comprendre chaque module
3. **Créer le squelette Rust** : `cargo new nautilus-my-computer-rs`
4. **Ajouter les dépendances** : Modifier Cargo.toml
5. **Implémenter par phases** : Suivre le plan ci-dessus

---

*Bon courage avec la conversion ! 🦀*
