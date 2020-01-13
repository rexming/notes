## 多个markdwon 文件生成 pdf (含中文)

 环境manjaro

1. 安装latex ,pandoc
2. 查找系统上的有效中文字体

```bash
[~]$ fc-list :lang=zh
/usr/share/fonts/source_han_serif/SourceHanSerifCN-ExtraLight.otf: Source Han Serif CN,思源宋体 CN,Source Han Serif CN ExtraLight,思源宋体 CN ExtraLight:style=ExtraLight,Regular
/usr/share/fonts/source_han_serif/SourceHanSerifCN-Bold.otf: Source Han Serif CN,思源宋体 CN:style=Bold
/usr/share/fonts/wqy-microhei/wqy-microhei.ttc: WenQuanYi Micro Hei Mono,文泉驛等寬微米黑,文泉驿等宽微米黑:style=Regular
/usr/share/fonts/wqy-zenhei/wqy-zenhei.ttc: WenQuanYi Zen Hei Sharp,文泉驛點陣正黑,文泉驿点阵正黑:style=Regular
/usr/share/fonts/source_han_serif/SourceHanSerifCN-SemiBold.otf: Source Han Serif CN,思源宋体 CN,Source Han Serif CN SemiBold,思源宋体 CN SemiBold:style=SemiBold,Regular
/usr/share/fonts/source_han_serif/SourceHanSerifCN-Medium.otf: Source Han Serif CN,思源宋体 CN,Source Han Serif CN Medium,思源宋体 CN Medium:style=Medium,Regular
/usr/share/fonts/wqy-microhei/wqy-microhei.ttc: WenQuanYi Micro Hei,文泉驛微米黑,文泉驿微米黑:style=Regular
/usr/share/fonts/cjkuni-uming/uming.ttc: AR PL UMing TW MBE:style=Light
/usr/share/fonts/source_han_serif/SourceHanSerifCN-Light.otf: Source Han Serif CN,思源宋体 CN,Source Han Serif CN Light,思源宋体 CN Light:style=Light,Regular
/usr/share/fonts/source_han_serif/SourceHanSerifCN-Heavy.otf: Source Han Serif CN,思源宋体 CN,Source Han Serif CN Heavy,思源宋体 CN Heavy:style=Heavy,Regular
/usr/share/fonts/wqy-zenhei/wqy-zenhei.ttc: WenQuanYi Zen Hei Mono,文泉驛等寬正黑,文泉驿等宽正黑:style=Regular
/usr/share/fonts/wqy-zenhei/wqy-zenhei.ttc: WenQuanYi Zen Hei,文泉驛正黑,文泉驿正黑:style=Regular
/usr/share/fonts/cjkuni-uming/uming.ttc: AR PL UMing TW:style=Light
/usr/share/fonts/cjkuni-uming/uming.ttc: AR PL UMing HK:style=Light
/usr/share/fonts/cjkuni-uming/uming.ttc: AR PL UMing CN:style=Light
/usr/share/fonts/source_han_serif/SourceHanSerifCN-Regular.otf: Source Han Serif CN,思源宋体 CN:style=Regular
```

方案1

使用下面pandoc指令

```bash
pandoc  src1.md src2.md -o srs.pdf --latex-engine=xelatex -V mainfont='WenQuanYi Micro Hei Mono'
```

方案2

在markdown 文档中加入如下配置

```
---
mainfont: WenQuanYi Micro Hei Mono
---
```

然后使用 `pandoc --latex-engine=xelatex test.md -o test1.pdf` 生成 PDF 文件.

方案3

你可以使用 `ctexart` 类而不需要手动指定字体 (宏包将替你执行此操作), 将以下设置添加到markdown文档中,

```
---
documentclass:
    - ctexart
---
```

生成 PDF 文档的指令和方案二是相同的. `pandoc --latex-engine=xelatex test.md -o test1.pdf`.

