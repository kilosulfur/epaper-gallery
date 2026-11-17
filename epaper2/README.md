# EPaper2 图库（方案 G：GitHub + jsDelivr）

本目录是 EPaper2 电子墨水屏天气站的**公开图床源**，对应《需求规格与架构设计》§7.3 方案 G。
仓库必须保持 **public**（jsDelivr 的 `/gh/` 只索引公开仓库）。

## 文件

| 文件 | 用途 |
|---|---|
| `index.txt` | 同步探针，单行 `EP2IDX1 <version> <count> <crc32_hex> <bytes> <filename>` |
| `bundle_0001.bin` | 64B 头 + 4 × 1696B 1bit 图片（128×106，MSB-first）= 6848B；**测试版** |
| `manifest_0001.tsv` | PC 侧逐图 CRC 表，设备**不下载** |
| `preview/0000..0003.png` | 4 张测试图的 4x 放大预览，仅供人工查看 |

生成方式（本地 `tools/`）：

```bash
python tools/img_convert.py src/ -o out/raw --mode dither --fit cover --preview
python tools/img_bundle.py publish -i out/raw -o publish --version 1
```

## 设备侧 URL

```
index  主： https://raw.githubusercontent.com/kilosulfur/epaper-gallery/main/epaper2/index.txt
index  备： https://cdn.jsdelivr.net/gh/kilosulfur/epaper-gallery@main/epaper2/index.txt
bundle 主： https://cdn.jsdelivr.net/gh/kilosulfur/epaper-gallery@main/epaper2
bundle 备： https://raw.githubusercontent.com/kilosulfur/epaper-gallery/main/epaper2
```

- `index.txt` 用 raw.githubusercontent.com 作主站：无 CDN 缓存，更新后立即生效；
  固件还会追加 `?t=<millis()>` 破缓存，jsDelivr 作备用。
- bundle 用 jsDelivr 作主站，**文件名随版本递增**（`bundle_0002.bin` …），
  因此新版本一定是全新 URL，CDN 不可能返回旧字节；旧文件保留以便回滚。
- 固件用 `setInsecure()`（TLS 策略 A），不维护根证书；内容完整性由
  包头 magic + 精确长度 + CRC32 保证。

## 更新规则

1. 每次更新**必须**递增 `--version`，产出新的 `bundle_XXXX.bin`；
2. 保留上一版 bundle 至少一次发布周期；
3. `index.txt` 的 `crc32` 覆盖 64B 头之后的 payload，`bytes` 为整包字节数；
4. 图片张数不固定（设计容量 256），设备从包头读 `count`，版本变化后 clamp 轮播索引；
5. 不要把 WiFi 凭据或任何私密内容放进本仓库——它是公开的。
