# chromebook-audio-ucm

ALSA UCM2 profiles that make Chromebooks' speakers, headphones, microphones and
HDMI work under Linux, switching when something is plugged in, following
ChromiumOS's own audio configuration for each model. They extend alsa-ucm-conf,
for PipeWire, PulseAudio and anything else that reads UCM.

## Covered

Chromebooks from about 2012 on, each with the profile ChromiumOS uses for its
model and SKU:

- Intel, from Bay Trail and Broadwell to Meteor Lake
- AMD, from Stoney Ridge on
- MediaTek, Qualcomm, Rockchip, Samsung Exynos and NVIDIA Tegra K1

Not covered:

- Sound cards Chromebooks share with other PCs, which alsa-ucm-conf already
  serves: HDA (Haswell and older, and Sarien) and SOF's HDA and SoundWire cards.
- Models newer than these profiles.
- Deku, which has no ChromiumOS profile, and Skate, Spring and Kitty, whose
  sound has no mainline device tree.
- ChromeOS's sound processing: equalisers, speaker protection and microphone
  tuning, which run in its sound server or come from private packages. Speakers
  can sound quieter or thinner than on ChromeOS.
- Hotword detection, Bluetooth audio through the sound card, and echo
  cancellation references.

Limits:

- For newer boards, which SKU uses which profile variant is private. Those
  models get their model's profile, or the plainest-named of several.
- MT8189 and MT8196 boards have no mainline device tree yet: their profiles use
  ChromiumOS's card names and leave out DisplayPort audio.
- Snow and Peach report no jacks under mainline, so headphones there are chosen
  by hand.

## Requirements

- alsa-lib 1.2.14 or newer, and alsa-ucm-conf.
- A kernel with the card's driver. Skylake to Apollo Lake use the AVS driver
  with linux-firmware's `intel/avs` topologies; SOF cards need sof-firmware.

## Installation

Copy the profiles into alsa-ucm-conf's directory, then restart the sound server:

```sh
sudo cp -r ucm2/. /usr/share/alsa/ucm2/
systemctl --user restart pipewire wireplumber
```

To check it, run `alsaucm -c hw:0 list _devices/HiFi`, with the card number from
`aplay -l`: it should list the speaker, headphones and microphones.

A few files replace alsa-ucm-conf's own for the same Chromebooks, so copy again
after an alsa-ucm-conf update.

To uninstall, delete the files, then reinstall alsa-ucm-conf (`alsa-ucm` on
Fedora) to restore its own:

```sh
(cd ucm2 && find . -type f -exec sudo rm /usr/share/alsa/ucm2/{} \;)
sudo find /usr/share/alsa/ucm2/conf.d -type d -empty -delete
```

### Packaging

Install `ucm2/conf.d` to `/usr/share/alsa/ucm2/conf.d/`. The package needs to
take over any files alsa-ucm-conf also ships, with a diversion or by replacing
them.

## Background

### Why Chromebooks need their own profiles

A Chromebook's sound is a codec, a speaker amplifier and microphones particular
to the model, each wired its own way. A UCM profile says which mixer controls to
set for each output and input, and how to detect what's plugged in. ChromiumOS
has one for every model, written for its own sound server, CRAS, and its own
kernels. alsa-ucm-conf has only a few, so most Chromebooks get no speaker or
headphone switching.

### How they differ from ChromiumOS's

Each profile keeps ChromiumOS's devices, mixer settings, jacks and PCMs, under
the names mainline Linux uses: AVS's current card names, Skylake's cards split
into AVS's card per codec, and the controls and jacks ChromiumOS's kernels name
differently. Anything mainline lacks is dropped, so every control, jack and PCM
a profile uses exists.

### How machines are matched

Each card's `conf.d/<driver>/<driver>.conf` reads the firmware name, from
Google's firmware version or MrChromebox's DMI product name, and the SKU, from
DMI or the device tree, then loads the profile ChromiumOS would. A SKU
ChromiumOS doesn't list gets a profile from the same firmware; any other machine
gets none.

## Contributing

Profiles come from ChromiumOS's sources; cite the one a merge request comes
from. CI runs alsa-ucm-conf's validator; to run it locally:

```sh
git clone --depth 1 https://github.com/alsa-project/alsa-tests
make -C alsa-tests/python/ucm-validator2 ALSA_UCM_DIR="$PWD/ucm2"
```

## Sources

- [ChromiumOS](https://chromium.googlesource.com/chromiumos): audio profiles,
  model configs, CRAS
- [ChromeOS releases](https://chromiumdash.appspot.com/serving-builds?deviceCategory=ChromeOS)
  and [MrChromebox](https://docs.mrchromebox.tech/docs/supported-devices.html):
  released models
- [Linux](https://github.com/torvalds/linux/tree/master/sound),
  [SOF](https://github.com/thesofproject/sof-bin) and
  [linux-firmware](https://gitlab.com/kernel-firmware/linux-firmware): card,
  control, jack and PCM names
- [alsa-ucm-conf](https://github.com/alsa-project/alsa-ucm-conf): UCM2 syntax

## License

[MPL-2.0](LICENSE). The profiles can be shipped as-is in any project, under any
license; changes to them, and anything copied out of them, stay MPL-2.0.
