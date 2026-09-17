# Uncropping a Ricoh GR III DNG

Sometimes I accidentally shoot a photo in 35mm or 50mm crop modes but want the full image later. When shooting in RAW, the uncropped information is all in the DNG file but is not easily exposed.

There have been [other helpful posts](https://www.johnmaguire.me/blog/uncrop-ricoh-gr-iii-photos/) on how to modify `DefaultCropOrigin` and `DefaultCropSize` to uncrop the image so that tools like Lightroom and Darktable can see it.

```bash
exiftool \
  -DefaultCropOrigin="5 6" \
  -DefaultCropSize="6000 4000" \
  <FILE>
```

Unfortunately that doesn't work for in-camera processing. I'm lazy and wanted to be able to use the in-camera RAW processing to apply the normal JPEG look without dealing with more professional software. Turns out there's a MakerNote field the camera uses internally for this: [`Pentax_0x0098`](Pentax_0x0098.md).

Using a custom `exiftool` configuration we can write a new value to it. See my full [ExifTool_config](ExifTool_config) file if you want all the features, but if you only care about the crop mode you can save the below as `gr3.config` and then reference it directly when using `exiftool`.


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

Write `0 0 0` as the value to uncrop the image,

```bash
exiftool \
  -config gr3.config \
  -GR3CropMode="0 0 0" \
  <FILE>
```

After running the above, rename the file then add it back to your camera SD card, and you should now be able to use the in-camera RAW processor to get the original full size image!

> [!NOTE]
> When browsing the photo in-camera, it'll still only show the cropped version since it's using the embedded thumbnail and we didn't touch that. Once you enter RAW development mode it'll show the full uncropped image ready for processing.
