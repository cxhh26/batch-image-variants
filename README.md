# batch-image-variants

批量图片变体 Skill：根据一张原图生成带数字编号水印的 JPG 图片，支持指定 EXIF 日期时间、生成 MD5 校验清单，并导出按批次编号的文件夹与 ZIP。

项目同时提供 Python 命令行工具和手机浏览器应用，可用于图片编号、测试素材制作和批量文件整理。

> 本工具不是 AI 绘图工具，不会根据文字生成图片。
> 不同 MD5 不代表视觉内容不同，也不保证平台将图片判定为新内容。

## 一、主要功能

- **批量生成**：每批支持生成 1–500 张图片。
- **数字水印**：从 `000001` 开始编号，支持四角位置选择。
- **日期设置**：将指定日期时间写入输出 JPG 的 EXIF 元数据。
- **MD5 校验**：根据最终图片文件计算 MD5，并检查同一批次内不重复。
- **自动分批**：输出到 `test_0001`、`test_0002` 等目录，不覆盖已有批次。
- **ZIP 导出**：便于备份、下载和传输。
- **尺寸控制**：支持长边限制为 2560、2048，或选择原始尺寸模式；不会放大原图。
- **原图保护**：不修改输入图片。
- **本地处理**：命令行在电脑本地处理，手机网页在浏览器本地处理。

## 二、随包内容

完整压缩包为 `batch-image-variants.zip`，解压后主要结构如下：

```text
batch-image-variants/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── scripts/
│   ├── generate_images.py
│   ├── serve_mobile.py
│   └── requirements.txt
├── references/
│   ├── cli.md
│   └── mobile.md
├── assets/
│   └── mobile-app/
│       ├── dist/
│       ├── src/
│       ├── public/
│       ├── licenses/
│       ├── tests/
│       ├── package.json
│       └── package-lock.json
└── tests/
```

- `SKILL.md`：面向支持 Skill 的 AI 助手的操作说明。
- `scripts/`：图片生成脚本与手机网页静态服务。
- `references/`：详细参数、输出规则和手机使用说明。
- `assets/mobile-app/`：手机网页源码、依赖锁文件、图标和构建产物。
- `tests/`：Python 自动测试；手机网页测试位于其源项目中。

如果仓库中只上传了 ZIP，需要先下载并解压，再运行下面的命令。

## 三、电脑命令行使用

### 环境要求

- Python 3.10 或更高版本。
- Pillow，依赖声明见 `scripts/requirements.txt`。
- 输入支持静态 JPG、PNG、WebP；HEIC 请先转换。

### 安装依赖

进入解压后的 `batch-image-variants` 目录，在终端运行：

```bash
python3 -m venv .venv
```

macOS / Linux：

```bash
source .venv/bin/activate
```

Windows PowerShell：

```powershell
.\.venv\Scripts\Activate.ps1
```

激活环境后安装依赖：

```bash
python -m pip install -r scripts/requirements.txt
```

### 生成示例

把原图放到当前目录并命名为 `input.jpg`，执行：

```bash
python scripts/generate_images.py --image "./input.jpg" --count 10 --time "2024-01-01T00:00:00" --timezone "Asia/Shanghai" --output-root "./generated" --position bottom-right --size 2560 --zip
```

这个示例会：

1. 从 `input.jpg` 生成 10 张 JPG 图片。
2. 在右下角添加数字编号水印。
3. 将 EXIF 日期设为北京时间 2024 年 1 月 1 日零点。
4. 将输出图片长边限制在 2560 像素以内，不放大原图。
5. 生成图片文件夹、MD5 清单、批次信息和 ZIP。

这里的日期是图片元数据，不是预约运行时间。

### 常用参数

| 参数 | 说明 |
| --- | --- |
| `--image` | 必填，原图文件路径 |
| `--count` | 必填，生成数量，范围 1–500 |
| `--time` | 指定日期时间；省略时使用执行时的当前时间 |
| `--timezone` | 日期未包含明确偏移时使用的时区，默认 `Asia/Shanghai` |
| `--output-root` | 必填，生成文件的根目录 |
| `--position` | 水印位置，默认 `bottom-right` |
| `--size` | `2560`、`2048` 或 `original` |
| `--zip` | 同时生成 ZIP 压缩包 |

