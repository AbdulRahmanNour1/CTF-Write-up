# c0rrupt
---

Hey, dear "hacker"... better quit the field before you find this challenge, why?

While this challenge might seem simple, a 10/10 difficulty rating won't even cover it if you lack knowledge in:
1. Hex digits
2. Hexdumps
3. Offsets
4. PNG file structure
5. Python or any calculator for hex offsets

A difficulty rating of 10/10 won't be enough for it. However, if you understand these concepts, this challenge drops to like 4/10 **because it's only time consuming and nothing else**.

So let's explain these concepts before diving into solutions (skip to the solution if you know):

### Hex digits

We deal with numbers everyday, any number consists of digits. If we talk about decimal digits, they are: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 and by using these digits we can make numbers such as 11, 35, 54 and etc...
Since we use 10 distinct digits (0-9), this is called the base-10 or decimal system.

Any number in any base X system can be converted to base 10 by being written as summation of X^n terms where n starts from 0 until N, such that N is the count of digits in any numbers minus 1.
e.g.,  54 in base 10 = 4x10^0 x 5x10^1 = 4 + 50

Computer at lowest level, only understands 1 and 0, which by the way is called **base 2** system as it digits doesn't exceed 2, so we can't build numbers that exceeds or equal to 2.
Same way, we can convert 1010 in binary (base 2) to decimal (base 10)  by:
(1010) in base 2 = 0x2^0 + 1x2^1 + 0x2^2 +1x2^3 = 10

For Hex, it has same digits as base 10 with 5 additional (can be uppercase or lowercase): 

- A (=10)
- B (=11)
- C (=12)
- D (=13)
- E (=14)
- F (=15)

Any binary number of N digits can hold up to 2^N - 1 as decimal value, let's illustrate with an example:
From 0000 to 1111 in binary is from 0 to 2^4 -1 = 15 in decimal (base 10)
If you notice, Hex digits are up to 15 also, so we can represent any hex digit as a 4-bits, and if there is more, we can concatenate the result:

9 F B A = 1001 1111 1011 1010 = 10011111 10111010
### Hexdump

Now, we can save screen or reading space by dealing with files as hex instead of binary, octal (base 8) and decimal (base 10). As every hex digit can express higher values than decimal digit, this improves readability and inspection.
It doesn't save space, as in memory/storage internally, data is always binary.
Representation of binaries stored in storage devices and RAM as hex are called hexdumps.

### Offsets

The offsets in any hexdump files are written in hex too. (the column on left)

e.g.,
```shell
00000500: b158 7c29 c6bf af9b bfdf f7e8 ccef fad4  .X|)............
00000510: a57d 40d3 30ee aaa7 cc33 e34f f3cf 1e71  .}@.0....3.O...q
00000520: 810b 47a4 96e6 042e 5f4b 527a 86c5 62b1  ..G....._KRz..b.
00000530: 582c 3e8c f5f5 d9a2 cbe5 3fa6 d0e5 bd8a  X,>.......?.....
00000540: 99d9 5c5e 4db1 d214 6c95 3b54 5d2c 168b  ..\^M...l.;T],..
00000550: c5e2 4bf1 59bf a14e 1d8a 79fe aaf2 a73d  ..K.Y..N..y....=
00000560: 3128 89a8 ced8 06a8 f145 13ae cdc7 1cfb  1(.......E......
00000570: 85ae 437c eca9 f94f e1f0 c47a 370f 88a5  ..C|...O...z7...
00000580: 04a9 ea4b 2286 c562 b158 2c3e 9ef5 f5d9  ...K"..b.X,>....
00000590: f767 f047 8d5e a9ea ae34 abc9 e04a b3e4  .g.G.^...4...J..
000005a0: ebb0 0455 afc1 62b1 582c 1693 7ce4 ef0e  ...U..b.X,..|...
000005b0: ffe5 f552 38c5 f719 e48c c063 8226 5493  ...R8......c.&T.
000005c0: a7b6 c856 4b8e 6ca4 955a 4a4a a475 8844  ...VK.l..ZJJ.u.D
000005d0: a1b4 9682 a618 f4f4 2fc2 73ef f675 1e29  ......../.s..u.)
000005e0: 37e1 c957 bc55 168b c562 b1f8 04d6 d767  7..W.U...b.....g
000005f0: df99 c11f 3226 fffc e1b6 d412 294a da7d  ....2&......)J.}
00000600: f5aa 1183 eb52 246a 777a fa62 b158 2c7e  .....R$jwz.b.X,~
```

First row: 0x500, 0x501, 0x502, ...., 0x50F
Second row: 0x510, 0x511, 0x512, ...., 0x51F
.
.
.
### PNG File structure

That's a very tall topic, but I will tell you the general structure that will make you understand the idea behind it and qualify you to research more (I will give the resources at the end).

