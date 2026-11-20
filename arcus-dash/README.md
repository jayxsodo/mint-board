# Arcus↔Bybit 干跑信号 · 可视化看板

本看板专为在手机浏览器（优先适配 iPhone Safari）中清晰、实时监控 **Arcus↔Bybit 跨所美股永续对冲机器人（干跑模式 Dry-Run）** 的运行状态、价差监控、套利信号及模拟盈亏而设计。

---

## 1. 核心原则与安全纪律

- **只读公开行情**：机器人仅读取 Arcus 与 Bybit 的公开 Orderbook/BBO 行情及资金费率。
- **纯干跑模拟（Dry-Run）**：`dry_run=true`，绝不连接真实交易所交易账户/钱包，**不下任何真单、不签名交易、不动用任何一分钱**。
- **纯静态只读**：看板前端仅通过 `fetch('signals.json')` 读取同目录下的静态数据文件，绝不直接调用任何交易所或外部交易 API。
- **零外部 CDN 依赖**：页面全部采用内联原生 HTML5、CSS 与 Vanilla JavaScript，所有折线图均采用原生 HTML5 Canvas 绘制，避免因 CDN 不稳定导致白屏或加载失败。
- **严禁编造**：所有数据均严格解析自机器人运行日志；若某标的暂无报价或遇到 404（如 Arcus 未上线该合约），均明确显示 `—` 或 `未上线`，绝不捏造假数据。

---

## 2. 访问与用法