水印位置还支持 `bottom-left`、`top-right`、`top-left`。

更多参数规则和输入限制见 `references/cli.md`。

## 四、输出文件

```text
generated/
├── test_0001/
│   ├── 000001.jpg
│   ├── 000002.jpg
│   ├── ...
│   ├── 000010.jpg
│   ├── md5.csv
│   └── manifest.json
└── test_0001.zip
```

- `md5.csv`：图片编号与 MD5 校验清单。
- `manifest.json`：原图信息、生成数量、尺寸、日期时间，以及每张输出图片的 MD5 和文件大小。
- 后续批次继续使用新的目录编号，不覆盖已有批次。

## 五、手机浏览器使用

随包提供适合手机访问的网页应用，不是 APK、IPA 或鸿蒙原生安装包。

### 局域网访问

在电脑上进入解压后的 Skill 目录，运行：

```bash
python3 scripts/serve_mobile.py --host 0.0.0.0 --port 8765
```

然后：

1. 将手机和电脑连接到同一个可信局域网。
2. 查询电脑当前的局域网 IP。
3. 在手机浏览器中访问 `http://电脑IP:8765`。
4. 在网页中选择原图、数量、日期等参数。
5. 生成并下载 ZIP，再通过手机文件管理器解压。

此模式下电脑必须保持服务运行。静态服务只分发网页，不接收图片上传；图片处理在手机浏览器内完成。

`assets/mobile-app/dist/` 已包含构建好的静态网页，直接使用不需要安装 Node.js。

### 使用限制

- 安卓、iOS、鸿蒙的实际体验取决于目标设备和浏览器。
- HEIC 是否可读取取决于浏览器，不支持时请先转为 JPG。
- 局域网普通 HTTP 访问不保证支持离线缓存、系统分享等能力。
- HTTPS 静态部署方式和其他说明见 `references/mobile.md`。

## 六、作为 Skill 使用

本项目既包含供 AI 助手读取的 `SKILL.md`，也包含实际执行所需的脚本和网页资源。

在支持此类 Skill 的助手中，按其安装方式放置完整目录后，可以这样提出需求：

> 使用 batch-image-variants，读取我提供的原图，生成 10 张带右下角数字水印的图片，将日期设为北京时间 2024 年 1 月 1 日零点，并导出 ZIP。

需要提供真实可读取的原图，并明确数量、日期和输出位置。

仅上传 GitHub 或下载压缩包，不等于已经在某个 AI 助手中安装或启用了 Skill。

## 七、注意事项

### MD5 不代表内容原创

MD5 根据最终文件字节计算，仅用于文件差异标识，不用于安全认证。

本工具检查同一批次内的 MD5 不重复，但不承诺与所有历史图片进行过全局比对，也不保证绕过平台的重复内容检测、审核或版权识别。

### 日期修改存在边界

- 支持的指定年份为 1980–2107。
- EXIF 日期只是写入的元数据，不证明图片在该日期拍摄。
- 电脑脚本会设置输出 JPG 的文件修改时间，不保证修改文件创建时间。
- ZIP 时间不包含时区，且只有 2 秒精度。
- 手机相册日期、系统文件时间和导入行为由系统及应用决定。

### 图片会重新编码

输出统一为 JPG。透明区域铺白，不复制原图旧 GPS、拍摄设备等元数据。水印、尺寸调整和重新编码可能改变画质与文件大小。

聊天转发、相册导入或其他软件再次保存后，MD5 和元数据可能变化。

### 使用范围

请仅处理自己拥有或已获得授权使用的图片，不要将输出文件用于伪造拍摄事实、侵犯版权或规避平台规则。

## 八、开发与测试

Python 测试，需要先安装 Pillow：

```bash
python -m unittest discover -s tests -v
```

手机网页开发需要符合 `assets/mobile-app/package.json` 要求的 Node.js 环境：

```bash
cd assets/mobile-app
npm ci
npm test
npm run build
npx playwright install chromium webkit
npm run test:e2e
```

自动化浏览器测试不等于安卓、iOS、鸿蒙真机兼容性认证。实际使用前，建议在目标设备上确认图片选择、下载、解压和日期显示行为。
