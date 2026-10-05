# 抖音下载实测记录

## 2026-10-05：本地 Codex + Tabbit 浏览器

- 视频页：`https://www.douyin.com/video/7659740091799241957`
- 视频：Stars fell here #梦核 #怀旧 #艺术 #dreamcore #liminalspace。
- 作者：Ozyman。
- 抖音页面显示约 15 秒；互动数为赞 10.7 万、评论 480、收藏 1.6 万、转发 1.9 万。这些为当次页面信息，后续可能变化。

### 成功路径

1. 从输入链接的 `modal_id` 得到视频 ID，直接导航到详情页，核对标题、作者和时长。
2. 在 `https://dlpanda.com/douyin` 填入原链接并点击 Download。页面出现 `DL_SUCCESS`、`MEDIA READY` 和 `MEDIA TYPE: VIDEO + AUDIO`，标题、作者与视频 ID 一致。
3. Download Video 提供当时有效的抖音 CDN 媒体地址，建议文件名为 `[DLPanda.com][Ozyman]7659740091799241957.mp4`。临时签名地址不保存到仓库。
4. 点击 Download Video 后，浏览器打开视频直链。此时播放器 `readyState=4`，时长 `15.487007` 秒；浏览器打开预览本身不能证明文件已保存。
5. 使用 Tabbit 的 `page.fetch` 在浏览器网络环境下保存媒体文件，得到 HTTP 200、`contentType=video/mp4`、`bytes=1645834` 和实际本地路径。
6. ffprobe 验证为 MP4，包含 HEVC 视频流（1080 x 1920）和 AAC 音频流，时长 15.487007 秒，文件大小 1,645,834 字节。
7. `ffmpeg -v error -i <file> -f null -` 完整解码退出码为 0，无报错。

### 验证范围

该案例证明此次解析和本地文件保存成功，不证明所有抖音链接都受支持，也未验证所有画面是否无水印。Tabbit 不提供 Playwright 下载事件，此次以文件请求结果和本地解码作为下载完成证据，没有声称浏览器原生下载清单返回 `state=completed`。

## 原始记录：E2B 沙箱与浏览器环境

原始 skill 记录了同一视频的一次成功下载：DLPanda 返回 CDN 直链，点击 Download Video 后页面进度达到 100%，浏览器下载清单显示文件完成，大小为 1,645,834 字节。该记录的运行日期和浏览器工具版本未注明；以下保留其环境探测结果。

| 探测路径 | 原记录的结果 |
| --- | --- |
| iesdouyin 分享页 SSR，iPhone UA | `_ROUTER_DATA` 仅有壳配置，无 videoInfoRes |
| iesdouyin iteminfo API | 返回空 |
| douyin / iesdouyin 详情 API，含 aid=6383 | 403 或空体 |
| 移动端 feed API | 404 或网络不可达 |
| Baiduspider / Googlebot / Bytespider UA | 200 但无 play_addr |
| 桌面详情页直接抓取 | JS 壳，RENDER_DATA 无视频数据 |
| Beam 抓取详情页 | 504 |
| tikwm，含浏览器与代理路径 | 403、空文本或 Cloudflare 验证 |
| douyin.wtf | 404 或 SSL 超时 |
| vvhan / cccyun | DNS 失败 |
| oioweb | 证书无效 |
| qingdou | 维护页 |
| suipai | 超时 |
| SnapAny 通用解析框 | Network error |
| 抖音网页版更多菜单、分享面板与播放器 DOM | 未找到下载入口或媒体直链 |

以上是特定环境的历史结果，不是所有环境的通用限制，也不是本次逐一复测的结论。后续遇到相同失败机制时应避免无依据的重复尝试；有新的环境证据时可以重新评估。
