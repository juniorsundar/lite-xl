# Survey: Rust-native font backends

Date: 2026-10-17. Planning evidence for **Survey Rust-native font backends** in the FreeType Wayfinder effort. No backend is selected; no production dependency, font API, feature removal, fork or C fallback is authorized.

## Result in brief

There are several real Rust rasterizers, not one complete FreeType substitute. **Swash** supplies the broadest integrated rendering API of the candidates examined, but its public hint switch does not expose the editor's current hinting policy. **Skrifa + Zeno** is a plausible small composition with explicit hinter/target control, not a ready-made integration. **Fontdue** has grayscale **and subpixel RGB** rasterization but its inspected raster path does not implement the current hinting modes. **ab_glyph** supplies outline coverage rasterization, metrics and fractional placement, not a comparable hint/LCD/color-rendering pipeline.

The audited Linux/default-feature dependency union is **33 external Rust packages**, with no native font library/toolchain dependency found. This is not a whole-editor C-free claim or an all-target/all-feature guarantee. Rendering/metric equivalence, hostile-font bounds and performance remain unmeasured.

Next evidence should compare **hint controls, mono output, LCD behavior, metrics and synthetic styles**, not presume a winner from API names. Direct Skrifa deserves comparison because the current renderer forces automatic hinting; stock Swash chooses its hinter differently.

## Method and source identity

Released crates were identified through the official sparse index, then downloaded through Cargo. Version-specific docs and registry source/manifests were inspected. The recovered draft was a lead, not evidence to accept unchanged. crates.io/lib.rs HTML returned 403 in the reading environment; the official index, Cargo source and docs.rs remained available. Context7 returned only a Skrifa 0.41.0 index and general metadata documentation, not the required 0.44/0.48 hinting details; versioned source takes precedence.

| Component | Released version examined | Release signal / maintenance evidence | Declared licence |
|---|---|---|---|
| Swash | 0.2.10, 2026-07-17 | Multiple 0.2 releases through July 2026 in the official index | Apache-2.0 OR MIT |
| Skrifa | 0.48.0, 2026-10-03; also 0.44.0 via Swash | Fontations release activity through October 2026; upstream states no-panic and unsafe-forbidden goals | MIT OR Apache-2.0 |
| Zeno | 0.3.3, 2025-05-08 | Existing rasterizer used by current Swash; release date is not a future support guarantee | Apache-2.0 OR MIT |
| Fontdue | 0.9.4, 2026-07-29 | Manifest marks maintenance experimental; do not infer a bus factor from an author field | MIT OR Apache-2.0 OR Zlib |
| ab_glyph | 0.2.32, 2025-09-28 | README states stable-Rust maintenance policy; no current issue/response-time audit | Apache-2.0 |
| ttf-parser | 0.25.1 in the resolved consumers | Current upstream README says maintenance mode/bug fixes only and recommends Fontations. That notice is **not** in the released 0.25.1 README | MIT OR Apache-2.0 |

