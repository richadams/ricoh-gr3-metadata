# Ricoh GR III Metadata Reverse Engineering

_My attempts at figuring out how to get settings from the Ricoh GR III's vendor-specific EXIF metadata; image control settings, crop modes, etc. None of this should be considered authoritative, these are essentially just research notes, but I figured they might be helpful to others._

> [!NOTE]
> Tested only with a Ricoh GR III on firmware v2.10. These were derived by changing camera settings and comparing the data on my camera. I have no idea if it's the same for the GR IIIx or any other flavours of GR camera (or even if it's specific to just my camera), so further experimentation may be required for your own.


## tl;dr

Save [ExifTool_config](ExifTool_config) as `~/.ExifTool_config`, then use this to see all of the settings in the order they'd normally be shown in the camera.

```bash
exiftool \
  -GR3CropModeName \
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
  -GR3CrossProcessingColorTone \
  -GR3HDRToneToning \
  -GR3HDRToneHDRToneLevel \
  -GR3Recipe \
  <FILE>
```

Here's an example of what the output should look like,

```text
GR3 Crop Mode Name              : L (28mm)
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

By having these available in `exiftool` they should also show up in any other software that uses it. For example, I have them shown in the Metadata sidebar in digiKam with a custom filter.

If you don't want the configuration to always be used with `exiftool`, you can save it as another file such as `~/.gr3.config` and then use `exiftool -config ~/.gr3.config` when you want to use it.

---

## Mapping GR III Image Control Metadata Fields

While `exiftool` already pulls _some_ values out, they tend to not be as useful as the raw numbers. Does `Saturation: High` mean `+1`, `+2`, `+3`, or `+4`?, etc. I want the raw numbers.

I took photos cycling through all the settings, then compared the metadata bytes to see where the changes are stored.

Here are some of my notes on the format. There are still some unknowns, and this is based entirely on what my Ricoh GR III produced, so it might be different for your own camera or other models. At the very least it should provide a good starting point.

### Pentax 0x0247 - Data Structure

The `Pentax_0x0247` MakerNote field seems to be a 44-byte blob where most of the image control settings are stored as signed 16-bit little-endian integers, with a few 4-byte values and some unknowns thrown in for good measure. Here are the fields I was able to identify:

```text
Offset   Size   Field                         Encoding
0        2      Saturation                    int16s LE
2        2      Hue                           int16s LE
4        2      High/Low Key                  int16s LE
6        2      Contrast                      int16s LE
8        2      Contrast Highlight            int16s LE
10       2      Contrast Shadow               int16s LE
12       2      Sharpness                     int16s LE
14       4      !! Unknown                    -
18       2      Shading                       int16s LE
20       2      Clarity                       int16s LE
22       4      !! Unknown                    -
26       2      BW Toning                     int16s LE
28       2      !! Unknown                    -
30       2      HDR Tone Toning               int16s LE
32       2      HDR Tone Level                int16s LE
34       4      BW Filter Effect              4 raw bytes
38       2      BW Grain Effect               int16s LE
40       2      !! Unknown                    -
42       2      Cross Processing Color Tone   int16s LE
```

A custom config can get `exiftool` to create new composite tags for each of them by unpacking the relevant bytes and either passing the value back or doing a lookup. See [ExifTool_config](ExifTool_config) for the full configuration.

```perl
%Image::ExifTool::UserDefined = (
    'Image::ExifTool::Pentax::Main' => {
        0x0247 => {
            Name     => 'GR3ImageControlData',
            Writable => 'undef',
            Count    => 44,
        },
    },

    'Image::ExifTool::Composite' => {
        GR3Saturation => {
            Require   => 'GR3ImageControlData',
            Condition => '$self->GetValue("ImageTone") ne "HDR Tone"',
            ValueConv => q{
                my $v = unpack('s<', substr($val, 0, 2));
                return 'N/A' if $v == -32768;
                return $v;
            },
        },
        # ... etc
    }
)
```

For the ones that require lookups, here are the mappings I was able to figure out.

#### Standard Image Control Settings

All the normal settings such as Saturation, Contrast, Sharpness, etc. are just signed integers that exactly match what the camera menu already shows. `-2` = `-2`, `+3` = `+3`, etc.

One exception is `-32768` for when the value isn't used, such as in "HDR Tone" mode, since the settings are unavailable.

#### B&W / Monotone Specific Settings

There are 3 settings which only show up in B&W/Monotone modes. Unlike the normal contrast/saturation-style fields, these aren't exposed as numeric values on the camera and instead have a mapping. Cycling through all of the options got me these values.

##### B&W Toning

```text
-1 = N/A (i.e. camera isn't in a B&W/Monotone mode)
 0 = Off
 1 = Brown
 2 = Red
 3 = Green
 4 = Blue
 5 = Purple
```

##### B&W Filter Effect

This one is 4-bytes and not the normal signed integers. I'm guessing the bytes refer to what the filters do in some way?

```text
08000000 = N/A (i.e. camera isn't in a B&W/Monotone mode)
00000000 = Off
010a4614 = 1
0128320a = 2
415a140a = 3
61780a0a = 4
```

##### B&W Grain Effect

This one seems to be in reverse order, and odd numbers only, except for the `N/A` value which is `8` instead of `-1` like it is for others.

```text
0 = Off
1 = 3
3 = 2
5 = 1
8 = N/A (i.e. camera isn't in a B&W/Monotone mode)
```

#### Cross Processing Specific Settings

##### Color Tone

This only shows up in the "Cross Processing 2" image control mode. It's called "Cross Processing" on my camera, but `exiftool -ImageTone` shows it as "Cross Processing 2".

```text
-1 = N/A (i.e. camera isn't in the "Cross Processing 2" mode)
 1 = Blue
 2 = Magenta
 3 = Yellow
```

#### HDR Tone

These 2 settings are only available in the "HDR Tone" mode. All the normal saturation/contrast-style settings also become unavailable in this mode.

##### Toning

```text
0 = Off
1 = BW
2 = S
```

##### HDR Tone Level

```text
1 = Low
2 = Med
3 = High
```

### Extra Credit - Recipes

With all of the image control settings now available in `exiftool`, you can also add configurations for your recipes and have them appear as a new tag.

Here's an example using [Reggie's Color Negative](https://reggiebphotography.com/blog/The-Most-Versatile-Ricoh-GR-III-GR-IIIx-Film-Simulation-Recipe-Reggies-Color-Negative), where it'll now show up under a new `GR3 Recipe` tag.

See [ExifTool_config](ExifTool_config) for how it all fits together.

```perl
GR3Recipe => {
    Require   => 'GR3ImageControlData',
    ValueConv => q{
        my $settings = join(',', map {
            unpack('s<', substr($val, $_, 2))
        } (0, 2, 4, 6, 8, 10, 12, 18, 20));

        return 'Reggie\'s Color Negative'
            if $self->GetValue('ImageTone') eq 'Negative Film'
            && $settings eq '2,0,0,3,-4,-1,1,0,0'
            && $self->GetValue('GR3HighlightCorrection') eq 'Auto'
            && $self->GetValue('ShadowCorrection') eq 'Normal'
            && $self->GetValue('HighISONoiseReduction') eq 'Off; Inactive'
            && $self->GetValue('WhiteBalance') =~ /Auto/
            && $self->GetValue('GR3WBShiftABName') eq 'A6'
            && $self->GetValue('GR3WBShiftGMName') eq '0'
            ;

        # Add more recipes here...

        return undef;
    },
},
```

```bash
exiftool -GR3Recipe RCN.JPG
GR3 Recipe : Reggie's Color Negative
```

I use this to document all of my recipes and have it show up in my photo management tools. Very convenient when I'm struggling to remember what settings I used and whether it was part of a recipe or just some ad-hoc experimentation.

---

## Uncropping a Ricoh GR III DNG

Sometimes I accidentally shoot a photo in 35mm or 50mm crop modes but want the full image later. When shooting in RAW, the uncropped information is all in the DNG file but is not easily exposed.

There have been [other helpful posts](https://www.johnmaguire.me/blog/uncrop-ricoh-gr-iii-photos/) on how to modify `DefaultCropOrigin` and `DefaultCropSize` to uncrop the image so that tools like Lightroom and Darktable can see it.

```bash
exiftool \
  -overwrite_original \
  -DefaultCropOrigin="5 6" \
  -DefaultCropSize="6000 4000" \
  <FILE>
```

Unfortunately that doesn't work for in-camera processing. I'm lazy and wanted to be able to use the in-camera RAW processing to apply the normal JPEG look without dealing with more professional software. Turns out there's a MakerNote field the camera uses for this: `Pentax_0x0098`.

### Pentax 0x0098 - Data Structure

It seems to be 3 unsigned 8-bit integers, where the 2nd byte is the only one that changes.

* `0 0 0` = L (28mm)
* `0 5 0` = M (35mm)
* `0 6 0` = S (50mm)


A custom `exiftool` configuration allows me to access the field and write to it. See my full [ExifTool_config](ExifTool_config) file if you want the entire thing, but if you only care about the crop mode you can save this as `gr3.config` or something and then reference it directly when using `exiftool`.

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

Then to read the value,

```bash
exiftool \
  -config gr3.config \
  -GR3CropMode \
  <FILE>

GR3CropMode : 0 5 0
```

And to write the value to uncrop the image,

```bash
exiftool \
  -config gr3.config \
  -GR3CropMode="0 0 0" \
  <FILE>
```

After running the above, rename the file then add it back to your camera SD card, and you can now use the in-camera RAW processor to get the original full size image!

> [!NOTE]
> When browsing the photo in-camera, it'll still only show the cropped version since it's using the embedded thumbnail and we didn't touch that. Once you enter RAW development mode it'll show the full uncropped image ready for processing.

### Extra Credit - Human Readable Crop Mode Value

If you want to get fancy, you can use this configuration to also show a human readable version of the value. That's what I've done in my larger [`exiftool` configuration](ExifTool_config).

```perl
%Image::ExifTool::UserDefined = (
    'Image::ExifTool::Pentax::Main' => {
        0x0098 => {
            Name     => 'GR3CropMode',
            Writable => 'int8u',
            Count    => 3,
        },
    },

    'Image::ExifTool::Composite' => {
        GR3CropModeName => {
            Require   => 'GR3CropMode',
            ValueConv => q{
                return 'L (28mm)' if $val eq '0 0 0';
                return 'M (35mm)' if $val eq '0 5 0';
                return 'S (50mm)' if $val eq '0 6 0';
                return "Unknown ($val)";
            },
        },
    }
)
```

It'll output like this,

```text
GR3 Crop Mode Name: L (28mm)
```

I find that easier when I don't want to have to remember which mode `0 5 0` was.
