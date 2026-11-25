# VIppy

**VIppy** is a tested build of [VIctory](https://github.com/Marvisak/VIctory), the compatibility layer that runs an Xbox Series title natively on Windows, built on [fxceless/victory-fork](https://github.com/fxceless/victory-fork). It combines upstream `main` with open pull requests that were each tried in the game and measured on real hardware, plus fixes of its own. Every fix is credited to its author and, where it is VIppy's own, sent upstream as a pull request too.

> **This repository contains no game files, code or assets.** You need your own copy of the game. VIppy is not affiliated with or endorsed by the game's developer or publisher. See the [asset policy](docs/asset-policy.md).

## Status (25 November 2026)

| | |
|---|---|
| **Progress** | Boots into the world and plays: the prison opening, the cell, corridors and common room, the counselor's office, Lucia's café flashback and the bank robbery in it. |
| **Frame rate** | Median 5.7 fps in the world with the settings below (peaks 8.3); median 7.4 fps (peaks 10.3) with #823's programs off. RTX 5080, Ryzen 7 9800X3D. |
| **Picture** | Textures in place (levels on demand), rooms lit (an ambient constant while ray tracing doesn't run), mouths move in dialogue. |
| **Stability** | No crash and no device loss in nine ten-minute runs on 25 November (this build and the builds leading up to it). |

Measured on one machine: RTX 5080 16 GB (driver 621.14), Ryzen 7 9800X3D (8 threads), Windows 11, Series X profile, release build. Test reports from other hardware are very welcome ([how](CONTRIBUTING.md#test-reports)).

### Known gaps

- **Reflections show a bright sky indoors** (eyes, glasses, picture frames, display glass), and with #823's reflection program on, floors and walls look oily or smeared. Ray tracing doesn't run yet.
- **Moving characters smear** and leave a short trail at these frame rates (the title's temporal upscaler).
- **One mesh pipeline is refused** by the driver in every run (`f5e50eb2108c425b` + `ce3f0432a08f1d35`); its draws are dropped.
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
$env:VICTORY_HEAP_WORKER = '1';     $env:VICTORY_LEVELS_ON_DEMAND = '1'; $env:VICTORY_AMBIENT = '0.6'
target\release\victory.exe run "<game folder>\<the title's exe>" --execute
```

The first launch translates the title's shaders into a cache and is slow; let it run once into the world. Later launches start quickly.

- **Faster, with a trade-off:** `$env:VICTORY_WAVE_WIDE_ONLY = 'none'` turns #823's four programs off. That's about 30 % more frames per second, natural eyes and no sky in floor reflections, but close characters lose their reflected light. It uses its own shader cache.
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