Almost all files in the world starts with pattern of bytes called magic bytes which is used to identify the file's type, for PNG file, it starts with:

```
89 50 4E 47 0D 0A 1A 0A
```

Which when converted to ASCII gives: PNG and some gibberish 

A **.png** file is made up of chunks of data, where each chunk contains data/information about the image. Each chunk starts with a 4-byte length field, which specifies the number of bytes in the data chunk, not length field or type field or CRC field. This is followed by a 4-byte type field, which identifies the type of data in the chunk. After the type field comes the chunk data, which can be of varying length depending on the type of chunk and length field. Finally, the chunk ends with a 4-byte CRC (Cyclic Redundancy Check) field, which is used to verify the integrity of the chunk data.

Structure of any chunk:
- Length (4 bytes): specifies the number of bytes in the data chunk only. 
- Type (4 bytes): identifies the type of data in the chunk.
- Chunk data (variable length): the actual data contained in the chunk.
- CRC (4 bytes): checksum calculated on the preceding bytes in the chunk, including the chunk type code and chunk data fields, but **not** including the length field.

```
[length (4 bytes)]  
[type (4 bytes)]  
[data (length bytes)]  
[CRC (4 bytes)]
```

A valid PNG image must contain an IHDR chunk as first chunk, one or more **consecutive** IDAT chunks, and an IEND chunk.

Some chunks must exist in every PNG file (with one as special case which we will ignore) and some are optional.

Let's start with critical chunks:

#### IHDR
---

The IHDR chunk must appear FIRST. It contains:

Width: 4 bytes  
Height: 4 bytes  
Bit depth: 1 byte  
Color type: 1 byte  
Compression method: 1 byte  
Filter method: 1 byte  
Interlace method: 1 byte

| Color Type | Allowed Bit Depths | Interpretation                                                 |
| ---------- | ------------------ | -------------------------------------------------------------- |
| 0          | 1, 2, 4, 8, 16     | Each pixel is a grayscale sample.                              |
| 2          | 8, 16              | Each pixel is an R, G, B triple.                               |
| 3          | 1, 2, 4, 8         | Each pixel is a palette index; a PLTE chink must appear        |
| 4          | 8, 16              | Each pixel is a grayscale sample, followed by an alpha sample. |
| 6          | 8, 16              | Each pixel is an R,G,B triple, followed by an alpha sample.    |

#### IDAT Image data
---

The IDAT chunk contains the actual image data.  
i.e contains the output datastream of the compression algorithm.

There can be multiple IDAT chunks; if so, they must appear consecutively with no other intervening chunks. The compressed datastream is then the concatenation of the contents of all the IDAT chunks.

#### IEND Image trailer
---

The IEND chunk must appear LAST. It marks the end of the PNG datastream. **The chunk's data field is empty**.

---
### **PNG Structure**

```
[Magic Bytes]

[IHDR length]
[IHDR]
[IHDR data]
[CRC]

[IDAT length]
[IDAT]
[zlib stream ONLY]
[CRC]

[IEND length = 0]
[IEND]
```
---

Some of Ancillary chunks:

#### gAMA Image gamma
---

The gAMA chunk specifies the gamma of the camera (or simulated camera) that produced the image, and thus the gamma of the image with respect to the original scene. More precisely, the gAMA chunk encodes the `file_gamma` value.

The gAMA chunk contains:

Image gamma: 4 bytes

The value is encoded as a 4-byte unsigned integer, representing gamma times 100000. For example, a gamma of 0.45 would be stored as the integer 45000.

If the gAMA chunk appears, it must precede the first IDAT chunk, and it must also precede the PLTE chunk if present.

#### sRGB Standard RGB color space
---

If the sRGB chunk is present, the image samples conform to the sRGB color space sRGB, and should be displayed using the specified rendering intent as defined by the International Color Consortium ICC.

The sRGB chunk contains:

Rendering intent: 1 byte

The following values are defined for the rendering intent:

0: Perceptual  
1: Relative colorimetric  
2: Saturation  
3: Absolute colorimetric

If the sRGB chunk appears, it must precede the first IDAT chunk, and it must also precede the PLTE chunk if present. The sRGB and iCCP chunks should not both appear.

#### pHYs Physical pixel dimensions
---

The pHYs chunk specifies the intended pixel size or aspect ratio for display of the image. It contains:

Pixels per unit, X axis: 4 bytes (unsigned integer)  
Pixels per unit, Y axis: 4 bytes (unsigned integer)  
Unit specifier: 1 byte (0: unit is unkown, 1: unit is meter)

When the unit specifier is 0, the pHYs chunk defines pixel aspect ratio only; the actual size of the pixels remains unspecified.  
One inch is equal to exactly 0.0254 meters.

If present, this chunk must precede the first IDAT chunk.

THERE ARE OTHER CHUNKS THAT CAN BE SEEN IN REFERENCES

