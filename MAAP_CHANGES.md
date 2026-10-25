# Changes from upstream

Upstream: https://github.com/emit-sds/tetracorder-lite, tag `v1.0.2` at commit 7e04d7ad3821c0c10161dfe39959256b013d94bb (2026-09-11), by the EMIT Science Data System team, Jet Propulsion Laboratory, California Institute of Technology. The vendored Tetracorder, specpr and command files under `tetracorder/` are by Roger N. Clark (Planetary Science Institute) and colleagues, from https://github.com/PSI-edu/spectroscopy-tetracorder. Licenses: Apache License 2.0 for the original work, GPL-3.0 plus the PSI conditions in the `license.txt` files for `tetracorder/` (see `LICENSE` and `NOTICE`, added below from upstream's `main`).

This copy starts from upstream's file tree at 7e04d7a without its git history. The tag has no `.github/` folder, so nothing was left out. It is unchanged except for the items below. The Tetracorder expert system (`cmd.lib.setup.t6.00a6`), the spectral libraries, the mineral matrix (`tetrapy/data/v6.00a6.csv`) and the tetrapy science code are untouched, and the results do not change.

## 2026-10-25: executable bits

`tetracorder/tetracorder/checkmultmapcompilemode` was committed without its executable bit. The tetracorder makefile runs it as `./checkmultmapcompilemode` in its install targets, so on a fresh clone `make install` stopped with `Permission denied` (make error 127) and the Containerfile build failed at the Tetracorder step.

The 24 scripts in `tetracorder/tetracorder.cmds/tetracorder6.00a.cmds/davinci-cmds.for.usr.local.bin/` had the same problem. They are put on PATH and called by name from `cmd.runtet` and the image scripts, so every call failed with `Permission denied`, and the GIF and JPEG browse images and `AAA.info/image-envi-header-1band-bsq.hdr` were never made. All 25 files are now executable. The mineral maps and the L2B products do not use these scripts and do not change.

## 2026-10-25: input and library paths up to 1024 characters

Tetracorder could only be run on short paths:

- `cmd-setup-tetrun` refused a reflectance cube whose path was over 72 characters, because Tetracorder reads the `cube:` line of `cmds.start` into specpr's 80-character input line (`iopcon`).
- The library paths in the restart file (`iwfl`, `iyfl` and the other `lblvol` file names, which tetrapy sets to `convolve.output.reflib` and `reslib`) were limited to 80 characters by the `lblvol` common block and the restart reader, and to 255 by `fsize()`.

Batch systems and cloud jobs often run in deep work folders, where these limits stopped a run before it started. Now:

- The `lblvol` file names are 1024 characters, and so are the arrays that alias them (`dummy` in `fass/dfasin.r` and `main/prochk.r`) and `closnr`'s file name (was 40).
- `rst/rstart.r` reads restart lines into a 1100-character buffer, parses the file names up to 1024 characters, and writes them without trailing blanks.
- `syslib/fsize.c` copies up to 1024 characters of the file name.
- `free/crtin.r` keeps the whole input line, up to 1024 characters, in `iopcln` (new, in the `cmd` common). `iopcon` still gets the first 80 characters, so every other command is read as before.
- `tetracorder/cubecorder.r` takes the `cube:` file name from that whole line when `iopcon` holds the same line (otherwise from `iopcon`, as before). Its cube name buffers are 1024 characters (1200 for the log line).
- `cmd-setup-tetrun` accepts cube paths up to 1024 characters.

Short paths are read exactly as before. specpr's own note in `common/spmaxes` (`maxcline`) about retrofitting every 80-character `iopcon` is still open; only the file names were changed.

## 2026-10-25: install folder and command tree can be set

The tetracorder makefile and four specpr makefiles (`src.sspp`, `src.fstospecpr`, `src.oceanopticstospecpr`, `src.specpr/gfit`) copied their programs to `/usr/local/bin`, and `cmd.runtet` ran Tetracorder from `/usr/local/bin`, while the rest of specpr installs into `$SP_LOCAL/bin`. `cmd-setup-tetrun` copied the command files from `/t1/tetracorder.cmds/...`, a link only the root setup script makes. Now:

- The makefiles install into `$(SP_LOCAL)/bin`, with `SP_LOCAL` defaulting to `/usr/local`.
- `cmd.runtet` runs `${SP_LOCAL:-/usr/local}/bin/tetracorder6.00` (and `tetracorder6.00single`).
- `cmd-setup-tetrun` takes the command tree from `TETRACORDER_CMDS`, default `/t1/tetracorder.cmds/tetracorder6.00a.cmds`.
- tetrapy's setup stage sets `TETRACORDER_CMDS` to the command tree it runs (under `tetracorder.root`) unless it is already set. In the container that is the folder `/t1` points to.

With `SP_LOCAL=/usr/local`, the Containerfile's value, nothing changes.

## 2026-10-25: building without Docker

`environment.yml` lists the build tools the Containerfile gets from apt, from conda-forge: gcc/gfortran 11 (as on Ubuntu 22.04), binutils, make, tcsh, flex, bison, libX11, gnuplot, ImageMagick, netpbm, a JDK, and pixi for the tetrapy environment. The README section "Building without Docker" gives the Containerfile's build variables and steps for a conda environment and how to run tetrapy outside the container. Two packages are not on conda-forge:

