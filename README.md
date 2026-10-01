# Aurora Notes

A notes app for **Aurora OS** and **Sailfish OS**, written in C++/Qt 5 and QML with Sailfish Silica. It keeps three kinds of notes in one list:

- ✏️ **Text notes**: a title and free-form text
- 🖍️ **Sketch notes**: freehand drawings made with your finger, saved as PNG images
- 🎙️ **Audio notes**: voice recordings made with the microphone, saved as WAV files, with playback in the app

Package ID: `ru.ivars.rozhleys.notes` · Version: 0.3.1 · License: BSD-3-Clause

## Features

- One main screen shows every note as a thumbnail in a two-column grid, newest changes first
- Create a note with the buttons in the footer (pen, marker or tape)
- Edit a note's title and content, or delete the note
- Record, play, pause and mute audio notes
- Translations in English and Russian
- A cover for the app switcher

## Architecture

The app follows an MVVM layout:

```
QML pages (qml/pages, qml/elements)
        │  context properties: _listViewModel, _textNoteViewModel,
        │                      _sketchNoteViewModel, _audioNoteViewModel
        ▼
ViewModels (src/viewmodels)   ← state shown to QML, Q_INVOKABLE actions
        ▼
Models (src/models)           ← one model per note type; handles media files
        ▼
DAO (src/dao/databasedao)     ← SQLite access; emits inserted/updated/removed
        ▼
DTOs (src/dto)                ← plain data structs
```

| Path | Purpose |
|------|---------|
| `src/main.cpp` | Starts the app, registers QML types and wires models to view models |
| `src/dao/` | `DatabaseDAO`: CRUD on the SQLite `media` table |
| `src/dto/` | `DatabaseEntry` (with the `NoteType` enum), `TextNote`, `SketchNote`, `AudioNote` |
| `src/models/` | Per-type models that read and write the media file and update the DB row |
| `src/viewmodels/` | `ListViewModel` (a `QAbstractListModel` for the grid) plus one view model per note type, all derived from `NoteViewModel` |
| `src/qmlitems/sketchitem.*` | `Sketch`, a `QQuickPaintedItem` drawing canvas exposed to QML |
| `src/audiorecorder.*` | `AudioRecorder`, a `QAudioRecorder` set to record PCM/WAV |
| `qml/` | UI: pages, reusable elements, the cover and SVG icons |
| `translations/` | Qt Linguist `.ts` files |
| `rpm/` | RPM spec and changelog templates |

### Data storage

All data lives under `~/Documents/notes/`:

```
notes/
├── notes.sqlite     # index of all notes
├── text/<uuid>.txt
├── sketch/<uuid>.png
└── audio/<uuid>.wav
```

The `media` table holds `id, type, title, media (file path), created, modified`. Note content is kept in a separate file, and the database stores only that file's path.

## Building

### Requirements

- Aurora OS SDK or Sailfish OS SDK
- CMake ≥ 3.5, C++17
- Qt 5: Core, Network, Qml, Gui, Quick, Sql, Multimedia, LinguistTools
- `auroraapp` (Aurora OS) or `sailfishapp` (Sailfish OS), found through pkg-config

### With the SDK

Open the project in the Aurora/Sailfish IDE (Qt Creator) and build the RPM for your target. The spec file detects the platform from `/etc/os-release` and passes it to CMake:

- **Aurora OS**: `cmake -DPLATFORM_ID=auroraos -GNinja`
- **Sailfish OS**: `cmake -DPLATFORM_ID=sailfishos` (Make)

### Manual CMake build (inside the build target)

```sh
cmake -B build -DPLATFORM_ID=auroraos   # or sailfishos
cmake --build build
```

`PLATFORM_ID` is required. It picks the platform library and sets a compile definition that `main.cpp` uses to choose between `Aurora::Application` and `SailfishApp`.

## Permissions

The app asks for these permissions in its `.desktop` file: `SecureStorage`, `Audio`, `Documents`, `MediaIndexing` and `Microphone`.

## Translations

Translation sources are in `translations/` (`notes.ts` and `notes-ru.ts`). CMake updates them from the C++ and QML sources and compiles them to `.qm` at build time.

## License

BSD 3-Clause © 2024 Ivars Rozhleys. See [LICENSE](LICENSE).
