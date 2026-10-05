用于下载抖音视频的 Codex Skill。

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

## 已知限制

- 依赖第三方解析站，页面、按钮和上游服务可能变化。DLPanda 失败时可按 skill 尝试 [SnapAny](https://snapany.com/)，该备用服务曾对同一抖音链接解析失败。
- 原有 E2B 环境曾遇到反爬、接口空响应和网络错误，这些记录不能推出所有环境中的 HTTP 请求都不可用。
- 预览可播放或页面显示 `DL_SUCCESS` 只证明解析结果可用，仍须确认文件实际保存并验证。
- 无水印效果取决于解析站返回的源文件，本仓库不保证所有视频都无水印。
- 登录、人机验证或不可访问的视频可能需要用户操作；skill 不绕过这些限制。

## 许可证

[MIT](LICENSE)。许可证适用于本仓库的 skill 和文档，不授予第三方视频的版权；下载和使用视频需遵守平台规则及作者授权。
