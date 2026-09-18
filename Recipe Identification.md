# Identifying Ricoh GR III Recipes

Recipes are a collection of "Image Control" settings that give your photos a specific look and feel. Great for when you don't want to deal with post-processing a RAW image and just want a JPEG that looks a certain way straight out of the camera.

One of the benefits of decoding the [`Pentax_0x0247`](Pentax_0x0247.md) tag is that you can also now define your recipes and have the name show up as a new tag.

See the [`ExifTool_config`](ExifTool_config) file for how everything fits together. There's an example at the end defining the [Reggie's Color Negative](https://reggiebphotography.com/blog/The-Most-Versatile-Ricoh-GR-III-GR-IIIx-Film-Simulation-Recipe-Reggies-Color-Negative) recipe as a starting point.

```perl
  return 'Reggie\'s Color Negative'
      if $self->GetValue('ImageTone') eq 'Negative Film'
      && $settings eq '2,0,0,3,-4,-1,1,0,0'
      && $self->GetValue('GR3HighlightCorrection') eq 'Auto'
      && $self->GetValue('ShadowCorrection') eq 'Normal'
      && $self->GetValue('HighISONoiseReduction') eq 'Off; Inactive'
      && $self->GetValue('WhiteBalance') =~ /Auto/
      && $self->GetValue('GR3WBShiftName') eq 'A6'
      ;
```

Any defined recipes will show up under a new `GR3Recipe` tag.

```bash
exiftool -GR3Recipe RCN.JPG
GR3 Recipe : Reggie's Color Negative
```

You can add as many recipe definitions as you want by adding more `return` statements, using any tags that are available (not just the new ones the config file defines).

I use this technique to document all of my recipes and have it show up in my photo management tools. Very convenient when I'm struggling to remember what settings I used and whether it was part of a recipe or just some ad-hoc experimentation.
