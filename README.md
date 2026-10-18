# Source assets: ts_central + weapons + Gordon

These are the original GoldSrc assets, pulled out of `Half-Life/` and decompiled. They're the input for the Godot rebuild. Nothing in here is "modernised" yet. `../new assets/` is still empty and is meant for the rebuilt versions.

## Layout

```
assets/
  ts_central.map / .jmf / .obj / .mtl   map, decompiled earlier with bspguy (untouched)
  ts_central_emb.wad                    embedded BSP textures
  textures/        145 x 24-bit BMP     map textures, as exported (untouched)
  textures_png/    145 x PNG            same, converted; '{' textures have pure-blue keyed to alpha
  map_deps/                             everything else ts_central.res pulls in
    models/sg_wntf/                     Schmung's model (models/Schmung/sg_wntf.mdl)
    sprites/                            flare1, WN_glow02 (additive) -> PNG + JSON, originals in source/
    decals/                             {crack2 {crack3 {ding10 {ding2 {scorch3 from ts/decals.wad (alpha = index)
    sound/ambience/                     apologize, mind_gap2, platform .wav
  models/<name>/
    source/        original .mdl (+ skin bmp / readme where they exist)
    decompiled/    <name>.qc, reference .smd per submodel, 8-bit .bmp textures (studiomdl-ready)
                   anims/*.smd, one per sequence (or per blend)
                   textures_png/*.png (masked textures get alpha)
                   <name>.json   bones, bind pose, attachments, hitboxes, bodygroups, sequences, events, skins
    preview/       Blender renders of the bind pose, all bodygroups shown at once
```

## Models

| Folder | Source file | Bones | Seqs | Textures | Bodygroups |
|---|---|---|---|---|---|
| `v_mp5k` | `ts/models/v_mp5k.mdl` | 49 | 11 | 11 | arms/gun parts, `scopeyz0r`, `silencerz0r` |
| `v_mp5sd` | `ts/models/v_mp5sd.mdl` | 39 | 7 | 12 | arms/gun parts, `scope` |
| `v_m4` | `ts/models/v_m4.mdl` | 50 | 16 | 15 | arms/gun parts, `silencer`, `flash`, `laser`, `ottica` (scope) |
| `v_beretta` | `ts/models/v_beretta.mdl` | 40 | 6 | 6 | arms/gun parts, `silencer`, `lam` |
| `v_berettas` | `ts/models/v_berettas.mdl` (akimbo) | 44 | 29 | 6 | arms/guns, 2x `silencer` |
| `gordon_ts` | `ts/models/player/gordon/gordon.mdl` ("Action Gordon" by Morfeo) | 22 | 171 (206 SMDs incl. blends) | 6 | 3 parts |
| `gordon_hl` | `valve/models/player/gordon/gordon.mdl` (stock HL1) | 42 | 77 (129 SMDs) | 2 | `body` (2 variants) |

TS player sequences are mostly 2-way aim blends (`blend XR -60 60`). They're written out as `<seq>_blend1/2.smd`.

## Decompiler notes

The tool is `../tools/goldsrc/gs_extract.py`. It runs on Blender 4.0's bundled Python, so nothing else had to be installed:

```
"C:\Program Files\Blender Foundation\Blender 4.0\4.0\python\bin\python.exe" -I gs_extract.py mdl <file.mdl> <out_dir>
```

Sub-commands: `mdl`, `wad`, `spr`, `bmp`. `preview_smd.py` renders a decompiled folder headless in Blender.

- Reference SMDs are in world space at the bind pose, wound counter-clockwise (the face winding Blender/Source expect), with UVs `u = s/w, v = 1 - t/h`.
- Linear root motion (`LX`/`LY`/`LZ`) is added back into the root bone of animation SMDs, the way Crowbar does it, so a recompile extracts the same motion.
- Duplicate sequence names (`v_mp5k` has four `shoot1`) are kept in the QC because weapons play sequences by index. The files get a `_seqN` suffix.
- Checked visually (renders in `preview/`) and numerically (no stray bone jumps). The one jump found is the M4 `Bullet` bone teleporting during reload, which is intentional.
- **Not yet checked:** recompiling the QCs with studiomdl to confirm a byte-level round trip.
