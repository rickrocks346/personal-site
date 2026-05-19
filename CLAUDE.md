# personal-site — 松鼠的自我介绍

Scrollytelling 个人作品集网站。用户滚动页面时，视频进度跟随滚动变化，
文字卡片在特定时间段弹出。

## 文件结构

```
personal-site/
├── index.html              # 唯一页面，包含所有 HTML/CSS/JS
├── assets/
│   ├── intro-loop.mp4      # 片头循环视频（5s, 4.2MB）
│   ├── main.mp4            # 主体视频（~15s, 13MB, 1+2+3.mp4 合并）
│   ├── outro-loop.mp4      # 片尾循环视频（5s, 4.2MB）
│   └── qrcode.webp         # 抖音二维码（300x300, 12KB）
└── CLAUDE.md               # 本文件
```

## 技术架构

- 纯静态 HTML 单文件，无框架，无构建工具
- CSS scroll-snap 实现滚动吸附（y mandatory + snap-stop: always）
- IntersectionObserver 检测当前可见的 snap-section，触发文字卡片切换
- requestAnimationFrame tick loop 监控视频播放进度，到达 timeEnd 时暂停
- 3 层视频叠加（z-index: 0），通过 opacity 淡入淡出切换
- 文字卡片固定在屏幕底部（z-index: 2），pointer-events: none 不阻挡交互

## 视频来源

| 文件 | 来源 |
|------|------|
| intro-loop.mp4 | D:\Pictures\首-循环.mp4 |
| main.mp4 | D:\Pictures\1.mp4 + 2.mp4 + 3.mp4 用 ffmpeg concat 合并 |
| outro-loop.mp4 | D:\Pictures\尾-循环.mp4 |

合并命令：
```bash
ffmpeg -f concat -safe 0 -i files.txt -c copy main.mp4
```

## 核心数据结构

### segments 数组

6 个 segment，每个对应一个 snap-section：

```
seg-0: intro 视频（片头循环）
seg-1: main 视频 第 1/4 段 — "他做了些有意思的东西"
seg-2: main 视频 第 2/4 段 — "Claude Code 入门指南"
seg-3: main 视频 第 3/4 段 — "Claude for CN Finance"
seg-4: main 视频 第 4/4 段 — "Token Estimator"
seg-5: outro 视频（片尾循环） — 抖音 + 结尾（并排卡片）
```

main 视频在 `init()` 中按特定时间戳分为 4 段（0→1.5s, 1.5→5s, 5→10.4s, 10.4→end），确保每一帧都被使用。

### 如何新增项目卡

编辑 `index.html`，在 `#text-overlay` 中新增一个 `text-segment` div，
然后在 JS 的 `segments` 数组中添加对应条目，并在 `#spacer` 中新增一个 `snap-section` div。
注意：新增 main 视频段时需要调整 `init()` 中的分段逻辑。

## 部署

上传 `personal-site/` 整个文件夹到静态托管服务。

**腾讯云 COS 静态网站托管：**
1. 创建 COS 存储桶，开启"静态网站"功能
2. 上传 index.html 到根目录，assets/ 保持子目录结构
3. 设置默认文档：`index.html`
4. 配置自定义域名 + CDN + HTTPS 证书
5. 如有域名，替换 HTML 中 `[YOUR_DOMAIN]` 为实际域名

所有资源使用相对路径，无需额外构建步骤。

## 字号 / 颜色参考

- 主标题：Noto Serif SC 900, clamp(1.5rem, 3vw, 2.4rem), #1a1a1a
- 副标题：Noto Serif SC 400, clamp(0.95rem, 1.8vw, 1.25rem), #606060
- 正文：Noto Sans SC 300, clamp(0.9rem, 1.6vw, 1.1rem), #333
- 强调色：Noto Sans SC 700, #1a1a1a（正文 strong 标签）
- 标签/tag：Noto Sans SC 500, #F5A623 边框+文字（圆角药丸）
- 链接：Noto Sans SC 500, #F5A623, hover 下划线动画
- 卡片背景：rgba(250, 250, 250, 0.92) + backdrop-filter: blur(12px)
