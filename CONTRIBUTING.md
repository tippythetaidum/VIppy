# Contributing to VIppy

Thanks for helping. VIppy is a tested build, so the most useful contributions are clear reports of what happens on your hardware, and fixes that come with a measurement.

## Ground rules

- **Nothing from the game goes into this repository or into issues and pull requests**: no assets, no executables or parts of them, no shader dumps, no screenshots or frames committed to the tree. Read the [asset policy](docs/asset-policy.md). `cargo xtask install-hooks` checks it on every commit. A link to your own recording in an issue is fine.
- Use your own copy of the game.
- Say so when a contribution is AI-assisted, and check its claims against your own runs before posting.
- Be precise: a number you measured, a commit you ran, the switches you set. "Seems better" is a start; a frame or a count is what lets someone act on it.

## Bug reports

Open an issue with the **Bug report** template. The essentials:

- the VIppy commit (`git rev-parse --short HEAD`), and whether it is a `release` or `diag` build;
- GPU, driver, CPU, RAM, Windows version and the console profile (Series X or S);
- the switches you set (`VICTORY_*` environment variables);
- the scene, what you saw and what you expected;
- the relevant lines of the log or `crashes.log` (no game data in them, only counts, hashes and messages).

## Test reports

A test report on hardware VIppy hasn't seen is a contribution in itself. Use the **Test report** template, and give:

- the hardware and build as above;
- how far you played, and the frame rate in the world (the window caption shows it; a median over a few minutes is best);
- what looked wrong, scene by scene;
- crashes or device losses, with the time into the run.

## Pull requests

1. Keep each pull request to one change, with a title that says what it fixes.
2. Run the checks before you push:
   ```powershell
   cargo fmt --all
   cargo clippy --workspace --all-targets
   cargo test --workspace
   ```
3. Add a test where the behaviour can be tested without the game (unit tests, synthetic shaders, WARP).
4. Describe what you measured in the game, before and after, on which hardware.
5. If the fix is general, send it to [fxceless/victory-fork](https://github.com/fxceless/victory-fork) too, so every build benefits.

## How upstream fixes get into VIppy

VIppy follows fxceless/victory-fork `main`. An open pull request from there is picked when:

- its diff has been read in full (nothing from another repository is run unread);
- it has been run in the game on at least one machine, with frames looked at;
- it makes things better without a regression it doesn't name.

The result goes back to that pull request as a test report. If you'd like a pull request considered, open an issue with a link to it.

## Licence

By contributing you agree that your work is licensed under GPL-3.0-or-later, as the rest of the project.