Official index: [Swash](https://index.crates.io/sw/as/swash), [Skrifa](https://index.crates.io/sk/ri/skrifa), [Zeno](https://index.crates.io/ze/no/zeno), [Fontdue](https://index.crates.io/fo/nt/fontdue), [ab_glyph](https://index.crates.io/ab/_g/ab_glyph). Licences and versions are also in the audited package manifests (appendix). Maintenance notice: [current ttf-parser README](https://raw.githubusercontent.com/harfbuzz/ttf-parser/main/README.md), contrasted with [v0.25.1 README](https://raw.githubusercontent.com/harfbuzz/ttf-parser/v0.25.1/README.md). These are maintenance signals, not support promises or an exhaustive bug-history audit.

## Editor contract this comparison must actually satisfy

The validated local inventory, at source revision `d9b3db31ab79339a60fe203ece364d17eb41786a`, is the authority for current behavior. Its source is already published on the repository's `rust-port` branch: [renderer](https://github.com/juniorsundar/lite-xl/blob/d9b3db31ab79339a60fe203ece364d17eb41786a/rust/src/renderer.rs), [Lua API](https://github.com/juniorsundar/lite-xl/blob/d9b3db31ab79339a60fe203ece364d17eb41786a/rust/src/renderer_api.rs), [stream](https://github.com/juniorsundar/lite-xl/blob/d9b3db31ab79339a60fe203ece364d17eb41786a/rust/src/ft.rs).

- freetype-rs 0.38.0, bundled FreeType 2.13.2. Load face index 0 from a configurable path; no exposed variation-axis or collection-face selector.
- Lua Font userdata/groups, copy/path/size/height/width/tabs/styles and deferred-draw lifetime must remain usable. Group fallback, tabs, missing-glyph policy and decoration rectangles already belong to the editor, not a library's shaping engine.
- **Advances and space metrics are unhinted**, separately from bitmap loading. Enabled bitmap hinting always uses **FORCE_AUTOHINT**. Slight selects LIGHT except mono; full selects NORMAL except mono; mono selects MONO. None disables hinting.
- LCD uses no filter with hinting none, or weights `10,40,70,40,10` (hex) when hinted. Fractional x uses three cached phases. Mono is actual monochrome rasterization, not just a label for grayscale.
- Synthetic bold emboldens X only, smoothing both axes, italic shears; decorations remain editor rectangles. The existing renderer does not add shaping, pair kerning or bidi.
- Bundled files are TrueType; configurable/custom fonts are in scope, but their required formats are **not yet decided**. Current color/bitmap/collection/variable coverage is not a blanket promise and not an approved exclusion.

A library implementing a similarly named option does not establish pixel/metric parity. The user must review measured differences.

## Capability comparison

**Present** means a released API or inspected implementation exists; **gap** means the inspected path lacks the required control; **unverified** means this survey does not settle it. None of these labels approves a feature loss.

| Responsibility | Swash 0.2.10 | Skrifa 0.48 + Zeno 0.3.3 | Fontdue 0.9.4 | ab_glyph 0.2.32 |
|---|---|---|---|---|
| OpenType outlines / collections | FontRef/indexing, TT and PostScript scaling; variations | Skrifa outline/collection/variation APIs; Zeno consumes paths, not font files | TTF/TTC/OTF parsing and collection_index; caches extracted outlines | TTF/OTF and explicit collection index; outline rasterizer |
| Hinting | Public boolean; **fixed internal target / AutoFallback** | Explicit Auto / Interpreter / AutoFallback and Mono/Smooth targets | **Gap:** inspected outline extraction and raster path have no hinter/mode API | **Gap:** inspected outline-to-coverage path has no hinter/mode API |
| Grayscale | Alpha mask | Zeno Alpha mask | Coverage bitmap | Coverage callback |
| LCD RGB | Zeno Subpixel/CustomSubpixel via Render | Same Zeno formats, adapter needed | **Present:** rasterize_subpixel / rasterize_indexed_subpixel, horizontal 3x RGB coverage | No comparable RGB API verified; not implied by fractional positioning |
| Mono | No native 1-bit Zeno Format; current high-level target not configurable | Skrifa Mono hint target exists; **rasterizer has no native 1-bit Format** | No monochrome raster mode verified | No monochrome raster mode verified |
| Current LCD filter / gamma | No equivalent configurable five-tap filter established | Not supplied merely by hint targets or subpixel offsets | 3x sampling is not the current filter; equivalence unverified | Not established |
| Fractional placement | Render offset and cached scaler data | Explicit path/Mask offset/transform | Raster path uses metrics_raw(...,0.0); no equivalent public phase/offset control found | Glyph scale/position API; coverage rasterizer |
| Synthetic styling | Embolden + affine transform; numeric/X-only equivalence needs care | Transform available; exact embolden/smoothing composition **unverified** | No current bold/smoothing/shear control on the inspected Font raster API | No current matching style API verified |
| Variable fonts | Variation APIs | Skrifa locations/axes | FontSettings has no axis setter; extracts default-location outlines | VariableFont::set_variation with feature enabled |
| Bitmap/color/SVG | Bitmap/color source APIs; exact format/version coverage must be tested | Parsing/color APIs do not themselves form a complete bitmap/color/SVG renderer with Zeno | Inspected raster path is outline-only | Raw bitmap/SVG access is **not** decoding/compositing |
| Cache ownership | ScaleContext scratch/LRU, borrowed FontRef and owned Image buffers | Caller owns file bytes, hint instances, paths and masks | Owned parsed geometry; caller owns raster results | Borrowed/owned font handles; caller owns mask/cache |
| Invalid-font errors | Many operations collapse missing/error to Option/bool | Checked parsing/DrawError; allocation/time bounds still need review | Font::from_bytes is fallible; indexed raster APIs assume valid IDs | Fallible font construction, optional outline/image data |
| Ready FreeType-equivalent integration | **No** | **No: proposed composition** | **No** | **No** |

### Why stock Swash hinting is a concrete gap, not an unknown

Its [ScalerBuilder](https://docs.rs/swash/0.2.10/swash/scale/struct.ScalerBuilder.html) exposes `hint(bool)`. Its actual [hinting cache source](https://docs.rs/crate/swash/0.2.10/source/src/scale/hinting_cache.rs) hard-codes:

```text
HintingMode::Smooth {
    lcd_subpixel: Some(Horizontal),
    preserve_linear_metrics: true,
}
```

This is passed into Skrifa 0.44's HintingOptions conversion, which selects **AutoFallback**, not forced Auto. [Skrifa 0.44 source](https://docs.rs/crate/skrifa/0.44.0/source/src/outline/hint.rs) defines that conversion and the engine rules: TrueType may use its interpreter when fpgm/prep exists; CFF/CFF2 use the PostScript interpreter under AutoFallback. That is not the editor's current FORCE_AUTOHINT policy, and the fixed horizontal-LCD target is not separate slight/full/mono selection.

This does **not** prove Swash text looks unacceptable. It proves the stock public switch cannot simply be wired to the existing three hint modes with equivalent policy. Returning to user review, considering a small direct composition, or stopping are legitimate outcomes; silently collapsing modes or forking is not authorized.

Direct Skrifa [Engine](https://docs.rs/skrifa/0.48.0/skrifa/outline/enum.Engine.html), [Target](https://docs.rs/skrifa/0.48.0/skrifa/outline/enum.Target.html), [SmoothMode](https://docs.rs/skrifa/0.48.0/skrifa/outline/enum.SmoothMode.html), [HintingInstance](https://docs.rs/skrifa/0.48.0/skrifa/outline/struct.HintingInstance.html) and [DrawSettings](https://docs.rs/skrifa/0.48.0/skrifa/outline/struct.DrawSettings.html) expose those distinctions. They supply outline hinting, not a final mask renderer. A Zeno adapter would pass existing outline commands to an existing rasterizer; no custom parser/rasterizer is implied. Exact bold-X/smoothing and mono behavior remain gaps to investigate, not promises that the composition already solves them.

### Output, styles and format distinctions that matter

- [Zeno Format](https://docs.rs/zeno/0.3.3/zeno/enum.Format.html) offers 8-bit Alpha and RGBA subpixel formats, including custom offsets. **Alpha is not one-bit mono**. Thresholding it would be a proposed behavior requiring comparison/review, not an already-proven FreeType mono mode.
- [Swash Render source](https://docs.rs/crate/swash/0.2.10/source/src/scale/mod.rs) exposes offset, transform, embolden and source ordering. A scalar embolden is not proof of the editor's X-only bold policy; inspect the outline operation used and compare. Synthetic italic is not synthetic bold.
- [Fontdue Font source](https://docs.rs/crate/fontdue/0.9.4/source/src/font.rs) explicitly implements horizontal 3x `rasterize_subpixel` coverage. Its constructor extracts outlines using ttf-parser into owned geometry, and its raster path draws that geometry without an exposed hinter, variation selector or arbitrary fractional offset. This corrects the earlier draft's “LCD undocumented” claim. Its parsing errors do not make out-of-range indexed raster calls safe.
- [ab_glyph Font](https://docs.rs/ab_glyph/0.2.32/ab_glyph/trait.Font.html), [FontRef](https://docs.rs/ab_glyph/0.2.32/ab_glyph/struct.FontRef.html), [VariableFont](https://docs.rs/ab_glyph/0.2.32/ab_glyph/trait.VariableFont.html) and [OutlinedGlyph](https://docs.rs/ab_glyph/0.2.32/ab_glyph/struct.OutlinedGlyph.html) cover collections, variation and grayscale outlines. Its raw raster/SVG image access does not provide a complete image decoder/compositor or prove current style/hinting parity.
- Swash advertises bitmap/color sources and PostScript/TrueType outlines. Do not transfer the full **latest Skrifa** format table to Swash's older pinned dependency, or assume decoded COLR/SVG tables mean final glyph rendering. Color/bitmap-only/custom-format behavior still needs actual fixtures.

### Safety and maintenance limits

Skrifa's [README/source package](https://docs.rs/crate/skrifa/0.48.0/source/README.md) states unsafe-forbidden and no-panic goals and its outline API returns DrawError. ttf-parser 0.25.1 likewise documents checked parsing/no-unsafe/no-panic policies. Those properties do not prove whole-stack safety: rasterizers, font indexing, bitmap decoders, Vec allocations and application caches are additional paths.

Swash's zero-transient-allocation design is not a hard allocation cap or a hostile-font guarantee. Its PNG bitmap path uses **Yazi**, a Rust zlib/DEFLATE implementation ([package](https://docs.rs/yazi/0.2.1/yazi/), [Swash PNG source](https://docs.rs/crate/swash/0.2.10/source/src/scale/bitmap/png.rs)); decompression is part of the audit, not absent. Every candidate still needs bounded malformed-font, oversized glyph/bitmap, size/ID and retention experiments. No fuzzing, OOM behavior, cancellation deadline or general work bound is certified here.

## Full native-dependency audit

A throwaway Cargo package outside the editor requested all five roots with **default features**, for `x86_64-unknown-linux-gnu`. Only resolution/fetch, metadata and tree inspection were performed: **no candidate compilation, build-script execution, prototype or editor lock change**.

Resolved roots:

- swash 0.2.10: std/scale/render; requires `skrifa >=0.31.1, <=0.44`, resolving **0.44.0**, plus yazi/zeno.
- direct skrifa 0.48.0: std/autohint_shaping.
- zeno 0.3.3: eval/std.
- fontdue 0.9.4: simd/hashbrown.
- ab_glyph 0.2.32: std/variable-fonts/gvar-alloc.

The union intentionally contains **two Skrifas (0.44/0.48) and two read-fonts (0.41/0.45.1)**. This is not a dependency-resolution failure; current Swash cannot use the 0.48 types as its own dependency. Direct Skrifa + Zeno does not require Swash at all. It is a proposed composition, **not** an upstream dependency edge between those two crates.

Every resolved package has a declared licence and no `links` value. Only **libm, proc-macro2, quote** have build targets; their inspected build sources/configuration perform Rust cfg/version/probe work, not C compilation or native linking. No cc/bindgen/cmake/pkg-config/native font-engine crate occurs in this graph. “No links” alone would not prove this; the build sources were inspected as well. Standard Rust/OS runtime and the editor's existing Lua/SDL dependencies are outside this font-subgraph claim.

Full resolved union (including proc-macro/build dependencies); manifest evidence is available as `https://docs.rs/crate/<name>/<version>/source/Cargo.toml` and exact registry checksums are preserved in the embedded lockfile:

| Package | Version | Declared licence | Build target |
|---|---|---|---|
| ab_glyph | 0.2.32 | Apache-2.0 | — |
| ab_glyph_rasterizer | 0.1.10 | Apache-2.0 | — |
| allocator-api2 | 0.2.21 | MIT OR Apache-2.0 | — |
| bytemuck | 1.25.2 | Zlib OR Apache-2.0 OR MIT | — |
| bytemuck_derive | 1.12.1 | Zlib OR Apache-2.0 OR MIT | — |
| core_maths | 0.1.1 | MIT | — |
| equivalent | 1.0.3 | Apache-2.0 OR MIT | — |
| foldhash | 0.1.5 | Zlib | — |
| foldhash | 0.2.0 | Zlib | — |
| font-types | 0.12.7 | MIT OR Apache-2.0 | — |
| fontdue | 0.9.4 | MIT OR Apache-2.0 OR Zlib | — |
| hashbrown | 0.15.5 | MIT OR Apache-2.0 | — |
| libm | 0.2.16 | MIT | Rust build.rs |
| once_cell | 1.21.4 | MIT OR Apache-2.0 | — |
| owned_ttf_parser | 0.25.1 | Apache-2.0 | — |
| proc-macro2 | 1.0.107 | MIT OR Apache-2.0 | Rust build.rs |
| quote | 1.0.47 | MIT OR Apache-2.0 | Rust build.rs |
| read-fonts | 0.41.0 | MIT OR Apache-2.0 | — |
| read-fonts | 0.45.1 | MIT OR Apache-2.0 | — |
| skrifa | 0.44.0 | MIT OR Apache-2.0 | — |
| skrifa | 0.48.0 | MIT OR Apache-2.0 | — |
| stable_deref_trait | 1.2.1 | MIT OR Apache-2.0 | — |
| swash | 0.2.10 | Apache-2.0 OR MIT | — |
| syn | 3.0.9 | MIT OR Apache-2.0 | — |
| synstructure | 0.14.0 | MIT | — |
| ttf-parser | 0.25.1 | MIT OR Apache-2.0 | — |
| unicode-ident | 1.0.27 | (MIT OR Apache-2.0) AND Unicode-3.0 | — |
| yazi | 0.2.1 | Apache-2.0 OR MIT | — |
| yoke | 0.8.3 | Unicode-3.0 | — |
| yoke-derive | 0.8.4 | Unicode-3.0 | — |
| zeno | 0.3.3 | Apache-2.0 OR MIT | — |
| zerofrom | 0.1.8 | Unicode-3.0 | — |
| zerofrom-derive | 0.1.8 | Unicode-3.0 | — |

The closure is a **combined-feature audit**, not an independently minimized graph for each possible adapter. All-feature/other-target graphs, future releases and licensing obligations must be rechecked at implementation lock time. A declared SPDX expression is recorded, not a legal compliance certification.

Audit lock SHA-256: `c16ede7a3070381405594411efcb576b84b7565af201b90e22f96893808adcbb`. The editor lock remained `a79cd4091669cfc1f896b013d654166e88148bfa9fef7d6761b70e244ed03272`.

## Likely integration boundary — not an interface decision

The existing direct FreeType responsibilities are in renderer.rs and ft.rs, with the Lua binding in renderer_api.rs. File bytes/charmap/metrics/outline hinting/raster masks could be supplied by an existing Rust stack while retaining the editor's tabs, group fallback, atlas/blending, decorations and deferred-draw lifetime. Current SDL stream ownership cannot simply be assumed to match a borrowed FontRef; file-byte lifetime and cache ownership need an explicit later design.

A direct Skrifa/Zeno path would need a bounded outline-command adapter and mask conversion; styling and mono gaps must be demonstrated solvable with existing primitives, not hidden as a new font engine. Stock Swash reduces plumbing but fixes hint policy. Fontdue/ab_glyph are useful contrasting designs, not rejected solely for lacking features the user has not yet required. No shaping/discovery/layout stack is proposed: embedded library functionality does not expand the editor's scope.

## Proposed narrow experiment and unresolved decisions

A **Compare candidate font controls and rendering** prototype/evidence ticket can now be specified, blocked by **Collect the FreeType rendering and cost baseline** and **Define font compatibility and acceptance criteria**, and made a blocker of backend selection:

1. Use the baseline's exact fonts/scenes and agreed candidate set; compare stock Swash and/or direct Skrifa + Zeno where viable. Keep Fontdue as an unhinted/subpixel comparison if the acceptance discussion wants that trade-off. This is a proposed experiment, not an approved shortlist or backend.
2. Demonstrate how none/slight/full + mono/grayscale/LCD are actually routed. Record unavailable controls and alternate algorithms before images/timings; do not silently map all modes to one boolean.
3. Compare unhinted advances/global metrics, three fractional phases, tabs, real mapped icons, genuine cross-font glyphs, .notdef, clipping/cursor placement and custom formats selected by the baseline. Separate parsing from actual glyph rendering.
4. Show synthetic X-only bold, smoothing and italic, plus editor-drawn decorations. Distinguish native mono from any proposed thresholding; document LCD sampling/filter differences.
5. Present measured images/metrics and bounded cold/warm, malformed-input and retention evidence to the **human**. API availability and automatic tests cannot stand in for their appearance/compatibility review.
6. If a required mode or format needs a fork, custom engine or unapproved sacrifice, stop and return that concrete gap to the user.

The baseline remains independently takeable; criteria remain blocked on it. Candidate implementation/prototype work is **not performed by this research**. No visual/performance winner, exclusion, threshold, cache redesign or exception to the Lua stability requirement has been chosen.

## Reproduce the dependency audit without touching the editor

Save the following as Cargo.toml in a fresh temporary directory, add an empty `src/main.rs`, and save the embedded appendix as Cargo.lock. The wildcard manifest matches the original audit request; **use --locked** to retain the recorded snapshot rather than resolving new releases.

```toml
[package]
name = "probe"
version = "0.0.0"
edition = "2021"

[dependencies]
swash = "*"
skrifa = "*"
zeno = "*"
fontdue = "*"
ab_glyph = "*"

[build-dependencies]
```

```sh
cargo metadata --locked --format-version 1 --filter-platform x86_64-unknown-linux-gnu > metadata.json
cargo tree --locked -e normal,build --target x86_64-unknown-linux-gnu
sha256sum Cargo.lock
```

These commands resolve/read the graph; do not run `cargo build` or execute untrusted font input as part of this audit. Publication contains only this public-source report, on an isolated research branch; it is never merged and no PR is implied.

<details>
<summary>Appendix: exact audit Cargo.lock (copy contents verbatim)</summary>

```toml
# This file is automatically @generated by Cargo.
# It is not intended for manual editing.
version = 4

[[package]]
name = "ab_glyph"
version = "0.2.32"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "01c0457472c38ea5bd1c3b5ada5e368271cb550be7a4ca4a0b4634e9913f6cc2"
dependencies = [
 "ab_glyph_rasterizer",
 "owned_ttf_parser",
]

[[package]]
name = "ab_glyph_rasterizer"
version = "0.1.10"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "366ffbaa4442f4684d91e2cd7c5ea7c4ed8add41959a31447066e279e432b618"

[[package]]
name = "allocator-api2"
version = "0.2.21"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "683d7910e743518b0e34f1186f92494becacb047c7b6bf616c96772180fef923"

[[package]]
name = "bytemuck"
version = "1.25.2"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "95832e849adfb21180ccb6826a99da14e5d266ae5c2e668e1602cf234f153797"
dependencies = [
 "bytemuck_derive",
]

[[package]]
name = "bytemuck_derive"
version = "1.12.1"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "6a1f896587b6f2c069c73d2f0913e2d590c3990285cd2f0b6aa02b786b4c679c"
dependencies = [
 "proc-macro2",
 "quote",
 "syn",
]

[[package]]
name = "core_maths"
version = "0.1.1"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "77745e017f5edba1a9c1d854f6f3a52dac8a12dd5af5d2f54aecf61e43d80d30"
dependencies = [
 "libm",
]

[[package]]
name = "equivalent"
version = "1.0.3"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "00d174d5400e5e8fd687ad1049e2f578285fa914201b1af7e8b112a4546bd826"

[[package]]
name = "foldhash"
version = "0.1.5"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "d9c4f5dac5e15c24eb999c26181a6ca40b39fe946cbe4c263c7209467bc83af2"

[[package]]
name = "foldhash"
version = "0.2.0"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "77ce24cb58228fbb8aa041425bb1050850ac19177686ea6e0f41a70416f56fdb"

[[package]]
name = "font-types"
version = "0.12.7"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "835ae9d781464a79026cde387f9cbc650c9a4909d7cf7e658b127e3c35d2f35d"
dependencies = [
 "bytemuck",
]

[[package]]
name = "fontdue"
version = "0.9.4"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "7894823fa221401399e2598f8b63f81ac77ff5c63248b7656779bff1632d7d3d"
dependencies = [
 "hashbrown",
 "ttf-parser",
]

[[package]]
name = "hashbrown"
version = "0.15.5"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "9229cfe53dfd69f0609a49f65461bd93001ea1ef889cd5529dd176593f5338a1"
dependencies = [
 "allocator-api2",
 "equivalent",
 "foldhash 0.1.5",
]

[[package]]
name = "libm"
version = "0.2.16"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "b6d2cec3eae94f9f509c767b45932f1ada8350c4bdb85af2fcab4a3c14807981"

[[package]]
name = "once_cell"
version = "1.21.4"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "9f7c3e4beb33f85d45ae3e3a1792185706c8e16d043238c593331cc7cd313b50"

[[package]]
name = "owned_ttf_parser"
version = "0.25.1"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "36820e9051aca1014ddc75770aab4d68bc1e9e632f0f5627c4086bc216fb583b"
dependencies = [
 "ttf-parser",
]

[[package]]
name = "probe"
version = "0.0.0"
dependencies = [
 "ab_glyph",
 "fontdue",
 "skrifa 0.48.0",
 "swash",
 "zeno",
]

[[package]]
name = "proc-macro2"
version = "1.0.107"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "985e7ec9bb745e6ce6535b544d84d6cd6f7ad8bd711c398938ae983b91a766d9"
dependencies = [
 "unicode-ident",
]

[[package]]
name = "quote"
version = "1.0.47"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "1fbf4db142a473a8d80c26bbf18454ed458bf8d26c8219c331daecfdbd079001"
dependencies = [
 "proc-macro2",
]

[[package]]
name = "read-fonts"
version = "0.41.0"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "046a7d674daf459825b32f5062056d6882db0d2f5a479fbd76ccfc870ac18709"
dependencies = [
 "bytemuck",
 "font-types",
 "once_cell",
]

[[package]]
name = "read-fonts"
version = "0.45.1"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "4edc8db0d972de88e53a085b1f01fb3fbdae7bb0cb7a0ce2b2f8285a412fe3ef"
dependencies = [
 "bytemuck",
 "foldhash 0.2.0",
 "font-types",
 "once_cell",
 "yoke",
]

[[package]]
name = "skrifa"
version = "0.44.0"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "819ab7d62b1d3e72d9d9dea5650bac30424f9111364bb94928dbf5ecad1baa68"
dependencies = [
 "bytemuck",
 "read-fonts 0.41.0",
]

[[package]]
name = "skrifa"
version = "0.48.0"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "a5213b4825bac3587046330052d61bfac7bd3bd0d203c758d7c2ef726fcea034"
dependencies = [
 "bytemuck",
 "read-fonts 0.45.1",
]

[[package]]
name = "stable_deref_trait"
version = "1.2.1"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "6ce2be8dc25455e1f91df71bfa12ad37d7af1092ae736f3a6cd0e37bc7810596"

[[package]]
name = "swash"
version = "0.2.10"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "6c2499c2d826531388872b2268718aed907a39bd785ab0dcfe57fab26283f92e"
dependencies = [
 "skrifa 0.44.0",
 "yazi",
 "zeno",
]

[[package]]
name = "syn"
version = "3.0.9"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "d78c8dee4c7bf0e14673097256fed6142ce9d3b85a408189d07482442145823b"
dependencies = [
 "proc-macro2",
 "quote",
 "unicode-ident",
]

[[package]]
name = "synstructure"
version = "0.14.0"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "901704edd0dfe137f1987838ee4f259e4e063c31371bdb423f7ae38ec6f77f02"
dependencies = [
 "proc-macro2",
 "quote",
 "syn",
]

[[package]]
name = "ttf-parser"
version = "0.25.1"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "d2df906b07856748fa3f6e0ad0cbaa047052d4a7dd609e231c4f72cee8c36f31"
dependencies = [
 "core_maths",
]

[[package]]
name = "unicode-ident"
version = "1.0.27"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "a2c754d6c33795a1c324727428e5a7dedb5b06195f9890bdbcba760d3e246563"

[[package]]
name = "yazi"
version = "0.2.1"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "e01738255b5a16e78bbb83e7fbba0a1e7dd506905cfc53f4622d89015a03fbb5"

[[package]]
name = "yoke"
version = "0.8.3"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "709fe23a0424b6a435d82152b1bd3fdfb0833487d5fa90d05d42762a9891fef5"
dependencies = [
 "stable_deref_trait",
 "yoke-derive",
 "zerofrom",
]

[[package]]
name = "yoke-derive"
version = "0.8.4"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "ec8ebde2db3681e8c9980cc27822030e68752690ddfa9473e739aeb4dbde6d71"
dependencies = [
 "proc-macro2",
 "quote",
 "syn",
 "synstructure",
]

[[package]]
name = "zeno"
version = "0.3.3"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "6df3dc4292935e51816d896edcd52aa30bc297907c26167fec31e2b0c6a32524"

[[package]]
name = "zerofrom"
version = "0.1.8"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "0ec05a11813ea801ff6d75110ad09cd0824ddba17dfe17128ea0d5f68e6c5272"
dependencies = [
 "zerofrom-derive",
]

[[package]]
name = "zerofrom-derive"
version = "0.1.8"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "f75b4683f6c7f45248d4d64056a24298c6281e0993356d7d1b4a1a962ef10d4a"
dependencies = [
 "proc-macro2",
 "quote",
 "syn",
 "synstructure",
]

```

</details>
