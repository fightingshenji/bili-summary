# 视频总结助手 · CLI 版使用手册

适用：`python main.py`（命令行）
版本：v0.4.0 ｜ 更新：2026-08-20

---

## 目录

1. [软件简介](#1-软件简介)
2. [环境准备](#2-环境准备)
3. [参数速查表](#3-参数速查表)
4. [基础用法](#4-基础用法)
5. [分P与多P处理](#5-分p与多p处理)
6. [批量总结](#6-批量总结)
7. [博主观点跟踪](#7-博主观点跟踪)
8. [输出目录结构](#8-输出目录结构)
9. [配置（.env）](#9-配置env)
10. [退出码速查](#10-退出码速查)
11. [自动化与脚本化](#11-自动化与脚本化)
12. [常见问题](#12-常见问题)

---

## 1. 软件简介

把 B站视频自动转化为结构化总结报告（核心观点/章节脉络/金句/博主风格），支持批量、多分P、博主观点跨视频跟踪。

CLI 版适合：脚本化批处理、服务器/无界面环境、与其它工具串联。

---

## 2. 环境准备

```bash
# 1. 安装依赖（仅3个）
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple openai python-dotenv requests

# 2. 准备 B站 Cookie（AI字幕必需）
python scripts/get_sessdata.py        # 自动从 Edge 提取 → bilibili_cookies.json

# 3. 准备 DeepSeek Key：项目根 .env 写入
#    DEEPSEEK_API_KEY=sk-xxxx
```

> 需要 Python 3.9+；Windows 下联网命令建议先清代理：`env -u HTTP_PROXY -u HTTPS_PROXY python main.py ...`

---

## 3. 参数速查表

```
python main.py [url] [选项]
python main.py --batch FILE [选项]
python main.py --track 博主名 [--vids BV1,BV2]
python main.py --owners
```

| 参数 | 说明 |
|---|---|
| `url` | 视频链接 / BV号 / av号 / b23.tv 短链 |
| `--batch FILE` | 批量模式：文件每行一个链接，`#` 注释，失败不中断 |
| `--part 1,2,3` | 指定分P；`0`=全部分P；默认第1P |
| `--out DIR` | 输出根目录（默认 `output/`） |
| `--no-llm` | 只取字幕拼全文，不调用大模型（免费） |
| `--no-cache` | 忽略缓存强制重跑 |
| `--no-track` | 总结时不入库观点 |
| `--track OWNER` | 生成指定博主的观点演变报告 |
| `--vids BV1,BV2` | 与 `--track` 组合：只分析指定视频 |
| `--owners` | 列出已跟踪博主 |
| `--verbose` | 输出调试日志 |
| `--version` | 版本号 |
| `-h, --help` | 帮助 |

---

## 4. 基础用法

```bash
# 单视频总结（最常用）
python main.py "https://www.bilibili.com/video/BV1uiuU6LEYF"

# 只取字幕不调大模型（免费，秒出）
python main.py "BV1ZRgX6rE3p" --no-llm

# 忽略缓存强制重跑
python main.py "BV1uiuU6LEYF" --no-cache

# 指定输出目录
python main.py "视频链接" --out /data/reports
```

**执行流程**：解析输入 → 抓元信息 → 抓字幕（缓存优先）→ 分块总结（Map-Reduce）→ 写报告 → 观点自动入库。

---

## 5. 分P与多P处理

```bash
# 只总结第 3 个分P
python main.py "https://www.bilibili.com/video/BV1nK4y1X7Tb?p=3"

# 合并总结所有分P（多P视频的完整版）
python main.py "BV1nK4y1X7Tb" --part 0

# 指定多个分P
python main.py "BV1nK4y1X7Tb" --part 1,4

# 默认只处理第 1P（不加 --part 时）
```

> 多P合并时全文按 P1→Pn 顺序拼接，报告覆盖全部内容；单P模式下输出文件名带 `P{n}` 后缀，互不覆盖。缓存按 P 集合隔离。

---

## 6. 批量总结

**批量列表文件**（每行一个链接，`#` 注释，空行忽略）：

```
# links.txt
https://www.bilibili.com/video/BV1uiuU6LEYF
BV1ZRgX6rE3p
BV1nK4y1X7Tb?p=2
```

```bash
python main.py --batch links.txt
```

- 逐个自动总结，**单个失败不中断**
- 结束打印汇总：`成功 X / 失败 Y / 共 N`
- 失败清单写入 `output/failures.csv`（链接/退出码/原因/时间）
- 有失败时退出码为 `1`，全成功为 `0`

---

## 7. 博主观点跟踪

每次总结后观点**自动入库**（SQLite `reports.db`），可跨视频追踪观点演变。

```bash
# 列出已跟踪博主
python main.py --owners

# 生成某博主的观点演变报告（全部视频）
python main.py --track 莫大韭菜

# 只分析指定视频（排除时间跨度过大的老视频）
python main.py --track 莫大韭菜 --vids BV1ZRgX6rE3p,BV1uiuU6LEYF

# 总结老视频时不想入库：
python main.py "老视频链接" --no-track
```

**报告内容**：视频时间线 + 逐期观点明细 + 主题演变分析（同主题跨期对比、变化点、转折信号）。
输出：`output/博主观点跟踪/{博主}_观点跟踪.md`

---

## 8. 输出目录结构

```
output/
├── 莫大韭菜/
│   └── 20260810_【直播回放】..._BV1uiuU6LEYF/
│       ├── 【直播回放】..._总结报告.md     ← 核心交付物
│       ├── 【直播回放】..._全文逐字稿.txt   ← 带时间戳全文
│       ├── meta.json                       ← 元信息（bvid/时长/字幕来源/覆盖时长）
│       └── cache/                          ← 字幕缓存 + 块摘要缓存（断点续传）
├── 博主观点跟踪/
│   └── 莫大韭菜_观点跟踪.md
└── failures.csv                            ← 批量失败清单（utf-8-sig，Excel可开）
```

---

## 9. 配置（.env）

项目根 `.env`（或 exe 模式 `data/.env`）：

```ini
DEEPSEEK_API_KEY=sk-xxxx          # 必填
SESSDATA=xxx                      # 可选回退（推荐用完整Cookie文件）
BUVID3=xxx                        # 可选
LLM_MODEL=deepseek-chat           # 可选
CHUNK_SIZE=5000                   # 分块字数（默认5000）
REQUEST_INTERVAL=2.0              # 请求间隔秒（默认2，防风控）
REQUEST_TIMEOUT=30                # 请求超时秒
OUTPUT_DIR=output                 # 输出目录
```

**Cookie 读取优先级**：`bilibili_cookies.json`（完整Cookie，推荐）> `.env` 的 SESSDATA。

> ⚠️ 实测：AI 字幕需要完整 Cookie（buvid3/buvid4/b_nut/bili_ticket），仅 SESSDATA 拿不到字幕。Cookie 提取用 `python scripts/get_sessdata.py`。

---

## 10. 退出码速查

| 退出码 | 含义 | 处理 |
|---|---|---|
| 0 | 成功 | — |
| 1 | 网络错误 / 批量有失败项 | 检查网络后重试 |
| 2 | 无字幕 | 该视频没有 CC/AI 字幕 |
| 3 | 配置缺失 / 参数错误 | 检查 Cookie、Key、参数 |
| 4 | 风控拦截 | 降低频率 / 更新 Cookie |
| 5 | 视频不存在 | 链接错误或已删除 |
| 6 | 大模型总结失败 | 检查 Key 余额 / 网络 |
| 130 | 用户取消 (Ctrl+C) | — |

---

## 11. 自动化与脚本化

```bash
# 每日定时批量总结（Windows 任务计划程序 / cron）
python main.py --batch links.txt --no-track >> summary.log 2>&1

# 循环跟踪多位博主
for owner in 莫大韭菜 威廉哥; do
  python main.py --track "$owner"
done

# 用 --owners + --track --vids 做增量观点报告
python main.py --owners
```

**提示**：缓存断点续传——中途中断后重跑同一命令，已完成的块不重复调用大模型。

---

## 12. 常见问题

**Q1：拿不到字幕？**
检查 Cookie 是否完整（`python scripts/get_sessdata.py` 重新提取），是否过期。

**Q2：老视频总结内容少？**
老视频 AI 字幕常不完整（meta.json 的 `subtitle_coverage` 可见覆盖到几点）。属正常，可用 `--no-track` 不入库。

**Q3：被风控（退出码4）？**
增大 `.env` 的 `REQUEST_INTERVAL`（如 3-5），或更换 Cookie。

**Q4：报"视频不存在"（-400）？**
不存在的合法格式 BV 号 B站返回 -400，映射为退出码 5。检查链接。

**Q5：能否离线运行？**
不能。需要网络访问 B站接口与 DeepSeek API。

**Q6：批量文件编码？**
支持 UTF-8（含 BOM）；Windows 记事本默认 UTF-8 即可。

---

*如有问题，随时提问。*
