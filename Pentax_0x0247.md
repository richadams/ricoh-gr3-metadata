# Pentax 0x0247

> [!NOTE]
> Tested only with a Ricoh GR III on firmware v2.10. These were derived by changing camera settings and comparing the metadata. I have no idea if other models will interpret the information differently, so further experimentation may be required for your own camera.

The `Pentax_0x0247` MakerNote field seems to be where the "Image Control" settings are stored for the Ricoh GR III camera.

## Table of Contents

- [Why?](#why)
- [Data Structure](#data-structure)
- [Mappings](#mappings)
  - [Standard Image Control Settings](#standard-image-control-settings)
  - [B&W / Monotone Specific Settings](#bw--monotone-specific-settings)
    - [B&W Toning](#bw-toning)
    - [B&W Filter Effect](#bw-filter-effect)
    - [B&W Grain Effect](#bw-grain-effect)
  - [Bleach Bypass Specific Settings](#bleach-bypass-specific-settings)
    - [Toning](#toning)
  - [Retro Specific Settings](#retro-specific-settings)
    - [Toning](#toning-1)
  - [HDR Tone Specific Settings](#hdr-tone-specific-settings)
    - [Toning](#toning-2)
    - [HDR Tone Level](#hdr-tone-level)
  - [Cross Processing Specific Settings](#cross-processing-specific-settings)
    - [Color Tone](#color-tone)
- [ExifTool Configuration](#exiftool-configuration)
  - [Bonus - Recipes](#bonus---recipes)
- [Testing Methodology](#testing-methodology)
- [Raw Test Data](#raw-test-data)

## Why?

While `exiftool` already pulls out some image adjustment values, they are the generic EXIF tags rather than the vendor-specific ones. For example, while `Saturation: High` is accurate based on the generic `0xa409` tag, I don't know whether that corresponds to `+1`, `+2`, `+3`, or `+4` on the Ricoh GR III. I want the raw camera values. There are also lots of other Ricoh-specific image control settings that don't seem to be available at all and I would like those too.

## Data Structure

The field appears to be a 44-byte blob where the image control settings are stored mainly as signed 16-bit little-endian integers, with a few unsigned values and some unknowns thrown in for good measure. Here are the fields I've been able to identify so far:

```text
Offset   Size   Field                         Encoding
0        2      Saturation                    int16s LE
2        2      Hue                           int16s LE
4        2      High/Low Key                  int16s LE
6        2      Contrast                      int16s LE
8        2      Contrast Highlight            int16s LE
10       2      Contrast Shadow               int16s LE
12       2      Sharpness                     int16s LE
14       2      !! Unknown                    -
16       2      Retro Toning                  int16s LE
18       2      Shading                       int16s LE
20       2      Clarity                       int16s LE
22       4      !! Unknown                    -
26       2      BW Toning                     int16s LE
28       2      Bleach Bypass Toning          int16s LE
30       2      HDR Tone Toning               int16s LE
32       2      HDR Tone Level                int16s LE
34       1      BW Filter Effect Flags        int8u (bitmask)
35       1      BW Filter Effect R%           int8u
36       1      BW Filter Effect G%           int8u
37       1      BW Filter Effect B%           int8u
38       2      BW Grain Effect               int16s LE
40       2      !! Unknown                    -
42       2      Cross Processing Color Tone   int16s LE
```

## Mappings

Here are the mappings I've been able to figure out:

### Standard Image Control Settings

All the normal settings such as Saturation, Contrast, Sharpness, etc. are just signed integers that exactly match what the camera menu already shows. `-2` = `-2`, `+3` = `+3`, etc.

One exception is `-32768` for when the value isn't used, such as in "HDR Tone" mode, since many of the settings are unavailable.

### B&W / Monotone Specific Settings

There are 3 settings which only show up in B&W/Monotone modes. Unlike the normal contrast/saturation-style fields, these aren't exposed as numeric values on the camera and instead have a mapping. Cycling through all of the options got me these values.

#### B&W Toning

```text
-1 = N/A (i.e. camera isn't in a B&W/Monotone mode)
 0 = Off
 1 = Sepia
 2 = Red
 3 = Green
 4 = Blue
 5 = Purple
```

#### B&W Filter Effect

This is not available in "Hard BW" mode for some reason, but is available in all the other BW/Monotone modes.

The values on the camera are "Off, 1, 2, 3, 4", but the entire section in the metadata is 4-bytes rather than just a single signed integer like the others.

I later noticed the presets can be configured further by pressing "Fn", allowing you to set specific R, G, and B percentages (from -200% to +200%). The presets are just specific combinations of those. The full 4-bytes encode all of this information.

The first byte encodes whether the filter mode is off/on, and whether the RGB values are negative or not. Then the next 3 bytes are the values for R, G, and B respectively.

##### Byte 1

The first nibble encodes the sign for the RGB values and are bitwise OR'd together.

```text
0x10 = R negative
0x20 = G negative
0x40 = B negative
```

So a value of `0x60` would indicate that both G and B values are negative.

The second nibble encodes whether the "Filter Effect" is off or on, or we're in a mode that doesn't support this setting.

```text
0x00 = Off
0x01 = On
0x08 = N/A (i.e. camera isn't in a B&W/Monotone mode)
```

So all together, a value of `0x61` would indicate the "Filter Effect" is on, and the values for G and B should be taken as negative.

##### Bytes 2-4

The next 3 bytes are just the RGB values (in that order) as unsigned 8-bit integers.

##### Presets

Here are the presets in my GR III, and the RGB values that show up when viewing in the camera, compared to the bytes.

```text
010a4614 = Preset 1 (R+10%, G+70%, B+20%)
0128320a = Preset 2 (R+40%, G+50%, B+10%)
415a140a = Preset 3 (R+90%, G+20%, B-10%)
61780a0a = Preset 4 (R+120%, G-10%, B-10%)
```

#### B&W Grain Effect

This one seems to be in reverse order, and odd numbers only, except for the `N/A` value which is `8` instead of `-1` like it is for others.

```text
0 = Off
1 = 3
3 = 2
5 = 1
8 = N/A (i.e. camera isn't in a B&W/Monotone mode)
```

### Bleach Bypass Specific Settings

#### Toning

This only shows up in the "Bleach Bypass 2" image control mode. It's just called "Bleach Bypass" on my camera though.

The values on the camera are "C" and "W" with coloured dots showing blue and red, so presumably the values mean "Cold" and "Warm".

```text
-1 = N/A (i.e. camera isn't in Bleach Bypass mode)
 0 = Off
 1 = C
 2 = W
```

### Retro Specific Settings

#### Toning

This only shows up in the "Retro" mode, and is a normal -4 to +4 value range like Saturation, Contrast, etc. It maps directly to what the camera already displays.

### HDR Tone Specific Settings

These 2 settings are only available in the "HDR Tone" mode. All the normal saturation/contrast-style settings also become unavailable in this mode.

#### Toning

```text
0 = Off
1 = BW
2 = S
```

#### HDR Tone Level

```text
1 = Low
2 = Med
3 = High
```

### Cross Processing Specific Settings

#### Color Tone

This only shows up in the "Cross Processing 2" image control mode. It's called "Cross Processing" on my camera, but `exiftool -ImageTone` shows it as "Cross Processing 2".

```text
-1 = N/A (i.e. camera isn't in the "Cross Processing 2" mode)
 1 = Blue
 2 = Magenta
 3 = Yellow
```

## ExifTool Configuration

See [ExifTool_config](ExifTool_config) for the full configuration, but the general idea is to extract the image control data, and then create new composite tags for each of the settings by unpacking the relevant bytes and either passing the value back or doing a lookup.

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
            ValueConv => 'unpack("s<", substr($val, 0, 2))',
            PrintConv => q{
                return 'N/A' if $val == -32768;
                return $val;
            },
        },

        # ... etc
    }
)
```

### Bonus: Recipes

With all of the image control settings available in `exiftool`, you can also add configurations for your recipes and have them appear as a new tag.

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

## Testing Methodology

For those interested in the exact technique I used to figure out which byte was which. I took a photo and then used the in-camera RAW developer to re-process it with all of the various settings. By diffing the output I found that `Pentax_0x0247` was the changing field. I dumped the `Pentax_0x0247` bytes using my custom field and compared them to see which values changed.

```bash
exiftool -b -GR3ImageControlData <FILE> | xxd -p -c 44
```

Rather than going through _literally_ every single possibility, I only needed to ensure each value was unique in some way and would allow me to fully isolate it. For example, for the main settings I ended up with 5 photos, A-E with these settings.

```text
             A   B   C   D   E
Saturation  -4  -2   0  +2  +4
Hue         -3  +1  +4  -1  +2
High/Low    -2  +3  -1  +4   0
Contrast    -1  +4  +2   0  -3
Highlight    0  -3  +1  +4  -2
Shadow      +1  -4  +3  -2   0
Sharpness   +2   0  -4  +3  -1
Shading     +3  -1  -2  +1  -4
Clarity     +4  +2  -3  -4  +1
```

Then comparing the bytes to figure out which was which and what the values meant. Because each one follows a specific pattern, it made them easier to isolate.

```bash
for f in A.JPG B.JPG C.JPG D.JPG E.JPG; do
  printf '%s: ' "$f"
  exiftool -b -GR3ImageControlData "$f" | xxd -p -c 44
done

A.JPG: fcfffdfffeffffff0000010002000080008003000400ffffffffffffffffffffffff0800000008000000ffff
B.JPG: feff010003000400fdfffcff000000800080ffff0200ffffffffffffffffffffffff0800000008000000ffff
C.JPG: 00000400ffff020001000300fcff00800080fefffdffffffffffffffffffffffffff0800000008000000ffff
D.JPG: 0200ffff040000000400feff0300008000800100fcffffffffffffffffffffffffff0800000008000000ffff
E.JPG: 040002000000fdfffeff0000ffff00800080fcff0100ffffffffffffffffffffffff0800000008000000ffff
```

I repeated this same idea for the settings that are specific to the B&W/Monotone modes, as well as the "Cross Processing" and "HDR Tone" ones. I think I ended up with about 30-40 photos overall, since I had to repeat some when I'd noted down the wrong values and couldn't figure out why nothing matched up properly.

### Settings Matrix

Since I missed a few settings the first time around, I wanted to make sure I'd gotten all of them. I found a [post on the GR blog](https://www.grblog.jp/en/article/1976/) with [a matrix of settings for the BW modes](https://www.grblog.jp/en/wp-content/uploads/sites/2/article/1976/BWMATRIX2.jpg) (albeit from an older firmware version, so no longer accurate), but couldn't find anything for any of the other modes. So I made myself a new one.

These are all the possible Image Control settings I could find on my GR III camera, and their possible values.

![](ricoh_gr3_setting_matrix.png)

---

## Raw Test Data

Here's the raw data I got from my testing in case it's useful to anyone. For each one I list the settings used for each photo, and then the 44 bytes for field `Pentax_0x0247` from the resulting files.

### Standard GR III Image Control Settings

```text
             A   B   C   D   E
Saturation  -4  -2   0  +2  +4
Hue         -3  +1  +4  -1  +2
High/Low    -2  +3  -1  +4   0
Contrast    -1  +4  +2   0  -3
Highlight    0  -3  +1  +4  -2
Shadow      +1  -4  +3  -2   0
Sharpness   +2   0  -4  +3  -1
Shading     +3  -1  -2  +1  -4
Clarity     +4  +2  -3  -4  +1
```

```text
A.JPG: fcfffdfffeffffff0000010002000080008003000400ffffffffffffffffffffffff0800000008000000ffff
B.JPG: feff010003000400fdfffcff000000800080ffff0200ffffffffffffffffffffffff0800000008000000ffff
C.JPG: 00000400ffff020001000300fcff00800080fefffdffffffffffffffffffffffffff0800000008000000ffff
D.JPG: 0200ffff040000000400feff0300008000800100fcffffffffffffffffffffffffff0800000008000000ffff
E.JPG: 040002000000fdfffeff0000ffff00800080fcff0100ffffffffffffffffffffffff0800000008000000ffff
```

### B&W / Monotone Settings

```text
                    A     B     C     D      E     F
Toning              Off   Sepia Red   Purple Green Blue
Filter Effect       Off   1     2     4      Off   Off
Grain Effect        Off   1     2     3      Off   Off
```

```text
A.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0000ffffffffffff0000000000000000ffff
B.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0100ffffffffffff010a461405000000ffff
C.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0200ffffffffffff0128320a03000000ffff
D.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0500ffffffffffff61780a0a01000000ffff
E.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0300ffffffffffff0000000000000000ffff
F.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0400ffffffffffff0000000000000000ffff
```

I didn't have enough info to fully isolate filter and grain effects, so did another set just for those.

```text
             A    B    C    D
Filter       Off  1    2    4
Grain        1    2    3    Off
```

```text
A.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0000ffffffffffff0000000005000000ffff
B.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0000ffffffffffff010a461403000000ffff
C.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0000ffffffffffff0128320a01000000ffff
D.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0000ffffffffffff61780a0a00000000ffff
```

This is when I realized the filter effect one wasn't a simple single byte change like the other fields. So I needed an example of filter effect #3 to get the full pattern for it as I couldn't just infer it like the numeric values earlier.

```text
             A
Filter       3
```

```text
3.JPG: 008000800100feff00000000ffff008000800000fdffffffffff0000ffffffffffff415a140a00000000ffff
```

Update: I didn't notice "Filter Effect" has even more options! Pressing "Fn" brings up the ability to change R, G, and B values in the range from -200% to +200%. The presets are just specific RGB values. That's why they're multiple bytes. Here I cycle through some values for each to properly isolate them.

```text
         A       B        C        D        E        F        G
R        0       +100     -100     0        0        0        0
G        0       0        0        +100     -100     0        0
B        0       0        0        0        0        +100     -100
```

```text
A.JPG: 00800080000000000000000000000080008000000000ffffffff0000ffffffffffff0100000000000000ffff
B.JPG: 00800080000000000000000000000080008000000000ffffffff0000ffffffffffff0164000000000000ffff
C.JPG: 00800080000000000000000000000080008000000000ffffffff0000ffffffffffff1164000000000000ffff
D.JPG: 00800080000000000000000000000080008000000000ffffffff0000ffffffffffff0100640000000000ffff
E.JPG: 00800080000000000000000000000080008000000000ffffffff0000ffffffffffff2100640000000000ffff
F.JPG: 00800080000000000000000000000080008000000000ffffffff0000ffffffffffff0100006400000000ffff
G.JPG: 00800080000000000000000000000080008000000000ffffffff0000ffffffffffff4100006400000000ffff
```

### Bleach Bypass Settings

```text
             A    B    C
Toning       Off  C    W
```

```text
A.JPG: 00000000040004000400000000000080008000000000ffffffffffff0000ffffffff0800000008000000ffff
B.JPG: 00000000040004000400000000000080008000000000ffffffffffff0100ffffffff0800000008000000ffff
C.JPG: 00000000040004000400000000000080008000000000ffffffffffff0200ffffffff0800000008000000ffff
```

### Retro Settings

A-I, go sequentially from -4 to +4 on the values.

```text
A.JPG: 000000000200fdff00000000fdff0080fcfffdff0000ffffffffffffffffffffffff0800000008000000ffff
B.JPG: 000000000200fdff00000000fdff0080fdfffdff0000ffffffffffffffffffffffff0800000008000000ffff
C.JPG: 000000000200fdff00000000fdff0080fefffdff0000ffffffffffffffffffffffff0800000008000000ffff
D.JPG: 000000000200fdff00000000fdff0080fffffdff0000ffffffffffffffffffffffff0800000008000000ffff
E.JPG: 000000000200fdff00000000fdff00800000fdff0000ffffffffffffffffffffffff0800000008000000ffff
F.JPG: 000000000200fdff00000000fdff00800100fdff0000ffffffffffffffffffffffff0800000008000000ffff
G.JPG: 000000000200fdff00000000fdff00800200fdff0000ffffffffffffffffffffffff0800000008000000ffff
H.JPG: 000000000200fdff00000000fdff00800300fdff0000ffffffffffffffffffffffff0800000008000000ffff
I.JPG: 000000000200fdff00000000fdff00800400fdff0000ffffffffffffffffffffffff0800000008000000ffff
```

### HDR Tone Settings

```text
        A       B       C       D       E
Toning  Off     Off     BW      BW      S
Level   Low     Med     Low     High    Med
```

```text
A.JPG: 02000000008000800080008000800080008012000080ffffffffffffffff000001000800000008000000ffff
B.JPG: 02000000008000800080008000800080008012000080ffffffffffffffff000002000800000008000000ffff
C.JPG: 02000000008000800080008000800080008012000080ffffffffffffffff010001000800000008000000ffff
D.JPG: 02000000008000800080008000800080008012000080ffffffffffffffff010003000800000008000000ffff
E.JPG: 02000000008000800080008000800080008012000080ffffffffffffffff020002000800000008000000ffff
```

### Cross Processing Settings

```text
             A       B        C
Color Tone   Blue    Magenta  Yellow
```

```text
A.JPG: 00000000000000000000000000000080008000000000ffffffffffffffffffffffff08000000080000000100
B.JPG: 00000000000000000000000000000080008000000000ffffffffffffffffffffffff08000000080000000200
C.JPG: 00000000000000000000000000000080008000000000ffffffffffffffffffffffff08000000080000000300
```
