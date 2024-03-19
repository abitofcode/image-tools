# Image processing image

A Docker container designed to simplify image conversion to the JPEG2000 format without cluttering the user's machine. It's based on the Harvard Library Imaging Services' straightforward build instructions for [installing OpenJPEG on Windows 10, Linux, and MacOS](https://wiki.harvard.edu/confluence/display/DigitalImaging/Installing+OpenJPEG+on+Windows+10%2C+Linux%2C+and+MacOS). This container encapsulates all necessary dependencies, providing users with a clean and efficient solution for their conversion needs.

## Building a multiplatform image on a Mac M1

The image is available on dockerhub at abitofcode/image-tools or you can build your own. The following instructions are for building a multiplatform build and pushing to dockerhub on a Mac M1. Takes a while...

```sh
# Create a new builder
docker buildx create --use
# Run a multiplatform build and push it to a repo
docker build --push --platform linux/amd64,linux/arm64/v8 --tag abitofcode/image-tools:1 .
```

## Running the image

From the directory with the images to be processed run the following command to start the container. The current directory is mapped to the _app_ directory in the container.

```sh
# Mac/Linux
docker run -it --rm -v $(pwd):/app abitofcode/image-tools
# Windows
docker run -it --rm -v %cd%:/app abitofcode/image-tools
```

## Sample image files

A selection of sample image files can be found at the following locations:

- [Sample files](https://people.math.sc.edu/Burkardt/data/tif/tif.html)
- [Sample Tiff](https://file-examples.com/index.php/sample-images-download/sample-tiff-download/)
- [Sample images with exif metadata](https://wiki.harvard.edu/confluence/display/DigitalImaging/Embedding+derived+JPEG2000+image+files+with+image+technical+metadata+transferred+from+source+images)

these can be used to test the tools.

## Test the tools

Some examples of the tools running are below. The commands are run from the terminal in the container.

```bash
# Check version
identify -version
# Check JPeg200 is supported
identify -list format | grep JP
# test mogrify
mogrify -format jp2 *.tiff
# Test convert
convert sample.tiff -quality 50 -define jp2:tilewidth=256 -define jp2:tileheight=256 sample2.jp2
# Check the new file
identify sample2.jp2
# Testing OpenJPEG installation (lossy compression example)
opj_compress -i sample.tiff -o sample_lossy_q42_opj.jp2 -p RLCP -t 1024,1024 -EPH -SOP -I -q 42
# Testing GrokImageCompression installation (lossy compression example)
grk_compress -i sample.tiff -o sample_lossy_q42_grk.jp2 -p RLCP -t 1024,1024 -EPH -SOP -I -q 42
# Are the JP2 files you created standard-compliant?
jpylyzer *.jp2 | grep '<isValid format="jp2">True</isValid>'
# Removing alpha channel
convert ALPHA_ISSUE_FILE.tif -alpha off  ALPHA_ISSUE_FIXED.tif
# Converting colorspace to sRGB
convert cmyk.tif -profile /icc/sRGB_v4_ICC_preference.icc new_rgb_file.tif
# Convert 16 bit image to 8 bit
convert file_16_bit.tif -depth 8 file_8_bit.tif
```

More example convertions

```bash
# Example convertions
convert JRL999999999.tif JRL999999999.jpg
convert JRL999999999.tif JRL999999999.jp2
convert JRL999999999.tif -quality 0 wizard.jp2
magick 'wizard.jp2[640x480+0+0]' wizard.png
```

## Mogrify vs Convert

It’s worth pointing out that Imagemagick has two main commands for modifying images. The main difference is that ‘convert‘ tends to be for working on individual images, whereas ‘mogrify‘ is for batch processing multiple files.

Another key distinction is that convert is designed to modify an image and output to a separate file. Mogrify on the other hand will quite happily change the original file, unless you specify a separate output location.

## Moving to grokImageCompression

From [wiki.harvard.edu](https://wiki.harvard.edu/confluence/display/DigitalImaging/Installing+OpenJPEG+on+Windows+10%2C+Linux%2C+and+MacOS)

> GrokImageCompression is an open-source JP2 encoder based on the OpenJPEG code that produces JP2 images encoded with the same colorspace -- includes the same ICC display profile -- resident in the source image.

Lossless example

```bash
grk_compress -TransferExifTags -i in.tif -o grk_out_lossless.jp2 -p RLCP -t 1024,1024 -EPH -SOP
```

```bash
grk_compress -TransferExifTags -i MM-47426-001.tif -o out.jp2 -r 8 -n 7 -c [256,256],[256,256],[128,128] -t 512,512 -p RPCL -b 64,64 -SOP -EPH
grk_compress -i MM-47426-001.tif -o out.jp2 -r 8 -n 7 -c [256,256],[256,256],[128,128] -t 512,512 -p RPCL -b 64,64 -SOP -EPH
```

## Creating Tiles for Zooming images

[OpenSeaDragon](https://openseadragon.github.io/) is an open-source, web-based viewer for high-resolution zoomable images, implemented in pure JavaScript, for desktop and mobile. [deepzoom.py](https://github.com/openzoom/deepzoom.py) has been added to the image to support [Creating Zooming Images](https://openseadragon.github.io/examples/creating-zooming-images/).

```python
#!/usr/bin/env python3
import os
import deepzoom

# Specify your source image
SOURCE = "sample.tiff"

# Create Deep Zoom Image creator with weird parameters
creator = deepzoom.ImageCreator(
    tile_size=128,
    tile_overlap=2,
    tile_format="jpg",
    image_quality=1.0,
    resize_filter="bicubic",
)

# Create Deep Zoom image pyramid from source
creator.create(SOURCE, "output/sample.dzi")
```

If deepzoom has a problem with the file having an alpha channel `cannot write mode RGBA as JPEG`, you can remove it with the following command:

```sh
convert sample.tiff -alpha off sample_no_alpha.tiff
```

Change the SOURCE in `tile-test.sh` to the new file `sample_no_alpha.tiff` and run the script to create the tiles in the output directory.

## Misc links

- [Imagemagick tuning options to convert images to JP2](http://www.imagemagick.org/discourse-server/viewtopic.php?t=27300)
- [Imagemagick Batch Convert](https://linuxhint.com/imagemagick-batch-convert)
- [ImageMagick JPEG-2000 Image Format](http://patrickdieudonne.com/ImageMagick-6.7.0-2/www/jp2.html)
- [Embedding derived JPEG2000 image files with image technical metadata transferred from source images](https://wiki.harvard.edu/confluence/display/DigitalImaging/Embedding+derived+JPEG2000+image+files+with+image+technical+metadata+transferred+from+source+images)