- ratfor, needed only to build: the Debian/Ubuntu package (http://archive.ubuntu.com/ubuntu/pool/universe/r/ratfor/ratfor_1.05-2_amd64.deb), unpacked with `dpkg-deb -x`.
- DaVinci, needed only for the browse images and image headers: ASU's Ubuntu 22.04 package (https://software.mars.asu.edu/davinci/davinci_3.0.1-1_amd64_ubuntu22_04.deb) with Ubuntu 22.04's `libcfitsio9`, `libhdf5-103-1` and `libcurl3-nss`. A run without DaVinci on PATH still exits 0 and makes the same `agg.nc` and `agg-uncert.nc`, value for value (checked on the 300 x 150 demo tile).

## 2026-10-25: license files

The v1.0.2 tree had no top-level license. `LICENSE`, `NOTICE` and `tetracorder/COPYING` are copied unchanged from upstream's `main` at commit 8af7206 (added there on 2026-09-22 and 2026-09-23).

## Known issues, not changed

- **The run writes into its install tree.** tetrapy's sensor stage writes `restart_files/r1-<name>`, `DELETED.channels/`, `DATASETS/`, `DISABLE/` and `COLOR.channels/` under `tetracorder/tetracorder.cmds/tetracorder6.00a.cmds/` (`tetrapy/sensor.py`), because `cmd-setup-tetrun` looks the sensor files up there. The build also edits tracked files (the specpr install script and `multmap.h`, by line number, as in the Containerfile). So the `tetracorder/` folder must be writable when the pipeline runs; with a read-only install, a copy of `tetracorder/` passed as `--tetracorder.root` should work (not tested). Not a small change, so left as is.
- **Memory: about 7.4 GB per scene.** The aggregate stage loads the whole reflectance and uncertainty cubes (3.6 GB as float32 for a full EMIT scene) and copies the pixels of each winning material. Tetracorder itself is single-threaded; `setup.cores` only sets parallel jobs for the browse images.
- **About 1.5% of group 2 differs from the official EMIT L2B MIN V002 product.** Most of it is this code choosing `Plastic_Vinyl GDS372` or `Coated_Steel_Girder` where the official product has calcite, calcite+dolomite, aragonite or dry grass. The cause is not known: the official run used its own `/data/config.yml`, which may differ from the repository's `config.yml`, and the LP DAAC NetCDF stores wavelengths as float32. Group 1 agrees on 99.95% of valid pixels.
- Also from v1.0.2: no `pixi.lock` (pixi solves fresh, Python 3.15 here although `.python-version` says 3.12); `setup.cores` defaults to `os.cpu_count()`, all host cores; `agg.nc` writes 0 ("no mineral") where the input is no-data, the official product -9999; `cubecorder.r` calls `/usr/local/bin/logtetracorder`, a usage log that appends to `/usr/log/tetracorder`; elsewhere the call fails and the run goes on.

## Check

On 2026-10-25, on Ubuntu 22.04 x86_64 without root or Docker: a fresh `git clone` of this repository, a conda environment made from `environment.yml` (29 s to solve and install; gcc and gfortran 11.4.0), ratfor 1.05-2 from the Ubuntu package, then the README's "Building without Docker" commands. specpr and both Tetracorder binaries built in 19 s; `pixi install` gave Python 3.15.1, numpy 2.5.4, GDAL 3.13.3, rasterio 1.5.2, xarray 2026.9.0. v1.0.2 has no tests.

The run used EMIT scene `emit20230728t214153` (L2A reflectance and uncertainty V002 from LP DAAC, written as ENVI BIL float32, 1.8 GB each) placed in a folder whose path is 291 characters long, so the reflectance path was 307 characters and the convolved library paths 310. Outputs went under the same folder. DaVinci 3.0.1 was on PATH. No network:

```sh
tetrapy run config.yml --tetracorder.root "$PWD/tetracorder" --output.base <long folder>/output \
  --data.rfl <long folder>/input/e0728_rfl --data.rfluncert <long folder>/input/e0728_uncert \
  --convolve.output.reflib <long folder>/output/conv/reflib \
  --convolve.output.reslib <long folder>/output/conv/reslib --setup.cores 4
```

It exited 0 in 13 min 19 s (Tetracorder 11 min 52 s) with 7.4 GB peak memory, and made 819 GIF and 2,021 JPEG browse images with no `Permission denied` messages. Compared with the official EMIT L2B MIN and MINUNCERT V002 granules for that scene, the mineral IDs agree on 99.95% (group 1) and 98.51% (group 2) of valid pixels, and every statistic of the comparison (band depth, uncertainty, fit, disagreement counts) is the same as for the unchanged v1.0.2 code run on short paths. On the 300 x 150 demo tile from the `demo-data-v6` release, run from a 334-character path, 97.4% (group 1) and 95.7% (group 2) of pixels are identified, as the release notes say; without DaVinci on PATH `agg.nc` and `agg-uncert.nc` were identical value for value.
