# 🚀 QuickStart : Conversion Nautilus My Computer

## TL;DR

Vous voulez convertir **nautilus-my-computer** de Python vers Rust ? Voici comment commencer en **5 minutes**.

---

## 🎯 Étape 0 : Prérequis

### Installer Rust

```bash
# Installer rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Ajouter les outils
rustup component add rustfmt clippy

# Installer les dépendances de développement
sudo apt install libgtk-4-dev libadwaita-1-dev libnautilus-extension-dev
```

### Installer les outils GNOME

```bash
# Pour Ubuntu/Debian
sudo apt install nautilus gsettings-desktop-schemas glib-2.0

# Pour Fedora
sudo dnf install nautilus gsettings-desktop-schemas glib2-devel
```

---

## 📁 Étape 1 : Structure du Projet

```bash
# Créer le projet
cargo new nautilus-my-computer-rs --lib
cd nautilus-my-computer-rs

# Créer la structure
mkdir -p src/models src/services src/views src/widgets src/utils
mkdir -p data assets/icons

# Créer les fichiers principaux
touch src/lib.rs src/extension.rs src/models/mod.rs src/services/mod.rs
```

---

## 📦 Étape 2 : Cargo.toml

```toml
# Cargo.toml
[package]
name = "nautilus-my-computer-rs"
version = "0.13.4"
edition = "2021"

[lib]
name = "nautilus_my_computer"
crate-type = ["cdylib"]

[dependencies]
gtk4 = "0.8"
libadwaita = "0.5"
glib = "0.18"
gio = "0.18"
gobject-subclass = "0.1"
once_cell = "1.18"
anyhow = "1.0"
log = "0.4"
env_logger = "0.10"

[build-dependencies]
glib-build-tools = "0.18"
```

---

## 🎨 Étape 3 : Points d'Entrée Nautilus

Créer **src/lib.rs** :

```rust
use gtk4::glib;
use std::panic::catch_unwind;

mod extension;
use extension::MyComputerExtension;

// Nautilus appelle cette fonction pour initialiser
#[no_mangle]
pub extern "C" fn nautilus_module_initialize(
    _module: *mut glib::ffi::GTypeModule,
) -> *mut glib::gobject_ffi::GType {
    catch_unwind(|| {
        MyComputerExtension::static_type().to_glib()
    }).unwrap_or_else(|_| {
        eprintln!("Failed to initialize nautilus-my-computer-rs");
        std::ptr::null_mut()
    })
}

// Nautilus appelle cette fonction pour nettoyer
#[no_mangle]
pub extern "C" fn nautilus_module_shutdown() {
    // Nettoyage si nécessaire
}

// Nautilus appelle cette fonction pour lister les types
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

---

## 🏗️ Étape 4 : MyComputerExtension

Créer **src/extension.rs** :

```rust
use gtk4::glib::{self, subclass::prelude::*, Properties};
use gtk4::prelude::*;

mod imp {
    use super::*;
    
    #[derive(Properties, Default)]
    #[properties(wrapper_type = super::MyComputerExtension)]
    pub struct MyComputerExtension {
        #[property(get, set)]
        pub start_on_disks: glib::Cell<bool>,
    }
    
    #[glib::object_subclass]
    impl ObjectSubclass for MyComputerExtension {
        const NAME: &'static str = "MyComputerExtension";
        type Type = super::MyComputerExtension;
    }
    
    impl ObjectImpl for MyComputerExtension {}
}

glib::wrapper! {
    pub struct MyComputerExtension(
        ObjectSubclass<imp::MyComputerExtension>
    ) @extends gtk4::Widget;
}

impl MyComputerExtension {
    pub fn new() -> Self {
        glib::Object::builder()
            .property("start-on-disks", true)
            .build()
    }
}
```

---

## 🔍 Étape 5 : Fichier .plugin

Créer **data/nautilus-my-computer-rs.plugin** :

```ini
[X-Nautilus]
Extensions=nautilus-my-computer-rs

[nautilus-my-computer-rs]
Module=nautilus-my-computer-rs
IAge=43
```

---

## 📄 Étape 6 : Schéma GSettings

Copier le schéma original :

```bash
cp /workspace/nautilus-my-computer/io.github.yannmasoch.nautilus-my-computer.gschema.xml \
   data/io.github.yannmasoch.nautilus-my-computer.gschema.xml
