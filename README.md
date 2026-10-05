# video-parse-download

用于 Codex 的抖音视频解析与下载 skill。通过浏览器核对视频信息，操作第三方解析站取得媒体直链，保存文件并验证下载结果。

本仓库提供 `SKILL.md` 操作规范和实测记录，不包含独立的抖音解析引擎，也不自动安装浏览器工具。

## 支持的链接

- 视频详情页：`https://www.douyin.com/video/7659740091799241957`
- 分享短链：`https://v.douyin.com/.../`
- 带 `modal_id` 的页面：`https://www.douyin.com/user/self?modal_id=7659740091799241957`

个人主页或收藏页必须指定一个视频 ID；批量下载不在本 skill 的范围内。其他平台尚未验证。

## 环境要求

- Codex 或支持 `SKILL.md` 的 Agent。
- 可操作的浏览器及对应自动化工具，例如 Tabbit CLI，或连接到浏览器的 Playwright / agent-browser。工具须已安装并可连接；需要登录或人机验证时由用户完成。
- 浏览器可以访问抖音、[DLPanda](https://dlpanda.com/douyin) 和解析出的媒体 CDN。
- 一个可写的下载目录。可选安装 FFmpeg / ffprobe，用于核对时长和完整解码验证。

## 安装到 Codex

在 Codex 中直接发送：

```text
使用 skill-installer 安装 https://github.com/lailoo/video-parse-download/tree/main/skills/video-parse-download
```

也可以使用 Codex 自带的安装脚本，需要 Python 3 和网络连接：

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo lailoo/video-parse-download \
  --path skills/video-parse-download
```

默认安装到 `${CODEX_HOME:-$HOME/.codex}/skills/video-parse-download`。安装器遇到同名目录会停止；更新前请先备份并移走旧目录，再重新安装。安装完成后，下一轮对话即可使用。

其他支持 skill 的 Agent 可以将 `skills/video-parse-download/` 整个目录放入其技能目录；浏览器操作方式取决于对应 Agent 的工具。

## 使用

将视频链接交给 Codex，并说明保存位置：

```text
使用 $video-parse-download 下载这条抖音视频：
https://www.douyin.com/video/7659740091799241957
保存到当前项目目录，下载后检查文件是否可播放。
```

预期报告包含标题、作者、视频 ID、媒体类型、文件路径、字节数和验证结果。解析成功但文件没有保存时，不应报告为下载成功。

## 工作原理

```text
抖音链接 -> 视频 ID -> 抖音详情页核对 -> DLPanda 解析
        -> 有时效的 CDN 媒体直链 -> 保存 MP4 -> 验证
```

skill 负责操作流程和结果验证，DLPanda 负责解析视频地址。仓库没有实现抖音接口签名算法，也无法说明第三方服务的内部解析机制。

优先使用解析站的 `Download Video`。部分浏览器会直接打开视频预览，此时可以通过浏览器工具的文件获取能力，使用已展示的媒体地址保存文件。Tabbit CLI 的 `page.fetch` 支持在浏览器网络环境下保存文件，不依赖 Playwright 下载事件。

CDN 地址可能过期，应在解析成功后尽快下载；不要将带签名的临时地址作为长期下载链接发布。

## 实测结果

2026-10-05，通过 Tabbit 浏览器和 DLPanda 完成一次实际下载：

| 项目 | 结果 |
| --- | --- |
| 视频 | Stars fell here #梦核 #怀旧 #艺术 #dreamcore #liminalspace |
| 作者 | Ozyman |
| 视频 ID | `7659740091799241957` |
| 媒体类型 | VIDEO + AUDIO |
| 文件名 | `[DLPanda.com][Ozyman]7659740091799241957.mp4` |
| 文件大小 | 1,645,834 字节 |
| 时长 | 15.487007 秒 |
| 分辨率 | 1080 x 1920 |
| 验证 | 浏览器文件请求 HTTP 200、ffprobe 识别 MP4、FFmpeg 完整解码无报错 |

视频文件不包含在本仓库中。更多过程和历史失败记录见 [probe-log.md](skills/video-parse-download/references/probe-log.md)。单个成功案例不能保证所有链接都可下载。

## 已知限制

- 依赖第三方解析站，页面、按钮和上游服务可能变化。DLPanda 失败时可按 skill 尝试 [SnapAny](https://snapany.com)，该备用服务曾对同一抖音链接解析失败。
- 原有 E2B 环境曾遇到反爬、接口空响应和网络错误，这些记录不能推出所有环境中的 HTTP 请求都不可用。
- 预览可播放或页面显示 `DL_SUCCESS` 只证明解析结果可用，仍须确认文件实际保存并验证。
- 无水印效果取决于解析站返回的源文件，本仓库不保证所有视频都无水印。
- 登录、人机验证或不可访问的视频可能需要用户操作；skill 不绕过这些限制。

## 仓库结构

```text
.
|-- README.md
|-- LICENSE
|-- .gitignore
`-- skills/
    `-- video-parse-download/
        |-- SKILL.md
        `-- references/
            `-- probe-log.md
```

## 许可证

[MIT](LICENSE)。许可证适用于本仓库的 skill 和文档，不授予第三方视频的版权；下载和使用视频需遵守平台规则及作者授权。