- **在线访问**：[https://jayxsodo.github.io/mint-board/arcus-dash/](https://jayxsodo.github.io/mint-board/arcus-dash/)
- **本地直接打开**：双击或使用本地 HTTP 服务打开 `index.html`：
  ```bash
  cd ~/agy-bridge/workdir/arcus-dash/
  python3 -m http.server 8080
  # 手机或电脑浏览器访问 http://localhost:8080
  ```
- **核心界面区域**：
  1. **顶部状态栏**：显示运行状态（干跑中/已停止）、开始时间、运行总时长（动态累加）、标的数（12 个）、数据最后生成时间、手动刷新按钮与 60s 倒计时自动刷新。
  2. **汇总卡片行**：总扫描轮数、套利信号触发次数（`DECISION_ENTRY`）、曾达套利档标的数、当前处于套利档标的数。
  3. **标的实时行情与价差表（核心）**：12 个美股标的按中间价差 `mid_bps` 从高到低排序，直观展示档位（套利档/刷量档）、价差进度条（对照 10.75 bps 触发红线）、资金费差（换算为每 8h 的 bps）、流动性满足情况、杠杆及历史套利次数。
  4. **价差时序走势图**：原生 Canvas 绘制，横轴时间、纵轴 mid bps，附带 10.75 bps 红色虚线触发阈值；支持切换「全部 / 仅套利标的 / 标的单独聚焦」。
  5. **套利决策记录**：展示历次 `DECISION_ENTRY` 模拟触发的双腿计划（Arcus 主腿买/卖与 Bybit 对冲腿卖/买）。
  6. **模拟持仓盈亏（MTM）**：展示持仓时长、模拟盈亏 PnL（USD）走势曲线与基差偏离。

---

## 3. signals.json 字段说明

`signals.json` 是由解析脚本 `~/arcus-bot/parse_signals.py` 解析机器人原始日志 `/tmp/arcus_longrun.log` 生成的标准 JSON 文件：

```json
{
  "generated_at": "2026-11-20T15:02:34+08:00", // 数据生成时间戳 (ISO-8601)
  "run": {
    "started": "2026-11-20T14:59:32+08:00",     // 机器人启动时间戳
    "seconds": 28800.0,                         // 计划干跑总秒数 (如 8 小时)
    "dry_run": true,                            // 是否干跑模式 (恒为 true)
    "universe": [                               // 监控的 12 个美股标的
      "QQQ", "SPY", "AAPL", "GOOGL", "AMZN", "MSFT", 
      "NVDA", "TSLA", "SNDK", "NBIS", "DRAM", "CRCL"
    ],
    "stopped": false,                           // 机器人是否已结束
    "stop_state": null                          // 停止状态原因说明
  },
  "need_bps": 10.75,                            // 触发线 bps (往返手续费 7.75 + 3.0 缓冲)
  "latest": {
    "NVDA": {
      "ts": "2026-11-20T15:02:34+08:00",        // 最新扫描时间
      "mode": "arb",                            // 当前档位: "arb" (套利档) 或 "volume" (刷量档)
      "mid_bps": 16.32,                         // 中间价差 (基点 bps)
      "need_bps": 10.75,                        // 所需阈值 bps
      "fund_diff": 0.00011778,                  // 资金费率差额 (8h 比例，乘以 10000 为 bps)
      "liq_ok": true,                           // 流动性是否满足要求
      "leverage": 13                            // 标的杠杆倍数
    }
  },
  "summary": {
    "NVDA": {
      "scans": 7,                               // 累计扫描轮数
      "arb_scans": 7,                           // 累计进入套利档轮数
      "volume_scans": 0,                        // 累计处于刷量档轮数
      "decisions": 4,                           // 累计触发套利决策次数
      "arcus_symbol": "NVDA-USD",               // Arcus 对应交易对
      "bybit_symbol": "NVDAUSDT",               // Bybit 对应交易对
      "arcus_funding": "0.000005092592592592",  // Arcus 当前资金费
      "bybit_funding": "0.00012288",            // Bybit 当前资金费
      "bybit_mark": "144.08",                   // Bybit 标记价格
      "coin_leverage": 13                       // 杠杆
    }
  },
  "series": {
    "NVDA": [                                   // 历史时序数据 (每标的每分钟采样 1 点)
      {
        "t": "2026-11-20T15:00:43+08:00",
        "mid": 13.885,
        "mode": "arb",
        "fund": 0.0001142,
        "liq": false
      }
    ]
  },
  "decisions": [                                // 历次套利决策详情
    {
      "ts": "2026-11-20T15:02:02+08:00",
      "base": "NVDA",
      "mode": "arb",
      "plan": "arb_long_arcus_short_hl",
      "edge_bps": 16.666,
      "mid_basis_bps": 16.666,
      "need_bps": 10.75,
      "leverage": 13,
      "notional_usd": "124.999994460",
      "main_venue": "arcus",
      "main_side": "BUY",
      "main_qty": "0.8687795",
      "main_px": "143.88",
      "hedge_venue": "bybit",
      "hedge_side": "SELL",
      "hedge_qty": "0.86",
      "hedge_px": "144.12"
    }
  ],
  "mtm": [                                      // 模拟持仓盯市盈亏
    {
      "ts": "2026-11-20T15:02:35+08:00",
      "base": "NVDA",
      "mode": "arb",
      "hold_sec": 33.1,
      "pnl_usd": 0.0255,
      "adverse_pct": -0.000243,
      "mid_basis_bps": 14.588
    }
  ],
  "scan_count": 7                               // 全局总扫描轮数
}
```

---

## 4. 更新方式（操作方周期同步）

操作方每 10 分钟重跑解析器并推送到 GitHub 仓库即可完成看板数据更新：

```bash
# 1. 运行解析器，将最新日志转为 signals.json
python3 ~/arcus-bot/parse_signals.py /tmp/arcus_longrun.log /tmp/signals.json

# 2. 将数据同步到看板发布目录
cp /tmp/signals.json ~/agy-bridge/workdir/arcus-dash/signals.json

# 3. 提交并推送到 GitHub Pages 仓库
cd ~/agy-bridge/workdir/mint-board
git add arcus-dash/signals.json
git commit -m "chore(dash): update arcus dry-run signals"
git push origin main
```

*(也可将上述命令配置为 macOS crontab 或后台定时任务自动执行)*

---

## 5. 已知限制与说明

1. **部分标的合约可用性**：
   - 机器人监控的 12 个标的中，个别美股标的（例如 `SNDK`）在 Arcus 侧可能尚未正式上线或合约名称不一致（日志中呈现 Arcus HTTP 404），看板会如实显示为「未上线」或价差为 `—`，不影响其他正常标的的套利计算。
2. **模拟平仓机制（FLATTEN / FLAT_DONE_SIM）**：
   - 在干跑模式下，当模拟双腿持仓的总 PnL 达到止盈目标（如 `arb_tp combined_pnl > 0`）时，机器人会记录平仓完成，因此持仓呈现短平快的特征（一般数秒至数十秒）。
3. **GitHub Pages 缓存**：
   - GitHub Pages 的 CDN 边缘缓存时间通常约为 5~10 分钟。前端已在请求中添加时间戳 `?_t=Date.now()` 以尽量穿透浏览器端缓存。