### Python or any calculator for hex
---

Just install python and type in your terminal

```shell
[arch@arch c0rrupt]$ python3
Python 3.14.2 (main, Jan  2 2026, 14:27:39) [GCC 15.2.1 20251112] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x1 + 0x2
3
>>> hex (0x1+0x2)
'0x3'
```

DON'T FORGET TO CONVERT RESULT INTO HEX USING `hex()` IF YOU WANT OFFSET OR VALUE

# Solution
---

Now, you are ready for the solution!

Given a file named `mystery`, we need to fix it... How many possible extension could it be??..!

We can't try them all however much time we have, identifying it's type is a MUST.

```shell
[arch@arch c0rrupt]$ file mystery
mystery: data
```

file is built-in linux tool that allows you to identify the file's type, as you can see, it's just `data`, not very useful as it's like saying "binary"... yes YES we know everything is a binary after all.

Even diec (Detect it easy) won't be able to

```shell
[arch@arch c0rrupt]$ diec mystery
Binary
    Unknown: Unknown
```

We have no choice but to look at the file hexdump

```shell
[arch@arch c0rrupt]$ xxd mystery | less
```

```shell
00000000: 8965 4e34 0d0a b0aa 0000 000d 4322 4452  .eN4........C"DR
00000010: 0000 066a 0000 0447 0802 0000 007c 8bab  ...j...G.....|..
00000020: 7800 0000 0173 5247 4200 aece 1ce9 0000  x....sRGB.......
00000030: 0004 6741 4d41 0000 b18f 0bfc 6105 0000  ..gAMA......a...
00000040: 0009 7048 5973 aa00 1625 0000 1625 0149  ..pHYs...%...%.I
00000050: 5224 f0aa aaff a5ab 4445 5478 5eec bd3f  R$......DETx^..?
00000060: 8e64 cd71 bd2d 8b20 2080 9041 8302 08d0  .d.q.-.  ..A....
00000070: f9ed 40a0 f36e 407b 9023 8f1e d720 8b3e  ..@..n@{.#... .>
00000080: b7c1 0d70 0374 b503 ae41 6bf8 bea8 fbdc  ...p.t...Ak.....
00000090: 3e7d 2a22 336f de5b 55dd 3d3d f920 9188  >}*"3o.[U.==. ..
000000a0: 3871 2232 eb4f 57cf 14e6 25ff e5ff 5b2c  8q"2.OW...%...[,
000000b0: 168b c562 b158 2c16 8bc5 62b1 582c 161d  ...b.X,...b.X,..
000000c0: d6d7 678b c562 b158 2c16 8bc5 62b1 582c  ..g..b.X,...b.X,
000000d0: 168b 4597 f5f5 d962 b158 2c16 8bc5 62b1  ..E....b.X,...b.
000000e0: 582c 168b c562 d165 7d7d b658 2c16 8bc5  X,...b.e}}.X,...
000000f0: 62b1 582c 168b c562 b158 7459 5f9f 2d16  b.X,...b.XtY_.-.
00000100: 8bc5 62b1 582c 168b c562 b158 2c16 5dd6  ..b.X,...b.X,.].
00000110: d767 8bc5 62b1 582c 168b c562 b158 2c16  .g..b.X,...b.X,.
00000120: 8b45 97f5 f5d9 62b1 582c 168b c562 b158  .E....b.X,...b.X
00000130: 2c16 8bc5 62d1 657d 7db6 582c 168b c562  ,...b.e}}.X,...b
00000140: b158 2c16 8bc5 62b1 5874 595f 9f2d 168b  .X,...b.XtY_.-..
00000150: c562 b158 2c16 8bc5 62b1 582c 165d d6d7  .b.X,...b.X,.]..
00000160: 678b c562 b158 2c16 8bc5 62b1 582c 168b  g..b.X,...b.X,..
00000170: 4597 f5f5 d962 b158 2c16 8bc5 62b1 582c  E....b.X,...b.X,
00000180: 168b c562 d165 7d7d b658 2c16 8bc5 62b1  ...b.e}}.X,...b.
00000190: 582c 168b c562 b158 7459 5f9f 2d16 8bc5  X,...b.XtY_.-...
000001a0: 62b1 582c 168b c562 b158 2c16 5dd6 d767  b.X,...b.X,.]..g
:
```

Every row contains 8 group, each group contain 4 hex digits, each pair of hex digits can be represented as a byte, so any row is 16 bytes.

We see some common chunks names we just discussed in PNG files: sRGB, gAMA, pHYs.
That rings the bell and solves the identifying file's type problem, however, a new problem arrives... how to fix such a file?? manually or tool???

Look, you can do it either way and i will show both but i prefer manual one to actually benefit from this challenge.