```

---

## 🏃 Étape 7 : Build et Test

### Build

```bash
cargo build
```

### Installer

```bash
# Créer le répertoire si nécessaire
mkdir -p ~/.local/lib/nautilus/extensions-4/

# Copier la bibliothèque
cp target/debug/libnautilus_my_computer.so ~/.local/lib/nautilus/extensions-4/

# Copier le fichier plugin
cp data/nautilus-my-computer-rs.plugin ~/.local/lib/nautilus/extensions-4/

# Copier le schéma GSettings
cp data/*.gschema.xml ~/.local/share/glib-2.0/schemas/

# Compiler les schémas
glib-compile-schemas ~/.local/share/glib-2.0/schemas/
```

### Tester

```bash
# Redémarrer Nautilus
nautilus -q

# Lancer Nautilus avec logs
RUST_LOG=debug nautilus
```

---

## ✅ Vérification

### L'extension se charge-t-elle ?

```bash
# Vérifier que Nautilus charge l'extension
nautilus --version

# Vérifier les extensions chargées
ls ~/.local/lib/nautilus/extensions-4/

# Vérifier les logs
journalctl -f -o cat | grep nautilus
```

### Problèmes courants

| Problème | Solution |
|----------|----------|
| Extension non chargée | Vérifier le chemin dans .plugin |
| Symbole manquant | `cargo build --release` puis reinstaller |
| Nautilus plante | Vérifier les logs avec `RUST_BACKTRACE=1` |
| Fichier .so introuvable | Vérifier `LD_LIBRARY_PATH` |

---

## 🎯 Prochaines Étapes

Une fois que l'extension se charge sans erreur :

### 1. Ajouter la détection des volumes

```bash
# Lire le code Python original
less /workspace/nautilus-my-computer/nautilus_my_computer/main.py | head -200

# Implémenter VolumeMonitor
# (voir AGENT_NAUTILUS_RUST_CONVERSION.md)
```

### 2. Créer le panneau My Computer

```bash
# Lire la vue originale
less /workspace/nautilus-my-computer/nautilus_my_computer/my_computer_view.py | head -100

# Implémenter MyComputerPanel
# (voir PROMPT_NAUTILUS_RUST_CONVERSION.md)
```

### 3. Injecter dans la sidebar

```bash
# Lire l'injection sidebar
less /workspace/nautilus-my-computer/nautilus_my_computer/main.py | grep -A 50 "inject"

# Implémenter l'injection
# (voir les exemples dans la documentation)
```

---

## 📚 Documentation Complète

Pour aller plus loin, consultez :

| Document | Contenu |
|----------|---------|
| **[AGENT_NAUTILUS_RUST_CONVERSION.md](./AGENT_NAUTILUS_RUST_CONVERSION.md) | Guide complet de conversion |
| **[PROMPT_NAUTILUS_RUST_CONVERSION.md](./PROMPT_NAUTILUS_RUST_CONVERSION.md) | Prompt technique détaillé |
| **[RESUME_CONVERSION.md](./RESUME_CONVERSION.md) | Résumé et checklist |
| **[README_NAUTILUS_RUST.md](./README_NAUTILUS_RUST.md) | Documentation principale |

---

## 💡 Astuces

### 1. Debug rapide

```bash
# Builder en mode debug
cargo build

# Installer
cp target/debug/libnautilus_my_computer.so ~/.local/lib/nautilus/extensions-4/

# Tester
nautilus -q && RUST_LOG=debug nautilus
```

### 2. Vérifier les symboles

```bash
# Vérifier que les symboles sont exportés
nm -D target/debug/libnautilus_my_computer.so | grep nautilus_module
```

### 3. Nettoyer

```bash
# Désinstaller
rm ~/.local/lib/nautilus/extensions-4/libnautilus_my_computer.so
rm ~/.local/lib/nautilus/extensions-4/nautilus-my-computer-rs.plugin

# Redémarrer Nautilus
nautilus -q
```

---

## 🎉 Félicitations !

Vous avez maintenant une **extension Nautilus minimale en Rust** qui se charge !

Pour ajouter les fonctionnalités, suivez le guide dans **AGENT_NAUTILUS_RUST_CONVERSION.md**.

---

*Bon développement ! 🦀✨*
