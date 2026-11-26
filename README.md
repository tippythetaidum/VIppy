# VIppy

**VIppy** is a tested build of [VIctory](https://github.com/Marvisak/VIctory), the compatibility layer that runs an Xbox Series title natively on Windows, built on [fxceless/victory-fork](https://github.com/fxceless/victory-fork). It combines upstream `main` with open pull requests that were each tried in the game and measured on real hardware, plus fixes of its own. Every fix is credited to its author and, where it is VIppy's own, sent upstream as a pull request too.

> **This repository contains no game files, code or assets.** You need your own copy of the game. VIppy is not affiliated with or endorsed by the game's developer or publisher. See the [asset policy](docs/asset-policy.md).

## Status (25 November 2026)

| | |
|---|---|
| **Progress** | Boots into the world and plays: the prison opening, the cell, corridors and common room, the counselor's office, Lucia's café flashback and the bank robbery in it, and on into Chapter 1 (the boat) with the newer build on branch [`integration-1125`](https://github.com/tippythetaidum/VIppy/tree/integration-1125): victory-fork `main` `d827419` plus the pull requests here. That branch, run with the settings below, gave the cleanest picture so far on 25 November; it isn't merged into this branch yet. |
| **Frame rate** | Median 5.7 fps in the world with the settings below (peaks 8.3); median 7.4 fps (peaks 10.3) with #823's programs off. RTX 5080, Ryzen 7 9800X3D. |
| **Picture** | Textures in place (levels on demand), rooms lit (an ambient constant while ray tracing doesn't run) at about the console's brightness (at `VICTORY_AMBIENT=0.3`, mean luma within 2-4 of 255 of Series X footage in three matched shots, measured in a test build of this tree with the ray-tracing pull requests merged), mouths move in dialogue. |
| **Stability** | No crash and no device loss in nine ten-minute runs on 25 November (this build and the builds leading up to it). A freeze with the ray-tracing structure builder on (`VICTORY_BVH_BUILDER=1`, not in this build's settings) is fixed here: the write guards no longer make the process heap read-only. |

Measured on one machine: RTX 5080 16 GB (driver 621.14), Ryzen 7 9800X3D (8 threads), Windows 11, Series X profile, release build. Test reports from other hardware are very welcome ([how](CONTRIBUTING.md#test-reports)).

### Known gaps

- **Reflections show a bright sky indoors** (eyes, glasses, picture frames, display glass). Ray tracing doesn't run yet. With #823's screen-space reflection tiling on, floors and walls look oily and smeared, so the settings below leave that one program off; close characters then look flatter in places.
- **Moving characters smear** and leave a short trail at these frame rates (the title's temporal upscaler).
- **Black textures and short black flickers** when the card is near its memory budget: past 90 % no new texture copies are made, and idle copies are let go and made again ([victory-fork#1080](https://github.com/fxceless/victory-fork/issues/1080)).
- Bright sun shafts can blow out characters in the bank scene.

## What is in it

Base: fxceless/victory-fork `main` at `ca8829f`, with these pull requests from that repository:

| Pull request | Author | What it does |
|---|---|---|
| [#24](https://github.com/fxceless/victory-fork/pull/24) | NuraadGit | Constant buffer views of at most 64 KiB |
| [#927](https://github.com/fxceless/victory-fork/pull/927) | tippythetaidum | 16-bit buffer accesses keep their bits: the face network works and mouths move in dialogue |
| [#823](https://github.com/fxceless/victory-fork/pull/823) | Felix2180 | Four compute programs NVIDIA refused in their 64-lane form run (water volumetrics, particle visibility, screen-space reflection tiling) |
| [#606](https://github.com/fxceless/victory-fork/pull/606) | laptopnotebook | A remap under an uploaded texture uploads it again |
| [#1002](https://github.com/fxceless/victory-fork/pull/1002) | brunofgmag | The tiled cascade shadow programs translate exactly on 32-lane hosts |
| [#907](https://github.com/fxceless/victory-fork/pull/907) | SpaghettiMaster545 | Pixel programs the console depth-tests first declare early depth |
| [#834](https://github.com/fxceless/victory-fork/pull/834) | SantannaCarlos | Indirect dispatches past 65,535 groups run whole (no band of old picture at the bottom edge) |
| [#1007](https://github.com/fxceless/victory-fork/pull/1007) | TG0000 | Textures picked by index are uploaded on a thread of their own |
| [#1031](https://github.com/fxceless/victory-fork/pull/1031) | Lkxex | Bounded texture queue work; views over a resident run of levels |
| [#917](https://github.com/fxceless/victory-fork/pull/917) | UwUCammie (pass by Belobug) | A constant for the indirect light while rays don't run |
| [#982](https://github.com/fxceless/victory-fork/pull/982) | brunofgmag | Clippy passes again on victory-gpu's warnings (CI green) |
| [#1051](https://github.com/fxceless/victory-fork/pull/1051) | meganegga | A depth-bounds test skips hosts without shader model 6.6 |
| [#501](https://github.com/fxceless/victory-fork/pull/501) | Yorivel | No blending on integer render targets: the one mesh pipeline the driver refused (`f5e50eb2108c425b`) is made |
| [#1147](https://github.com/fxceless/victory-fork/pull/1147) | tippythetaidum | The write guards make only views read-only, never committed private memory or free memory (a freeze with the structure builder on) |

VIppy's own changes:

- `VICTORY_TEXTURE_IDLE_MS`: how long an idle host texture is kept (5,000 ms unless set).

Each pick was read in full before it went in, and the reports from testing it are on its pull request.

## Getting started

Requirements: Windows 10/11 x64, a CPU with AVX2, a Direct3D 12 GPU with shader model 6.6 and mesh shaders, Rust (pinned in `rust-toolchain.toml`), and several GiB of free Windows commit (raise the paging file if `crashes.log` reports error 1455).

```powershell
git clone https://github.com/tippythetaidum/VIppy
cd VIppy
cargo xtask install-hooks                # asset-policy check on every commit
copy victory.example.toml victory.toml   # then set your paths (outside the repo)
cargo build --release -p victory
```

Run the title with VIppy's settings:

```powershell
$env:VICTORY_INDIRECT_WINDOW = '1'; $env:VICTORY_COPY_PLAIN = '1'; $env:VICTORY_JOIN_PARTS = '1'
$env:VICTORY_BACK_OWN_VIEW = '1';   $env:VICTORY_INDIRECT_GATHER = '1'; $env:VICTORY_CARD_REHASH_RUNS = '1'
$env:VICTORY_ASYNC_SUBMIT = '1';    $env:VICTORY_QUEUE2 = '1';     $env:VICTORY_TEXTURE_NO_WAIT = '1'
$env:VICTORY_HEAP_WORKER = '1';     $env:VICTORY_LEVELS_ON_DEMAND = '1'; $env:VICTORY_AMBIENT = '0.3'
$env:VICTORY_TEXTURE_STAGE_MB = '4096'; $env:VICTORY_TEXTURE_STAGE_MS = '8000'   # finish due textures before the frame
$env:VICTORY_WAVE_WIDE_ONLY = '5dc3b25986622949,6f225b641c5b0708,369cbe906ea72764'   # #823 without the reflection tiling: no oily floors
$env:VICTORY_FILL_WAIT = '1'   # constants copied for the card wait for the card's work before them, as on the console: no blurred oval in the café
target\release\victory.exe run "<game folder>\<the title's exe>" --execute
```

The first launch translates the title's shaders into a cache and is slow; let it run once into the world. Later launches start quickly.

- **Reflection tiling back on:** leave `VICTORY_WAVE_WIDE_ONLY` unset for all four of #823's programs. Close characters keep their reflected light, but floors and walls turn oily (the reflection rays find no ray-tracing structures and return the sky).
- **Why `VICTORY_FILL_WAIT=1`:** by default the layer copies constant data for a submission without waiting for earlier work on the card that may change it, and waits only after it catches one copy that was stale. That one stale copy comes once a run, as the café flashback starts, and a blurred oval over the middle of the picture then stays for the whole scene (6 of 7 runs without the switch, 0 of 3 with it; [victory-fork#1025](https://github.com/fxceless/victory-fork/issues/1025)). It costs about one frame per second.
- **Faster, with a trade-off:** `$env:VICTORY_WAVE_WIDE_ONLY = 'none'` turns all four off. That's about 30 % more frames per second, but the water and particle programs are dropped as well.
- These apply to NVIDIA and other 32-lane cards: on 64-lane cards (AMD) the four programs run in their own form and the switch has no effect. Each value has its own shader cache.
- **Input:** XInput pads. Without one, the keyboard acts as a pad while the window has focus (WASD and IJKL for the sticks, the arrow keys for the D-pad, Enter, Backspace, X and Y for A, B, X and Y).
- **Commands, logging and debugging aids:** see [upstream's README](docs/victory-fork-README.md#commands).

## Contributing

Bug reports, test reports from other hardware and pull requests are all welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) first. Fixes that are general are best sent to [fxceless/victory-fork](https://github.com/fxceless/victory-fork) as well, so every build benefits.

## Credits

- **[VIctory](https://github.com/Marvisak/VIctory)** by Marvisak and contributors is the base of everything here: the loader, runtime services, the Direct3D 12 back end and the shader recompiler.
- **[fxceless/victory-fork](https://github.com/fxceless/victory-fork)** and its many contributors: the fork VIppy follows, with its own credits in [its README](docs/victory-fork-README.md#credits).
- The authors of the pull requests listed above.
- VIppy's testing, integration and own changes are developed with [Claude Code](https://claude.com/claude-code).

## Licence

GPL-3.0-or-later, as VIctory and victory-fork. See [LICENSE](LICENSE).
