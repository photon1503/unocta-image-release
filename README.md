# UNOCTA-image

**From raw astro frames to a finished color image, in one click.**

Point UNOCTA at your folders of bias, dark, flat and light frames. It sorts them by their FITS
headers, builds the calibration masters, calibrates, aligns and stacks your lights, and makes a
color image you can fine-tune with a few sliders. Careful, high-quality processing without
the learning curve.

![NGC 7129, made with UNOCTA-image from LRGB + Hα frames](images/ngc7129.jpg)
<sub>NGC 7129: L, R, G, B and Hα frames, processed by UNOCTA-image with default settings (RC Astro tools installed).</sub>

## Download

<p align="center">
<a href="https://github.com/photon1503/unocta-image-release/releases/latest/download/unocta-windows-x86_64.exe"><img alt="Download for Windows" src="https://img.shields.io/badge/Windows-Download-0078D6?style=for-the-badge&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI%2BPHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0yIDJoOS41djkuNUgyek0xMi41IDJIMjJ2OS41aC05LjV6TTIgMTIuNWg5LjVWMjJIMnpNMTIuNSAxMi41SDIyVjIyaC05LjV6Ii8%2BPC9zdmc%2B&logoColor=white"></a>
<a href="https://github.com/photon1503/unocta-image-release/releases/latest/download/unocta-macos-arm64"><img alt="Download for Mac with Apple Silicon" src="https://img.shields.io/badge/macOS-Apple%20Silicon-000000?style=for-the-badge&logo=apple&logoColor=white"></a>
<a href="https://github.com/photon1503/unocta-image-release/releases/latest/download/unocta-macos-x86_64"><img alt="Download for Mac with Intel processor" src="https://img.shields.io/badge/macOS-Intel-555555?style=for-the-badge&logo=apple&logoColor=white"></a>
<a href="https://github.com/photon1503/unocta-image-release/releases/latest/download/unocta-linux-x86_64"><img alt="Download for Linux" src="https://img.shields.io/badge/Linux-Download-E95420?style=for-the-badge&logo=linux&logoColor=white"></a>
</p>

