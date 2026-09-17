# Ricoh GR III Metadata Reverse Engineering

_My attempts at figuring out how to get settings from the Ricoh GR III's vendor-specific EXIF metadata; image control settings, crop modes, etc. None of this should be considered authoritative, these are essentially just research notes and hacked together configurations, but I figured they might be helpful to others._

---

If you've ever wanted to get all of the image control settings from the metadata of a Ricoh GR III photo, the files and information here should hopefully help.

## tl;dr

Save [`ExifTool_config`](ExifTool_config) as `~/.ExifTool_config`, then use this command to see all of the settings in the order they'd normally be shown in the camera.

```bash
exiftool \
  -GR3CropMode \
  -WhiteBalance \
  -GR3WBShiftABName \
  -GR3WBShiftGMName \
  -PeripheralIlluminationCorr \
  -GR3HighlightCorrection \
  -ShadowCorrection \
  -SensitivityAdjust \
  -ImageTone \
  -GR3Saturation \
  -GR3Hue \
  -GR3HighLowKey \
  -GR3Contrast \
  -GR3ContrastHighlight \
  -GR3ContrastShadow \
  -GR3Sharpness \
  -GR3Shading \
  -GR3Clarity \
  -GR3BWToning \
  -GR3BWFilterEffect \
  -GR3BWGrainEffect \
  -GR3BleachBypassToning \
  -GR3RetroToning \
  -GR3CrossProcessingColorTone \
  -GR3HDRToneToning \
  -GR3HDRToneHDRToneLevel \
  -GR3Recipe \
  <FILE>
```

Here's an example of what the output should look like,

```text
GR3 Crop Mode                   : L (28mm)
White Balance                   : Multi Auto
GR3 WB Shift AB Name            : A6
GR3 WB Shift GM Name            : 0
Peripheral Illumination Corr    : On
GR3 Highlight Correction        : Auto
Shadow Correction               : Normal
Sensitivity Adjust              : +0.7
Image Tone                      : Negative Film
GR3 Saturation                  : 2
GR3 Hue                         : 0
GR3 High Low Key                : 0
GR3 Contrast                    : 3
GR3 Contrast Highlight          : -4
GR3 Contrast Shadow             : -1
GR3 Sharpness                   : 1
GR3 Shading                     : 0
GR3 Clarity                     : 0
GR3 Recipe                      : Reggie's Color Negative
```

If you don't want the configuration to always be used when running `exiftool` commands, you can save it as another file such as `gr3.config` and then use `exiftool -config gr3.config` when you do want to use it.

By having all of these fields available in `exiftool` they should also show up in any other software that uses it. For example, I have them shown in the Metadata sidebar in digiKam with a custom filter.

> [!NOTE]
> It is worth noting that I have absolutely no idea what I'm doing when it comes to ExifTool configuration files. I used the tried and true method of copy/pasting from documentation, changing some values, fixing any errors that showed up, and hoping for the best. I did discover `PrintConv` and updated everything to use that though. If you know what you are doing, don't hesitate to explain what I did wrong so I can learn. This is very much a "works on my machine" type of thing.

## Recipes

Recipes are a collection of "Image Control" settings that give your photos a specific look and feel. Great for when you don't want to deal with post-processing a RAW image and just want a JPEG that looks a certain way straight out of the camera.

Ultimately this entire endeavor was me trying to automatically identify which recipe I may have used on previous photos without having to look at them on the camera itself.

See ["Bonus: Recipes"](Pentax_0x0247.md#bonus-recipes) for information on how to add your own recipes to the [`ExifTool_config`](ExifTool_config) file I put together.

## Details

### Pentax_0x0098

See [`Pentax_0x0098`](Pentax_0x0098.md). This field stores the "Crop Mode" and can be used to [uncrop a photo](Uncropping.md) to get the full image back if you accidentally shot at 35mm or 50mm instead of 28mm, etc.

### Pentax_0x0247

See [`Pentax_0x0247`](Pentax_0x0247.md). This field stores all of the "Image Control" settings. Basically anything from the below settings matrix is encoded in the field.

![](ricoh_gr3_setting_matrix.png)
