# Blender Internals Notes

A small collection of **source-backed notes** for understanding how Blender starts up, builds, and routes runtime state through the window-manager stack.

These documents are written to accompany the local `blender_fork/` source tree and focus on **entry points, bootstrapping, context, events, notifiers, and the Window Manager**.

---

## 📚 Document Index

| Document                                                   | Focus                 | Summary                                                                                                                           |
| ---------------------------------------------------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [`Blender_App_Project.md`](./Blender_App_Project.md)       | Top-level build graph | Reviews the root `CMakeLists.txt`, project structure, and how the final `blender` target is assembled.                            |
| [`Blender_Bootsrapping.md`](./Blender_Bootsrapping.md)     | Application startup   | Follows Blender startup from `main()` / `wWinMain()` into `WM_init()` and `WM_main()`, including CLI parsing and background mode. |
| [`Blender_Context.md`](./Blender_Context.md)               | `bContext`            | Explains what `bContext` stores, where it is created, how it is populated, and where it is used.                                  |
| [`Blender_Window_Manager.md`](./Blender_Window_Manager.md) | WM subsystem          | Covers the role of the Window Manager, its lifecycle, event dispatch, operators, jobs, and runtime services.                      |
| [`Blender_Events.md`](./Blender_Events.md)                 | Event system          | Traces how OS/GHOST input becomes `wmEvent` queue data and how events are processed in `WM_main()`.                               |
| [`Blender_Notifiers.md`](./Blender_Notifiers.md)           | Notifier system       | Explains how change notifications are queued and dispatched to screens, areas, and regions for refresh/update.                    |

> **Note:** The file name `Blender_Bootsrapping.md` keeps the current spelling used in this folder.

---

## 🧭 Suggested Reading Order

If you are new to Blender internals, read the notes in this order:

1. [`Blender_App_Project.md`](./Blender_App_Project.md)
2. [`Blender_Bootsrapping.md`](./Blender_Bootsrapping.md)
3. [`Blender_Context.md`](./Blender_Context.md)
4. [`Blender_Window_Manager.md`](./Blender_Window_Manager.md)
5. [`Blender_Events.md`](./Blender_Events.md)
6. [`Blender_Notifiers.md`](./Blender_Notifiers.md)

This path moves from **build/setup** → **startup flow** → **runtime context** → **WM loop and event/update systems**.

---

## 🔎 Main Source Areas Referenced

These notes mainly cross-reference code from the local Blender source tree:

- `CMakeLists.txt`
- `source/creator/`
- `source/blender/blenkernel/`
- `source/blender/windowmanager/`
- `source/blender/editors/screen/`
- `tests/`

---

## 🎯 Purpose

This folder is intended as a practical study guide for:

- understanding Blender's startup path,
- navigating important source files faster,
- connecting high-level concepts to real implementation points,
- and building a clearer mental model of Blender's runtime architecture.