Before anything, we need to fix the first 8 bytes of the corrupted file to match PNG magic bytes.
We need to change `8965 4e34 0d0a b0aa` to `89 50 4E 47 0D 0A 1A 0A`
**Note:** the space doesn't matter, it just differs among tools and utilities.

To add/delete/modify hexdump, there are 2 options: 
1. https://hexed.it/
2. Install any tool and work from terminal

I will work with first option, but, you can't do that if you are working in a company with sensitive data that can't be exposed.

Now the file looks like

![](attachments/Pasted%20image%2020260322135143.png)

Notice the text representation on the right changed to PNG

If you think we are done already and exported the file to open it... LOL IT WON'T OPEN
Because IHDR chunk is corrupted!

Using our calculator to confirm our calculations and ease up the process
```python
Python 3.14.2 (main, Jan  2 2026, 14:27:39) [GCC 15.2.1 20251112] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 0xB - 0x8 + 1
4
```
*Length = End - Start + 1*

 From 0x8 to 0xB is IHDR length field (4 bytes), assume IHDR length field (from 0x8 to 0xB) is valid (not corrupted) until contradiction happens, from 0xC to 0xF is IHDR chunk type, which is always "IHDR". LET'S FIX IT!

Validating our theory via `pngcheck` tool (Tool method)

```shell
[arch@arch Downloads]$ pngcheck mystery
mystery(1):  invalid chunk name "C"DR" (43 22 44 52)
ERROR: mystery
```

**Note:** if you used the tool before changing the magic bytes, it won't recognize the file as PNG and will raise an error

**Fixed hexdump**

![](attachments/Pasted%20image%2020260322140432.png)

From pHYs chunk description, if it's present, it MUST precede IDAT chunk.
Let's calcalute the offsets of the fields pHYs to help you understand PNG structure deeply:

Offset 0x3E to 0x41 is pHYS length field (4 bytes) which holds a value of **0x9**, 0x42 to 0x45 is chunk type "PHYs" (4 bytes), so what's after is: the data which is exactly equal to length field. (from 0x46 to 0x4E)
So 0x45 + 9 + 1  should give us the offset of CRC

```python
>>> hex(0x45 + 9 + 1)
'0x4f'
```

0x45: end of chunk type
+9: length of data chunk
+1: to get to the start of CRC

Using our python interpreter result, CRC starts from 0x4f to 0x52 (4 bytes)

**Note: I assumed that pHYs chunk is fully valid, but actually the CRC is corrupted too. However, that doesn't affect our work as we before said that pHYs is ancillary chunk (not critical) so fixing it or not won't harm us.**

proof
```shell
[arch@arch Downloads]$ pngcheck mystery
mystery  CRC error in chunk pHYs (computed 38d82c82, expected 495224f0)
ERROR: mystery
```

After finishing our calculation, we are certain that IDAT chunk starts from 0x53.
If we take a look at offsets from 0x57 to 0x5A which belongs to IDAT type field, they are partially corrupted, only 'D' and 'T' are the correct bytes of "IDAT", so let's fix it.

![](attachments/Pasted%20image%2020260322150848.png)

Still, after all what we did, the image doesn't open... why??!!
Cuz, there is only one last thing left... IDAT LENGTH FIELD IS TOO DAMN BIG
WHO SAVES 2.6 TB IMAGE !!
The encoder buffer is gonna be hella big and will continue reading forever.

Proving our theory
```shell
[arch@arch Downloads]$ pngcheck mystery
mystery  invalid chunk length (too large)
ERROR: mystery
```
Notice our tool scan the PNG file **chunk by chunk**

To calculate the valid IDAT length field value, we need to consider the existing of other IDAT chunks, as by definition, there can be more than one but must be **consecutive**.

We found another one by searching (CTRL + F)

![](attachments/Pasted%20image%2020260322151904.png)

Our value is gonna be: Second IDAT length offset - (First IDAT data field offset + 4 + 1) 

```python
>>> hex(0x00010004 - (0x5A + 4 + 1))
'0xffa5'
```

1:  to not count the first byte of Second IDAT length offset

**IDAT length field fixed**

![](attachments/Pasted%20image%2020260322152747.png)

oohh... OHHHHHH SSH- SHINE LIKE A DIAMOND !

![](attachments/flag.png)

Thanks for reading all of that hacker bs, feel free to ask me anything

My GitHub CTF repo: ... (Updated periodically)
*I may download automated solution for this challenge*

Follow for more solutions!

---
# References

[https://www.w3.org/TR/PNG-Structure.html](https://www.w3.org/TR/PNG-Structure.html)  
[https://www.w3.org/TR/PNG-Chunks.html](https://www.w3.org/TR/PNG-Chunks.html)  
[https://www.libpng.org/pub/png/spec/1.2/PNG-Chunks.html](https://www.libpng.org/pub/png/spec/1.2/PNG-Chunks.html)
