# B站视频总结助手（bili-summary）

把 B站视频自动转化为**结构化总结报告**的桌面小工具（Windows）。

输入一个视频链接，自动完成：抓取字幕（CC 优先、AI 兜底）→ DeepSeek 大模型提炼 → 输出 Markdown 总结报告。

## 功能特性

- **结构化总结报告**：一句话总结 / 核心观点（带时间戳）/ 章节脉络 / 金句摘录 / 博主风格与立场 / 可执行结论
- **全文逐字稿**：带时间戳的完整字幕文本，免费导出
- **批量总结**：一次跑多个视频，失败不中断
- **多分P支持**：多P视频合并总结，`?p=N` 单P处理
- **博主观点跨视频跟踪**：观点自动入库，跨期生成"观点演变报告"（可勾选视频排除老视频）
- **两种形态**：图形界面（GUI）+ 命令行（CLI）
- **断点续传缓存**：重复总结秒出结果、省钱

## 快速开始

### GUI 版（推荐日常使用）

1. 从 [Releases](../../releases) 下载 `bili-sum_share.zip`（纯净分享版，不含任何个人数据）
2. 解压 → 双击 `bili-sum.exe`（启动约 1 秒）
3. 设置页粘贴你的 **DeepSeek API Key** 与 **B站完整 Cookie**
4. 首页粘贴视频链接 → 开始总结

### CLI 版

```bash
pip install openai python-dotenv requests
python scripts/get_sessdata.py          # 自动提取 Edge 中的 B站 Cookie
# 在 .env 写入 DEEPSEEK_API_KEY=sk-xxx

python main.py "https://www.bilibili.com/video/BV1uiuU6LEYF"
python main.py --batch links.txt        # 批量
python main.py --track 莫大韭菜         # 博主观点跟踪
```

## 文档

- [GUI 使用手册](docs/GUI使用手册.md)
- [CLI 使用手册](docs/CLI使用手册.md)

## 目录结构

```
main.py          CLI 入口
core.py          核心业务（CLI/GUI 共用）
gui/             tkinter 图形界面（五页）
bilibili/        B站 API（元信息/wbi签名/字幕）
llm/             DeepSeek 分块总结（Map-Reduce）
tracker/         博主观点跟踪（SQLite）
scripts/         工具脚本（Cookie提取/分享版生成/图标生成）
tests/           pytest 测试（29 离线 + 8 网络）
```

## 免责声明

- 本工具**仅供个人学习研究使用**，请勿商用或传播抓取内容
- 视频字幕内容版权归 B站及原作者所有
- 使用本工具产生的一切后果由使用者自行承担

## License

MIT License
