lvgl的解码器非常弱，只能解码最基础的JPG和PNG图片
- 能够解码的JPG为 `Baseline JPEG + 8-bit + YCbCr + 无 EXIF + 无 ICC + 标准 Huffman`
- 能够解码的PNG为 `PNG + 8-bit + Truecolor / RGBA + 非 interlace + 无调色板 + 无扩展 chunk`

如何是自己手工使用一些图像编辑软件生成图片会很麻烦，可以使用`ImageMagick`这个工具来快速制作能够被解码的图片

- 对于PNG 如果需要透明图层
```shell
#linux
magick input.png \
  -strip \
  -depth 8 \
  -type TrueColorAlpha \
  -interlace none \
  output_lvgl.png
#windows

```

- 如果不需要透明图层
```shell
#linux
magick input.png \
  -strip \
  -depth 8 \
  -type TrueColor \
  -interlace none \
  output_lvgl.png
#windows
magick input.png -strip -depth 8 -type TrueColor -interlace none output_lvgl.png
```

- 对于JGP
```shell
#linux
magick input.jpg \
  -strip \
  -interlace none \
  -sampling-factor 4:2:0 \
  -quality 85 \
  output_lvgl.jpg
#windows
magick input.jpg -strip -interlace none -sampling-factor 4:2:0 -quality 85 output_lvgl.jpg

```

### 对于软件版本的选择

对于windows 选择 `ImageMagick-7.1.2-12-Q8-x64-dll.exe`,选择Q8版本，生成的图片会是8BIT的