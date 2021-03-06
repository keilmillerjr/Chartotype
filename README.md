# Chartotype  

***Char To Type***  
Convert and extend bitmap font dumps into a series of SVG files that are loaded into FontForge and exported as a font file.  
**Chartotype** was made to be used with pixel fonts generated by classic 8-bit and 16-bit computers. It began as a little modification to Andy Teijelo's excellent [Making an MSX font](http://www.ateijelo.com/blog/2016/09/13/making-an-msx-font) and become... something else.  
**Chartotype** also makes it easy to extend the original (usually limited) character set. Just draw or modify characters on the bitmap dump, pairing them to their corresponding unicode character on the text file.  

> Go read Teijelo's blog, by the way. If you are here you probably like the subject and his explanation of the process behind the code is excellent.  

> **Chartotype** was only tested on a Mac but should (could? would?) work on a Windows or Linux PC? Maybe? Tell me.  

> Also take a look at [**Vectotype**](https://github.com/farique1/Chartotype/tree/master/Vectotype) for an Illustrator template and a standalone scripts to convert SVG files into fonts.  

## How to use  

### Running  

**Chartotype** works by reading a bitmap font dump `.png` and matching it with a `.txt` file with *unicode* characters on the same order as the ones on the bitmap, left to right, top to bottom. It will save enlarged B&W `.bmp` versions of the bitmap character and convert each of them to `.svg`. Then, it generates another script to be read inside **FontForge** that will load the `.svg` and create the font file.  

To run the program, open Terminal and go to the **Chartotype** folder. There, run the `Chartotype-A.py` program with the *font* or *system* name as argument.  

`Chartotype-A.py <sys_name> [0|1|2|3]`  

After the system name there can be a multipurpose argument in the form of a single digit from `0` to `3`  
`0` run the program without feedback  
`1` writes a dot for each converted character (default)  
`2` returns complete information on every character converted  
`3` (**not implemented yet** generate only  `Chartotype-B.py` without processing the bitmaps and SVGs)  

### What is needed  

**Chartotype** was developed on Python 2.7 and needs:  
- [**potrace**](http://potrace.sourceforge.net) (the `potrace` executable must be on the `Chartotype` folder)
- The [**Pillow**](https://pillow.readthedocs.io/en/stable/#) Python library  
- [**FontForge**](https://fontforge.github.io/en-US/)  

Inside the **Chartotype** folder there must be sub folders with the name (no spaces) of the intended systems or fonts to be converted. This will be called `sys` from now on.  

Inside the `sys` folder there must be:  

A bitmap font dump `.png` with the characters on a regular grid, cropped to the exact bounding size called `sysCharMap.png`  

![CoCo character map](https://github.com/farique1/Chartotype/blob/master/CoCo/CoCoCharMap.png?raw=true)  

A *unicode* encoded text file following the bitmap character sequence called `sysCharMap.txt`  
```
 !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_↑abcdefghijklmnopqrstuvwxyz{|}~←█▛▜▀▙▌▚▘▟▞▐▝▄▖▗▒
 ```

Also in this folder there must be a file called `sysChartotype.ini` (copy one of the examples) with the following content:  

`[BITMAP]` Section with information about the bitmap cropping and `.bmp` and `.svg` saving settings.  
`char_width =` `NUMBER` The characters horizontal size in pixels.  
`char_height =` `NUMBER` The characters vertical size in pixels.  
`trim_width =` `NUMBER` An horizontal crop factor (in pixels, from the right) for the character (ie. on MSX screen 0 mode the characters are 6x8 versions cropped from the 8x8 originals. The same 8x8 characters bitmap can be used to generate both sets)  
`trim_height =` `NUMBER` Same as above but vertical (from the bottom)  
`resize_factor =` `NUMBER` A factor to scale the bitmap for better vector interpretation. `8` works well.  
`invert_luma =` `BOOLEAN` **Chartotype** tries to distinguish between the foreground and background color to generate a back and white `.bmp` with the font in black even on systems where the font is lighter than the background. However, If the colors are still inverted change this to `True` otherwise use `False`  

`[FONT]` Section with configuration for the export font file .  
`glyph_width =` `NUMBER` The width of the exported font characters (the font will be monospaced).  
`ascent =` `NUMBER` The capital letter height of the output font. (To help the calculation this number can be on the format `NUMBER - NUMBER`, the first number being the full font height and the second being the `descent`)  
`descent =` `NUMBER` The portion below the font baseline (the lower part of the lowercase "g" for instance)  

> It is recommended the height to be a power of 2 for TrueType fonts, usually 1024.  
> So it is common to have the `ascent = 1024 - 128`,  `descent = 128` and `glyph_width = 1024` with `width` and `descent` varying according to the font proportions.  

`fontname =` `STRING` the font name (no spaces).  
`fullname =`  `STRING` the font full name.  
`familyname =`  `STRING` the font family name.  

`[OPTIONAL]` Optional configurations for the exported font.  
`os2_ascent =` `NUMBER` An alternate ascent value.  
`os2_descent =` `NUMBER` An alternate descent value.  
`os2_linegap =` `NUMBER` A line gap value.  

> These values can be left blank.  
> They are useful if you want to add vertical (line) spacing to the font without changing its geometry. Usually, increasing the descent number is enough.  

On the **Chartotype** folder there is a `Chartotype.ini` file with general program settings with content as follows:  

`[CHARTOTYPE]`  
`auto_save =` `BOOLEAN` If `True` automatically saves the **FontForge** project and the `.ttf` font file on the `sys` folder.  
`save_report =` `BOOLEAN` If `True` saves a CSV `.txt` report with information about all the characters converted.  
`save_raw_png =` `BOOLEAN` **Chartotype** uses enlarged B&W `.bmp` files to do the conversion. If this is `True` it will also save the separated original bitmapped characters on `.png` files.  
`delete_folders =` `BOOLEAN` If `True` will delete all the working folders (except the `PNGs`) after the conversion.  

After `Chartotype-A.py` is done, if told to, a `sysReport.txt` will be generated on the `sys` folder with information about all the characters found and the `BMPs` folder will be deleted.  
Also another script will be generated/modified, `Chartotype-B.py`, with all that is needed to make the font file.  

### Chartotype-B  

`Chartotype-B.py` must be called from inside FontForge on the `File > Script Menu` menu (pasting the script straight to the `Execute Script...` box doesn't always work). Configure the `Script Menu` on `File > Preferences... > Script Menu` this only needs to be done once as long as the path to `Chartotype-B.py` doesn't change.  

Once set up, calling `Chartotype-B.py` inside **FontForge** will load each `.sgv`, assign them to their character slot, perform some optimisations and, if told to, save the project, export the font and delete the `SVGs` folder.  

At this time you may want to revise the font on **FontForge** to see if all went according to plan and maybe do some manual adjustments.  

>This is often not needed but the initial `Chartotype-A` font settings can take some trial and error to get right.  

> I specially had problems with an horizontal proportioned font (`A2600 Blocky`, one of the examples) and had to adjust it manually on **FontForge** (configuring as square (ascent+descent=width), selected all the glyphs and changed *size* and *scale* on the `Elements > Transformations > Transform...` menu, checking `Transform Width Too`)  


## Acknowledgements  

Enjoy and send feedback.  
Thanks.  

***Chartotype** is offered as is, with no guaranties whatsoever. I think it will behave however (mostly).*  
