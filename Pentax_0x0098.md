# Pentax 0x0098

> [!NOTE]
> Tested only with a Ricoh GR III on firmware v2.10. These were derived by changing camera settings and comparing the metadata. I have no idea if other models will interpret the information differently, so further experimentation may be required for your own camera.

## Table of Contents

- [Why?](#why)
- [Data Structure](#data-structure)
- [ExifTool Configuration Example](#exiftool-configuration-example)
  - [Bonus: Make It Fancier](#bonus-make-it-fancier)
- [GR IIIx and IV](#gr-iiix-and-iv)

---

## Why?

I wanted to [uncrop a Ricoh GR III DNG](Uncropping.md) so I could use the in-camera RAW developer. All other uncropping techniques I'd seen only allowed tools like Lightroom and Darktable to process the image, but not the in-camera RAW developer.

## Data Structure

It looks like 3 unsigned 8-bit integers, where the 2nd byte is the only one that actually changes.

```text
0 0 0 = L (28mm)
0 5 0 = M (35mm)
0 6 0 = S (50mm)
```

## ExifTool Configuration Example

Here's a quick `exiftool` configuration that will allow you to access and write to the field. See [ExifTool_config](ExifTool_config) for a more complete configuration that covers all the other settings I've found too.

```perl
%Image::ExifTool::UserDefined = (
    'Image::ExifTool::Pentax::Main' => {
        0x0098 => {
            Name     => 'GR3CropMode',
            Writable => 'int8u',
            Count    => 3,
        },
    },
);

1;
```

### Bonus: Make It Fancier

If you want to get fancy, you can use this updated configuration to show the value the same way the camera does.

```perl
%Image::ExifTool::UserDefined = (
    'Image::ExifTool::Pentax::Main' => {
        0x0098 => {
            Name      => 'GR3CropMode',
            Writable  => 'int8u',
            Count     => 3,
            PrintConv => q{
                return 'L (28mm)' if $val eq '0 0 0';
                return 'M (35mm)' if $val eq '0 5 0';
                return 'S (50mm)' if $val eq '0 6 0';
                return "Unknown ($val)";
            },
        },
    },
)
```

It'll then output like this,

```text
GR3 Crop Mode: L (28mm)
```

I find that easier when I don't want to have to remember which mode `0 5 0` was.

---

## GR IIIx and IV

I don't have these camera models in order to test, but Ricoh very handily provide example JPEGs for both the [III/IIIx](https://www.ricoh-imaging.co.jp/english/products/gr-3/ex/) and [IV](https://www.ricoh-imaging.co.jp/english/products/gr-4/ex/).

The examples for the GR IIIx don't have any crop added, but two of the GR IV photos do have a crop applied. They describe photo #3 as having a `35mm` crop, and photo #6 has having a `50mm` crop. The tag values correspond exactly to how they were on my GR III.

```
======== ex-pic02.jpg
GR3 Crop Mode                   : 0 0 0
======== ex-pic03.jpg
GR3 Crop Mode                   : 0 5 0
======== ex-pic06.jpg
GR3 Crop Mode                   : 0 6 0
```

So based on this limited sample, it seems the tag works the same way on the IV.

I searched other places for IIIx sample photos, but pretty much all of them have the metadata stripped. Any I did find just had `0 0 0` as the value.
