<div align="center">

# INU Core GTA

**Pure-Python library for GTA SA / VC / III file formats — read, write and audit game files without Blender.**

<p>
  <img src="https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Games-III%20%7C%20VC%20%7C%20SA-orange" alt="Games">
  <img src="https://img.shields.io/badge/License-GPL--3.0-blue" alt="License">
</p>

The format core of **[INU Tools](https://github.com/INU-ez/INU_Tools-GTA-Blender)**, published as a standalone library.

</div>

---

## Highlights

- **Native parsers and writers:** DFF / COL / TXD / IFP / IMG / IDE / IPL / FXP / CST, plus the `data/` text files. The only dependency is `numpy`.
- **Checked against the engine:** each writer has a pre-write audit built from what `gta_sa.exe` really reads and dereferences. Anything the game would crash on is reported by name, before the file is written.
- **Multi-game support:** GTA III, Vice City and San Andreas. The game is auto-detected from RW versions and TXD platform IDs. Surface IDs, IDE flags and ped masks are translated between games.
- **Byte-exact round-trips** where the format allows it, so re-saving a vanilla file doesn't change it.
- **Fast:** DXT1/DXT3/DXT5 are decoded and DXT1/DXT5 encoded with vectorised numpy. IMG directories are read without unpacking the archive.
- **Tested in production:** every import and export in INU Tools goes through this code.

## Format Support

| Format | Read | Write | Audit | What it is |
|---|:---:|:---:|:---:|---|
| **DFF** | ✅ | ✅ | ✅ | RenderWare clump: geometry, skinning, materials, 2DFX, UV animation |
| **COL** | ✅ | ✅ | ✅ | Collision: COLL / COL2 / COL3 / COL4 |
| **TXD** | ✅ | — | ✅ | Texture dictionary (D3D8 / D3D9, DXT, mobile containers); DXT encoder in `dxt` |
| **IFP** | ✅ | ✅ | ✅ | Animations: ANPK / ANP2 / ANP3 |
| **IMG** | ✅ | ✅ | — | Resource archive, VER1 / VER2 |
| **IDE** | ✅ | ✅ | ✅ | Object definitions (objs / tobj / anim / cars / peds / weap / hier / txdp / 2dfx) |
| **IPL** | ✅ | ✅ | ✅ | Placement, text and binary |
| **FXP** | ✅ | ✅ | ✅ | `effects.fxp` particle systems |
| **CST** | ✅ | ✅ | — | Steve M.'s Collision File Editor II text format |
| **timecyc.dat** | ✅ | ✅ | ✅ | Time cycles for SA, VC and III |
| **water.dat** | ✅ | ✅ | ✅ | Water quads |
| **map.zon / info.zon** | ✅ | ✅ | ✅ | Zones |
| **plants.dat** | ✅ | ✅ | ✅ | Procedural grass |
| **gta.dat** | ✅ | — | ✅ | Resource list (IDE / IPL / IMG paths) |

## Quick Start

```python
from core import img, dff, txd, col, mapdff_lint

IMG = r"C:\Games\GTA San Andreas\models\gta3.img"

# Read a model straight from the archive
clump = dff.read_dff(img.extract_file(IMG, "barrierblk.dff"))
print(len(clump.geometries), "geometries")

# Audit it against what gta_sa.exe expects
fatal, warnings = mapdff_lint.check_map_clump(clump, "barrierblk")

# Write it back
data = dff.write_dff(clump)

# Textures come out decoded to RGBA
for tex in txd.read_txd(img.extract_file(IMG, "barrierblk.txd")):
    print(tex.name, tex.width, tex.height)

# Collision: read, edit, write
models = col.read_col(img.extract_file(IMG, "veh_mods.col"))
data = col.write_col(models, target_game="SA")
```

Scan a folder for crash-prone files:

```python
from core import file_lint

for issue in file_lint.lint_dff(r"C:\mods\my_model.dff"):
    print(issue.format_short())
```

## Modules

<details>
<summary>Full module list</summary>

| Group | Modules |
|---|---|
| **Binary formats** | `dff`, `txd`, `txd_mobile`, `dxt`, `col`, `cst`, `ifp`, `img`, `rwbinary`, `texture_index` |
| **Text data** | `ide`, `ipl`, `gta_dat`, `timecyc`, `water`, `zon`, `plants_dat`, `fxp`, `mapsync/` |
| **Engine audits** | `mapdff_lint`, `skin_lint`, `col_lint`, `txd_lint`, `ifp_lint`, `textdata_lint`, `file_lint`, `map_lint`, `lint_profile` |
| **Multi-game** | `game_versions`, `surface_translate`, `ide_flag_translate`, `ped_mask_translate`, `vc_layers` |
| **Helpers** | `model_classify`, `tex_name`, `paths`, `validate`, `fs_ci` |
| **Map resources** | `col_library`, `img_remove`, `img_routing`, `ipl_geom`, `map_files`, `mapsync.groups`, `mapsync.ide_match` |

Modules added during the Blender / Max parity work:

| Module | Purpose |
|---|---|
| `col_library` | Edit one collision model inside a COL library, preserving other records. |
| `img_remove` | Plan archive removals while keeping TXDs and collisions used by other models. |
| `img_routing` | Route models to their own archives and resolve shared TXD destinations. |
| `ipl_geom` | Convert IPL cull-zone and garage geometry to and from scene bounds. |
| `map_files` | Gather text/stream IPLs, use game archive priority and index extraction caches. |
| `mapsync.groups` | Group several meshes of one model placement. |
| `mapsync.ide_match` | Match scene models to IDE rows using their own links and guarded IDs/names. |
| `fs_ci` | Resolve game paths regardless of letter case and compare file identities. |

`dxt_gpu` and `bitmap_diff` are used by the Blender addon and need Blender to run. Everything else imports without it.

</details>

## Installation

Clone the repository and put its root on `sys.path`:

```bash
git clone https://github.com/INU-ez/inu_gta_core.git
pip install numpy
```

```python
import sys
sys.path.insert(0, "path/to/inu_gta_core")
from core import dff
```

## Compatibility

| | |
|---|---|
| **Python** | 3.9+ |
| **Dependencies** | `numpy` |
| **Game** | GTA San Andreas (primary); Vice City and III |
| **OS** | Windows / Linux / macOS |

## Credits

- **[DragonFF](https://github.com/Parik27/DragonFF)** (Parik, GPL-3.0): reference for the DFF / COL parsers.
- **[RenderWare](https://en.wikipedia.org/wiki/RenderWare)**: format documentation for the GTA engine.
- **[re3 / reVC](https://github.com/halpz/re3)** and **gta-reversed**: engine references for the III / VC / SA audits.

**Author:** INU (Discord `1.n.u` · [server](https://discord.gg/sqtGAVTGdy))

**License:** [GPL-3.0](LICENSE)

## Compiled SA NODES graph editing (2026-10-01)

`core.paths_graph` rebuilds an edited local node graph, preserves original
directed links and outgoing regional links, creates vehicle navigation,
remaps incoming references and removes links to deleted nodes. New branches
use one lane each way; unambiguous subdivisions retain the original lanes
and direction. Navigation addresses use SA's area/index packing. Imported
NODES with zero sector padding, including empty regions, now parse their
link sections instead of falling back to an unknown raw tail.

Hosts must supply stable original vertex identities and all mesh categories.
When old physical IDs change, the Blender adapter requires all 64 regions
and validates every binary before writing. Unknown nonzero tails, ambiguous
coincident duplicates, foreign-stub deletion and capacity overflow are
rejected. Multi-file replacement is still not a filesystem transaction.
The Blender tests cover unchanged vanilla files and adding/deleting a
vehicle node across the full original SA map. Blender/Max/game GUI was not
launched; installed addons were not changed.


## Native Blender verification (2026-10-01)

Blender 5.1.2 was launched with factory settings in background mode and loaded
the addon directly from its repository. All 20 native checks passed without
skips: 11 compiled NODES scenarios and 9 IFP tests, including the 3 preview
checks that require native bpy. The NODES checks include real Extrude,
Subdivide, duplicate/delete, .blend save/reload and full 64-region export
from Object Mode and multi-object Edit Mode.

The native run found and fixed an adapter issue: Mesh Attribute.data is
unavailable in Edit Mode. Stable identities now use the live BMesh integer
layer there, including migration of an unchanged legacy scene. Core files
are unchanged by this adapter fix. The regular suite also passed again:
1807 passed, 4 skipped. Max and game runtime remain unverified; installed
addons were not updated.
