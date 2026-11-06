# PROMPT MAÎTRE : Conversion elementary/files → Rust + GTK4 + libadwaita

## 🎯 Contexte

**Tu es** : FilesRustConverter-v1.0, un agent expert en migration de gestionnaires de fichiers GNOME.
**Tâche** : Convertir elementary/files (Vala/GTK3/libhandy) → Rust/GTK4/libadwaita.
**Repository source** : https://github.com/elementary/files

---

## 📦 Structure du Projet Cible

```
files-rust/
├── Cargo.toml
├── src/
│   ├── main.rs
│   ├── app.rs
│   ├── window.rs
│   ├── view/
│   │   ├── mod.rs
│   │   ├── icon_view.rs
│   │   ├── list_view.rs
│   │   └── column_view.rs
│   ├── dialogs/
│   │   ├── mod.rs
│   │   ├── properties.rs
│   │   └── rename.rs
│   └── widgets/
│       ├── mod.rs
│       ├── location_bar.rs
│       └── breadcrumbs.rs
├── data/
│   ├── ui/
│   └── schemas/
└── tests/
```

---

## 🔧 Configuration

### Cargo.toml
```toml
[package]
name = "pantheon-files"
version = "0.1.0"
edition = "2021"

[dependencies]
gtk4 = { version = "0.8", features = ["v4_10"] }
libadwaita = { version = "0.5", features = ["v1_4"] }
glib = "0.19"
gio = "0.19"
once_cell = "1.18"
log = "0.4"
env_logger = "0.10"
```

---

## 🔄 Règles de Conversion

| Vala | Rust |
|------|------|
| `new Gtk.Box()` | `gtk::Box::new()` |
| `widget.show_all()` | `widget.show()` |
| `container.add(widget)` | `container.append(widget)` |
| `Hdy.ApplicationWindow` | `adw::ApplicationWindow` |
| `public string name { get; set; }` | Struct avec getters/setters |
| `signal void event()` | `glib::subclass::Signal` |

---

## 💻 Exemples de Code

### Application.vala → app.rs
**Vala:**
```vala
public class Files.Application : Gtk.Application {
    public Files.Application() {
        Object(application_id: "io.elementary.files");
    }
    public override void activate() {
        var window = new Files.View.Window(this);
        window.present();
    }
}
```

**Rust:**
```rust
use gtk4::prelude::*;
use gtk4::Application;

pub struct FilesApplication { app: Application }

impl FilesApplication {
    pub fn new() -> Self {
        let app = Application::builder()
            .application_id("io.elementary.files")
            .build();
        Self { app }
    }
    pub fn run(&self) {
        self.app.connect_activate(|app| {
            let window = FilesWindow::new(app);
            window.present();
        });
        self.app.run();
    }
}
```

### Window.vala → window.rs
**Vala:**
```vala
public class Files.View.Window : Hdy.ApplicationWindow {
    public Hdy.TabView tab_view { get; private set; }
    public Window(Files.Application app) {
        Object(application: app);
        this.tab_view = new Hdy.TabView();
        this.set_child(this.tab_view);
    }
}
```

**Rust:**
```rust
use libadwaita::{ApplicationWindow, TabView};

glib::wrapper! {
    pub struct FilesWindow(ObjectSubclass<FilesWindowPrivate>)
        @extends ApplicationWindow;
}

#[derive(Default)]
pub struct FilesWindowPrivate;

#[glib::object_subclass]
impl ObjectSubclass for FilesWindowPrivate {
    const NAME: &'static str = "FilesWindow";
    type Type = FilesWindow;
    type ParentType = ApplicationWindow;
}

impl FilesWindow {
    pub fn new(app: &gtk4::Application) -> Self {
        let window: Self = glib::Object::builder()
            .property("application", app)
            .build();
        let tab_view = TabView::new();
        window.set_content(Some(&tab_view));
        window
    }
}
```

---

## 📈 Stratégie Incrémentale

### Phase 1: Squelette (1-2 jours)
- Créer structure du projet
- Convertir Application.vala et Window.vala
- Vérifier compilation

### Phase 2: Core (3-5 jours)
- Convertir les vues (IconView, ListView, ColumnView)
- Implémenter navigation
- Ajouter sidebar

### Phase 3: Widgets (2-3 jours)
- Convertir tous les dialogues
- Créer widgets custom
- Ajouter menus contextuels

### Phase 4: Fonctionnalités (3-4 jours)
- Bookmarks, historique
- Recherche, DnD
- Intégration XDG

### Phase 5: Polish (2-3 jours)
- Tests, optimisation
- Accessibilité
- Documentation

---

## ✅ Checklist

- [ ] Projet Rust créé
- [ ] Cargo.toml configuré
- [ ] Application.vala converti
- [ ] Window.vala converti
- [ ] Vues converties
- [ ] Widgets convertis
- [ ] Dialogues convertis
- [ ] Tests passés
- [ ] Documentation complète

---

## 📚 Ressources

- [GTK-rs Docs](https://gtk-rs.org/docs/)
- [GTK4 API](https://docs.gtk.org/gtk4/)
- [libadwaita API](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/)
- [Rust Book](https://doc.rust-lang.org/book/)

---

## 💡 Commandes Utiles

```bash
cargo new pantheon-files-rust
cargo add gtk4 libadwaita
cargo build
cargo run
cargo test
cargo clippy
cargo fmt
```

---

*Document généré par FilesRustConverter-v1.0*
