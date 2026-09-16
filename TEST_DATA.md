# Raw Test Data

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