The buttons always download the newest version. It is the whole program: no installer.
Not sure which Mac you have? Apple menu → *About This Mac*: "Chip: Apple M…" means Apple Silicon.
All versions and their release notes: [releases](https://github.com/photon1503/unocta-image-release/releases).

**Windows:** double-click the file. If SmartScreen warns about an unknown app, click
*More info → Run anyway*.

**Mac:** open Terminal in the download folder and run once:
```sh
chmod +x unocta-macos-arm64
xattr -d com.apple.quarantine unocta-macos-arm64
./unocta-macos-arm64
```

**Linux:** `chmod +x unocta-linux-x86_64`, then start it.

## How it works

![The main window](images/main-window-frames.png)

1. **Drop your folders** on the window (or click the dashed area to choose them).
   Sub-folders are included; frames are recognized from their FITS headers.
2. **Check what was found.** Each group shows its frame count and how it will be used.
   Untick anything you don't want.
3. **Click *Start processing*.** Masters and stacked lights are saved to the output folder
   (XISF or FITS). When there are color filters, the color image opens with sliders for
   brightness, color and stars. Save it as PNG, TIFF, XISF or FITS.

![The color image with its sliders](images/color-window.png)

Recognized filters: L, R, G, B, Hα, OIII, SII (also spelled Lum, Red, H-alpha, "Ha 7nm", …).
If a name isn't recognized, UNOCTA asks once and remembers.

**Optional:** with RC Astro's command-line tool `rc-astro` installed and licensed,
BlurXTerminator, NoiseXTerminator and StarXTerminator are used automatically.

## Updates

UNOCTA checks at start whether a newer version is available. If so, an **Update** button
appears at the top. It shows what changed and installs only when you confirm. Your settings
stay. Turn the check off under **Settings → Updates**.

## Good to know

- Your original frames are only read, never changed.
- Nothing is sent anywhere, except the update check.
- What changed in each version: see the [releases](https://github.com/photon1503/unocta-image-release/releases).

## Command line

The same program also runs from a terminal, for scripts and batch work. Without arguments it
opens the window. In the examples, `unocta` stands for the downloaded file
(e.g. `./unocta-macos-arm64` or `unocta-windows-x86_64.exe`).

```sh
unocta scan /data/M31                                     # what was found
unocta masters /data/calibration -o /data/masters         # calibration masters
unocta lights /data/M31 -m /data/masters -o /data/stacked # calibrated, stacked lights
unocta compose /data/stacked -o /data/final --name M31    # finished color image
```

`unocta --help` and `unocta <command> --help` list everything below. `unocta --version` shows the version.

### `unocta gui [FOLDERS…]`

Opens the window, with these folders or files already added.

```sh
unocta gui /data/2026-10-12 /data/calibration
```

### `unocta scan [FOLDERS…]`

Lists the groups of frames (bias, darks, flats, lights) found in the folders or files. Changes nothing.

```sh
unocta scan /data/NGC7129 /data/calibration
```

### `unocta masters [FOLDERS…] -o OUT`

Builds master bias, dark, dark flat and flat frames.

| Option | Meaning |
|---|---|
| `-o, --out FOLDER` | Output folder (required). |
| `--format xisf\|fits` | File format of the masters. Default: `xisf`. |
| `--rebuild` | Build masters again even if an up-to-date one is already in the output folder. |
| `--flat-rejection linear\|winsorized\|none` | Outlier rejection for flats. Default: `linear` (as in PixInsight's WBPP). |
| `--no-flat-calibration` | Do not subtract the master bias from the flats. |
| `--flat-calibration-frame FILE` | Calibrate the flats with this existing master (FITS/XISF) instead. |
| `--scale-darks-for-flats` | If there is no dark flat or dark with the flats' exposure, use bias + the nearest dark, scaled to the flat exposure. |

```sh
# Masters as FITS, built again from scratch
unocta masters /data/calibration -o /data/masters --format fits --rebuild

# Flats without dark flats: calibrate them with bias + a scaled dark
unocta masters /data/calibration -o /data/masters --scale-darks-for-flats
```

### `unocta lights [FOLDERS…] -m MASTERS -o OUT`

Calibrates, aligns and stacks the light frames into one master light per filter.

| Option | Meaning |
|---|---|
| `-m, --masters FOLDER` | Folder with the calibration masters (required; can be given more than once). |
| `-o, --out FOLDER` | Output folder (required). |
| `--format xisf\|fits` | File format of the master lights. Default: `xisf`. |
| `--clean` | Delete the aligned single frames after a successful run. By default they are kept, so a repeated run is faster. |
| `--no-local-normalization` | Match the frames' brightness with one offset and scale per frame instead of local normalization. |
| `--unprocessed-rgb tiff\|xisf\|fits` | Also save R, G and B as one RGB image, only aligned and cropped. |

```sh
# Stack two nights with masters from two folders, delete the aligned frames afterwards
unocta lights /data/NGC7129/night1 /data/NGC7129/night2 \
  -m /data/masters/2026-10 -m /data/masters/darks -o /data/NGC7129/stacked --clean

# Also save an unprocessed RGB TIFF for your own processing
unocta lights /data/M82 -m /data/masters -o /data/M82/stacked --unprocessed-rgb tiff
```

### `unocta compose [FOLDERS…] -o OUT`

Makes the finished color image from master lights (L, R, G, B, Hα, OIII, SII, picked by their
filter names). Writes `<name>.png` (stretched, 16-bit) and `<name>_linear.xisf` (linear RGB).
Options left out use the same automatic settings as the window; some of them depend on the
image type (broadband, narrowband, galaxy). The defaults below are for nebulae;
`--galaxy-white` changes those for stretch, sky level, object and stars.

| Option | Meaning |
|---|---|
| `-o, --out FOLDER` | Output folder (required). |
| `--name NAME` | File name of the result. Default: `composite`. |
| `--palette auto\|rgb\|lrgb\|halrgb\|sho\|hoo` | How the filters are combined. Default: `auto`. |
| `--sho-colors gold-hubble\|gold\|hubble\|natural\|foraxx` | Colors of an SHO image. Default: `gold-hubble`. |
| `--filters auto\|rgb\|photometric` | Color filters: recognized from the names, imaging RGB, or photometric (Johnson/Cousins, Sloan). Default: `auto`. |
| `--galaxy-white` | Make a galaxy's overall light white (instead of the color of a Sun-like star), with the galaxy look. |
| `--nebula-fill` | The nebula fills the whole frame: take the darkest part as the sky instead of the median. |
| `--unprocessed-rgb tiff\|xisf\|fits` | Also save `<name>_RGB_unprocessed`: R, G and B only aligned and cropped. |
| **Brightness** | |
| `--background N` | Stretch strength: where the sky sits on the stretch curve (0–1). Default: 0.10. |
| `--sky-level N` | Brightness of the darkest sky in the result (0–0.5). Default: 0.09 (≈ 23/255), 0.04 with `--galaxy-white`. |
| `--object N` | Target brightness of the object (0–1). |
| `--black-point N` | This level and darker becomes black (0–0.3). Default: 0. |
| **Color** | |
| `--saturation N` | Color saturation of the nebula/galaxy. |
| `--star-saturation N` | Color saturation of the stars. Default: 1.0. |
| `--nebula-temperature K` | White balance of the object, as the assumed temperature of the average star (lower = warmer). Default: automatic. |
| `--star-temperature K` | White balance of the stars, separately (lower = warmer). Default: automatic. |
| `--object-hue DEG` | Rotate the object's hue, not the stars' (−180 to 180°). Default: 0. |
| `--ha-strength N` | How much of the Hα signal is added to red (0 = none). Default: 3. |
| `--ha-blend N` | Strength of the stretched Hα layer added to red (0 = off). |
| **Stars** | |
| `--star-stretch N` | Star stretch relative to the nebula (smaller = smaller stars). Default: 0.2. |
| `--halo-reduction N` | How much of bright stars' halos is removed (0–1). Default: 0.35. |
| **Detail and noise** | |
| `--contrast N` | Local contrast on the object (0 = none). Default: 1.2. |
| `--detail N` | Fine-detail enhancement on the object (0 = none). Default: 0.8. |
| `--noise-reduction N` | Noise reduction threshold in σ (0 = off). Default: 2. |
| `--deconvolution N` | Sharpening iterations (Richardson–Lucy) on the luminance (0 = none). Default: 0. |
| **RC Astro tools** (used when `rc-astro` is installed) | |
| `--no-rc-astro` | Don't use BlurXTerminator, NoiseXTerminator and StarXTerminator. |
| `--bxt-stars N` | BlurXTerminator "sharpen stars" (0–0.7). Default: 0.5. |
| `--bxt-nonstellar N` | BlurXTerminator "sharpen nonstellar" (0–1). Default: 0.5. |
| `--nxt-denoise N` | NoiseXTerminator "denoise" (0–1). Default: 0.9. |
| **Output** | |
| `--rotate 0\|90\|180\|270` | Rotate the finished image clockwise (the linear XISF keeps the original orientation). Default: 0. |
| **AstroBin** | |
| `--astrobin FOLDER` | Also write an AstroBin upload set to this folder: `<name>.jpg` (full resolution), `acquisition.csv` (AstroBin's CSV import format) and `description.txt`. Nothing is uploaded. The filter IDs come from the window's saved AstroBin settings. |
| `--frames FOLDER…` | Folders with the light and calibration frames, for the AstroBin table's nights and calibration frame counts. Without it, the table is made from the master lights (one row per master light, no dates). |

```sh
# Nebula with default settings
unocta compose /data/NGC7129/stacked -o /data/NGC7129/final --name NGC7129

# Galaxy: white galaxy light, a darker sky, turned a quarter clockwise
unocta compose /data/M82/stacked -o /data/M82/final --name M82 --galaxy-white --sky-level 0.06 --rotate 90

# Narrowband in Hubble colors, a bit more saturated, without RC Astro's tools
unocta compose /data/Veil/stacked -o /data/Veil/final --name Veil \
  --palette sho --sho-colors hubble --saturation 1.6 --no-rc-astro

# Warmer stars, smaller star halos, a little sharpening of the luminance
unocta compose /data/M31/stacked -o /data/M31/final --name M31 \
  --star-temperature 3500 --halo-reduction 0.6 --deconvolution 10

# Finished image plus an AstroBin upload set, with nights and calibration counts from the raw frames
unocta compose /data/Veil/stacked -o /data/Veil/final --name Veil \
  --astrobin /data/Veil/astrobin --frames /data/Veil/lights /data/calibration
```

### `unocta compare A B`

Compares two images pixel by pixel and prints the differences (in 16-bit ADU).

```sh
unocta compare /data/masters/masterFlat_BIN-1_6248x4176_FILTER-L.xisf /data/pixinsight/masterFlat_L.xisf
```

### `unocta update [--yes]`

Checks for a newer version and installs it after asking. `--yes` installs without asking.

```sh
unocta update          # asks before installing
unocta update --yes    # e.g. from a script
```
