# Feature map

Every capability a layer manager is expected to have, where it lives in the design, and what
CADCraft has at `fc4e983`. "Engine" means a command exists that does it; "UI" means today's
window shows it.

| Capability | In the design | Engine today | UI today | Work |
|---|---|---|---|---|
| List layers with all properties | Table | `layer`, `layer.set` | yes | redesign |
| Resize columns; fit / spread all | Column menu ⋯ | – | partial | UI |
| Show/hide optional columns | Column menu ⋯ | – | – | UI |
| Sort by any column, reverse | Header click | – | name only | UI |
| Search with wildcards | Search field | – | yes (`crate_wild`) | keep |
| New layer; several in a row (⌥N) | New Layer button | `layer.new` | yes | keep |
| New layer frozen in every viewport | Toolbar icon | `layer.new` + `vplayer` | – | UI |
| Delete selected (unused only) | Toolbar, Delete key | `layer.delete` | yes | guard text |
| Make current (double-click) | Row, toolbar, inspector | `layer.current` | yes | keep |
| On/off, freeze/thaw, lock/unlock, plot | Row toggles | `layer.set` | yes | multi-select |
| Colour, linetype, lineweight, transparency | Row + inspector | `layer.set` | yes | redesign |
| Load a standard linetype while picking | Linetype dropdown | `linetype` load | partial | UI |
| Plot style (named tables) | Inspector | `layer.set plotStyle` | yes | move |
| Frozen in new viewports | Inspector / column | `layer.set newVpFreeze` | yes | move |
| Description | Column + inspector | `layer.set description` | yes | keep |
| Locked-layer fading | Settings (decision) | – | – | engine + UI |
| Multi-select and apply to all | Table | (per layer) | – | UI, one undo |
| Built-in filters All / Used | Sidebar | – | yes | keep |
| Filter: from references | Sidebar | – | – | UI |
| Filter: viewport overrides | Sidebar | – | – | UI |
| Rule (property) filters, saved | Sidebar + rule editor | – | – | engine + UI |
| Group filters, drag rows in | Sidebar | – | – | engine + UI |
| Group from selected objects | "+" menu | – | – | engine + UI |
| Invert filter | Sidebar checkbox | – | – | UI |
| Layer states: save, restore, delete, rename | State ▾ | `layerstate.*` | partial | redesign |
| Layer states keep viewport overrides | State ▾ | – | – | engine |
| Layer states import/export | State ▾ (`.cclayers`) | – | – | engine + UI |
| VP freeze in a viewport | Viewport scope, Frz column | `vplayer freeze/thaw` | yes | keep |
| VP colour | Viewport scope, Color | `vplayer colors` (not drawn/saved?) | yes | verify + fix |
| VP linetype, lineweight, transparency | Viewport scope | – | – | engine + UI |
| Mark overrides; remove overrides | Dots, "Differs here … Remove", filter | – | – | UI + `vplayer clear` |
| Isolate / unisolate | Inspector | `layiso`, `layuniso` | – | UI |
| Merge into another layer | Inspector | `laymrg` | – | UI |
| Select objects on a layer | Inspector | selection API | – | UI |
| Reference (xref) layers | Dimmed prefix, filter | session-only overrides | partial | UI |
| Object count per layer | Inspector | – | – | UI (count in doc) |
