# 实现细节

## SVG预处理
iconfount的配置文件主要包括字体的基本信息，拿到这些信息后，首先做的是SVG文件的预处理。由于TTF文件支持
的特性相比SVG要少很多，所以SVG中很多属性和tag都会被剔除，并且所有图形都会被转换成一个复合路径。
内部最终会得到一个类似这样的数据来描述一个SVG文件：

`path`里面包含了这个SVG中最重要的路径描述，也就是SVG所画的图形；`code`是这个SVG图标最终在字体文件
中对应的字符codepoint，css中引用这个图标就是通过这个数字引用的。

```json
{
  "src": "custom.svg",
  "code": 59392,
  "css": "custom",
  "keywords": [
    "custom"
  ],
  "width": 1000,
  "d": "M36-79l35-35 0 43c0 44 36 67 150 107 2 1 2 1 3 1 144 50 195 84 195 171 0 2 0 2 0 5 0 6 0 6 0 12 1 13 1 20 1 28l-1 15-10 10c-30 30-57 70-69 116l-4 17-16 7c-8 3-16 12-22 26-9 23-8 50 5 64l11 12-2 16c-1 5-1 5-2 11-4 39-3 72 5 104 7 26 19 49 37 66 37 38 89 60 150 62 59-2 110-24 147-62 18-17 30-39 37-65 9-32 10-65 5-103 0-6 0-6-1-12l-2-15 10-12c13-15 15-43 5-66-5-14-13-23-21-26l-17-7-4-17c-12-46-38-85-69-116l-10-10-1-15c0-27 0-35 0-47 0-84 51-119 191-170 3-1 3-1 6-2 114-42 151-67 151-113l0-35 35 35-928 0z m964-71l0 36 0 35c0 90-54 127-197 180-3 1-3 1-6 2-110 40-145 63-145 103 0 12 1 20 1 46l-36 1 25-26c39 39 73 89 88 149l-34 9 13-33c27 11 48 34 60 65 19 46 16 101-16 139l8-27c1 5 1 5 1 12 5 47 5 88-7 130-10 37-28 70-55 97-50 50-119 80-198 82-82-2-150-32-200-82-28-27-46-61-56-99-11-42-12-83-7-129 1-7 1-7 2-13l9 28c-33-36-37-92-18-138 13-31 33-54 60-65l14 33-35-9c16-60 49-110 88-149l25 26-36-1c0-7 0-13 0-25 0-7 0-7 0-14 0-2 0-2 0-5 0-41-35-64-147-103-2-1-2-1-3-1-145-51-198-86-198-175l0-43 0-36 36 0 928 0 36 0z",
  "segments": 86
}
```

这步预处理只保留了SVG文件中图形的轮廓，颜色之类的信息都会被丢掉，因为这些信息无法转换到TTF中。同时
SVG的路径会被缩放到一个标准尺寸，最终生成TTF时，每个字符的画布尺寸都是一样的，这个过程用了`svgpath`
来处理各种路径的变换。

## 生成字体

SVG预处理完成后，就开始生成字体文件了，首先生成的是SVG字体文件，SVG字体其实就是一个用SVG文件描述的
字体文件，格式就是一个SVG文件，所以我们通过一个模版`templates/font/svg.tpl`来生成这个SVG字体文件。

有了这个SVG文件后，iconfount就通过`svg2ttf`从SVG字体文件生成等价的TTF字体文件，得到TTF文件后，我们就
从TTF生成其他类型的字体文件，包括eot, woff, woff2，这些文件本质上都是对TTF文件的一个封装，添加了一些
元数据和压缩特性，使得这些字体文件更适合在网络环境下使用。

这一步的时候同时会生成`config.json`和`codes.json`，`config.json`主要是调试使用的，里面包含了
最终用来生成字体文件的所有信息；`codes.json`包含了每个字符的详细信息。

## 生成CSS和demo

字体文件生成完成后，就开始生成CSS文件和demo文件，这些都是通过`templates`下的模版文件生成的。
