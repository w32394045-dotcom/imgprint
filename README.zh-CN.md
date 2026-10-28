# imgprint —— 浏览器里的图片打印器

单文件、离线、纯本地的多图打印排版工具。拖入几张图，选好纸张，直接打印。
**不上传、不注册、不留历史、不联网**。

- **主站**：[imgprint.xszx.cc.cd](https://imgprint.xszx.cc.cd/)
- **备用**：[imgprint.w32394045.workers.dev](https://imgprint.w32394045.workers.dev/)
- **GitHub Pages**：[w32394045-dotcom.github.io/imgprint](https://w32394045-dotcom.github.io/imgprint/)
- **源码**：本仓库，一个文件 [`index.html`](index.html)，约 89 KB

---

## 为什么又做一个？

用浏览器打图片这件事，通常以浪费纸收场：图被裁错、1200×1600 的手机图铺满 A4 只有 146 dpi、
满版打印却莫名其妙多出一圈白边。imgprint 只做一件事——**按预览的样子把 N 张图排到纸上**，
其余的一律不做：不要账号、不要相册、不要云同步、不要「AI 增强」。

整个应用就是一个 HTML 文件，本地双击、U 盘、自有静态托管都能跑。

## 功能

| | |
|---|---|
| **导入** | 点击、拖到页面任意位置、`Ctrl+V` 粘贴截图；支持多选，最多 60 张 |
| **排序** | 每张图可上移 / 下移 / 移除；列表顺序就是打印顺序 |
| **排版** | A4 / A5 / B5 / Letter / 6 寸照片；纵向或横向；边距 无 / 5 mm / 12 mm；每页 1、2、4 张；完整显示（留白）或铺满裁剪 |
| **预览** | 预览就是打印用的那棵 DOM，只做等比缩放，没有第二套渲染 |
| **DPI** | 每张图实时显示当前纸张/排版下的估算分辨率（≥300 绿、≥150 灰、<150 琥珀） |
| **色彩检测** | 逐张识别 sRGB / 无配置 / AdobeRGB / Display P3 / ProPhoto / CMYK / HDR，打印前给提示 |
| **水印** | 每页页脚的项目网址，默认开启，可在设置里关闭 |
| **7 种语言** | 简体中文 · 繁體中文 · 日本語 · 한국어 · Deutsch · Français · English，自动识别并记住 |
| **离线** | 不引字体、不引 CDN、无统计；内嵌严格 CSP（`default-src 'none'`） |
| **不保存** | 图片不落盘、不上传；只有「语言 + 排版」这类设置存 `localStorage`，关掉标签页图片就没了 |

## 页脚水印

设置 → **页脚推广信息**（默认开）。每张打印出来的页面底部是：

```
imgprint.xszx.cc.cd · github.com/w32394045-dotcom/imgprint
```

这两个地址是**故意印成文字的**：纸上的链接点不了，能传播的只有地址本身。屏幕上同一行文字是真链接。
需要干净页面时在设置里关掉即可。想改成自己的项目，改 app 脚本顶部的 `SITE_TEXT` / `REPO_TEXT`。

## 色彩：这个工具做什么、不做什么

浏览器在**打印环节拿不到色彩管理**：`window.print()` 无法指定打印机 ICC、渲染意图，也做不了软打样。
所以 imgprint 走的是老实路线（策略 A）：

- 按浏览器给出的像素、**以 sRGB 直接打印，不做任何转换**；
- 同时**解析图片元数据**（JPEG APP2/APP1、PNG iCCP/cICP、WebP ICCP、AVIF/HEIC 的 `colr`），
  把每张图的色彩空间标出来：

| 徽标 | 含义 | 风险 |
|---|---|---|
| `sRGB` | 带 sRGB 配置 / EXIF ColorSpace=1 | 无 |
| `无 ICC · 按 sRGB` | 没找到配置 | 通常没事，但文件本身没有保证 |
| `AdobeRGB`、`Display P3`、`ProPhoto` | 宽色域配置 | 打印一定偏色，先转 sRGB |
| `CMYK` | CMYK JPEG | 浏览器只是粗略转换，大概率有偏差 |
| `HDR` | PQ/HLG 传输曲线（cICP / `colr`） | 打印是 SDR，会变暗、变灰 |
| `ICC 配置` | 有配置但没解析出来 | 未知，按不可信处理 |

这个工具**不做**：色彩空间转换、读取打印机 profile、保证纸面等于屏幕。
要颜色准，先用图像软件把图转成 sRGB 再打。这是刻意的边界，界面上写清楚，不藏着。

## 打印清单

1. 设置里选纸张、方向、边距、每页张数、填充方式。
2. 点「打印」或按 `Ctrl+P`。
3. 打印对话框里：
   - **边距选「无」**（否则浏览器会在页面里再套一圈边距）；
   - **缩放保持 100%**（任何缩放都会破坏按毫米算好的版）；
   - **勾上「背景图形」**，否则满版图可能留白。
4. 先看预览页数和 DPI 徽标：150 dpi 是能用下限，300 dpi 才舒服。

## 分辨率参考

| 纸张 | 满版 300 dpi 需要 |
|---|---|
| A4 | 2480 × 3508 |
| A5 | 1748 × 2480 |
| B5 | 2079 × 2953 |
| Letter | 2550 × 3300 |
| 6 寸照片（102 × 152 mm） | 1205 × 1795 |

4032 × 3024 的手机图打 A4 很宽裕；1200 × 1600 铺满 A4 只有约 146 dpi，徽标会直接标出来。

## 实现方式

- 版式全部用毫米计算，`@page { size: <宽>mm <高>mm; margin: 0 }` 由设置动态生成，打印出来的页就是纸张大小。
- 预览和打印共用同一棵 DOM 和同一份 CSS；打印样式只做两件事：隐藏应用外壳、取消缩放。
- 图片用 `object-fit` 摆放——不经过 canvas 重采样、不重新编码，配置不会丢。
- 设置了 `print-color-adjust: exact`，水印和图片背景不会被打印流程剥掉。

## 国际化

所有界面文案外置在 `I18N` 词表里：7 种语言 × 81 条 = 567 条。翻译按**意译**处理，
按目标语言的产品文案语感重写，不做逐字对照；key 是语义化的（`set.watermark`、`badge.dpiLow`），
占位符是位置式的（`{0}`），构建时会校验 7 份词表的 key 与占位符完全一致，并检查 en/de/fr 没有中文残留、
ja 有假名、ko 有韩文。

新增语言：在 `index.html` 里复制一份词表（`var PACKS = {...}`），把语言码加进 `LANGS`、
母语名加进 `NATIVE`，自动识别规则按需补一条即可。

## 隐私与安全

- 无任何网络请求（发布闸门里有断言）。
- 不用 `innerHTML` / `eval`，文件名一律 `textContent` 写入。
- 内嵌 CSP：`default-src 'none'; img-src 'self' blob: data:;` 等。
- 图片的 object URL 在移除和卸载时回收；不持久化任何图片数据。

## 验证

开发期用 Node 闸门跑过（工具不入库，仓库只留单文件应用）：

```
gate.js         PASS 122 · FAIL 0
dom-smoke.js    PASS  28 · FAIL 0
```

- 用 Pillow 造了 19 个真实图片夹具：sRGB/AdobeRGB/ProPhoto/P3 的 ICC（`desc` 与 v4 `mluc` 两种标签）、
  EXIF ColorSpace=1/2、CMYK JPEG、PNG iCCP、PNG cICP（HDR 与 P3）、WebP ICCP、GIF、AVIF/HEIC 的 `colr`、
  垃圾字节——全部分类正确。
- DOM 冒烟测试用最小 DOM 桩**真的执行 app.js**：导入真实夹具、渲染 3 页、检查水印、切到日文、
  开关水印，并通过真实的设置项 `change` 事件重新注入 `@page` 规则。
- 版面 / DPI / 分页 / `@page` 全部对手算毫米值断言。
- i18n：7 × 81 key 一致、占位符一致、en/de/fr 无中文、ja 有假名、ko 有韩文。
- 卫生检查：单文件、无外链脚本/样式/远程 URL、无网络 API、无 HTML 注入 API、`localStorage` 仅 2 处、CSP 在位。

**机器测不到的**：真实浏览器里的观感、打印对话框的实际行为（边距/背景图形要用户自己设）、
真实打印机的颜色。版面数学和 DOM 接线都验过了，外观需要人眼过一遍再信。

## 浏览器支持

较新的 Chrome / Edge / Safari / Firefox。AVIF 都支持；HEIC 只有 Safari 能解（Chrome 会提示「打不开」，
先转 JPEG/PNG）。PNG 的 iCCP 需要 `DecompressionStream`，不支持时降级为「ICC 配置」徽标。

## 许可

MIT，见 [LICENSE](LICENSE)。
