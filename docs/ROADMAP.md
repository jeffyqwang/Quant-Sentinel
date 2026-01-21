# Quant Sentinel - Development Roadmap

> Last Updated: 2026-01-21

## Overview

本项目采用渐进式开发策略，从本地 CLI 工具逐步演进为完整的量化交易系统。

---

## Sprint 1: Infrastructure & Data Pipeline ✅ [CURRENT]

**目标**: 建立稳固的项目基础设施和通用数据获取能力。

### Tasks

| Task | Description | Status |
|------|-------------|--------|
| 1.1 | Project Scaffolding (pnpm, TypeScript, ESM) | ✅ Done |
| 1.2 | CLI Entry Point (`src/cli.ts` with `cac`) | 🔄 In Progress |
| 1.3 | `IMarketProvider` Interface Definition | ⬜ Todo |
| 1.4 | `YahooAdapter` Implementation | ⬜ Todo |
| 1.5 | Core Indicators: MA, ATR, Slope | ⬜ Todo |
| 1.6 | `calculateMedianPrice(ohlc)` Utility | ⬜ Todo |

### Deliverables

- `pnpm start fetch --symbol AAPL` 能够获取并打印 K 线数据
- 基础技术指标库可用 (MA, ATR, Slope)

---

## Sprint 2: Persistence Layer (Docker + PostgreSQL)

**目标**: 引入数据持久化，避免重复请求 API，为回测提供数据基础。

### Tasks

| Task | Description |
|------|-------------|
| 2.1 | Docker Compose Setup (PostgreSQL + pgAdmin) |
| 2.2 | Prisma ORM Integration |
| 2.3 | Database Schema Design (Candles, Symbols) |
| 2.4 | `DbMarketProvider` Implementation |
| 2.5 | Data Sync Command (`pnpm start sync`) |

### Deliverables

- 本地 Docker 环境一键启动
- K 线数据自动缓存到数据库
- 支持从 DB 读取历史数据进行分析

---

## Sprint 3: Backtesting Engine & Operations

**目标**: 构建回测引擎，验证策略有效性；准备生产环境部署。

### Tasks

| Task | Description |
|------|-------------|
| 3.1 | Backtesting Engine Core |
| 3.2 | Strategy Interface (`IStrategy`) |
| 3.3 | Performance Metrics (Sharpe, MaxDrawdown, Win Rate) |
| 3.4 | Linux VPS Setup (Ubuntu) |
| 3.5 | PM2 Process Management |
| 3.6 | Basic Monitoring & Alerts |

### Deliverables

- `pnpm start backtest --strategy system3` 可运行回测
- 策略必须通过回测验证才能进入 Live

---

## Sprint 4: Live Trading & AI Integration

**目标**: 接入真实券商，实现自动化交易；引入 AI 辅助分析。

### Tasks

| Task | Description |
|------|-------------|
| 4.1 | Futu OpenD SDK Integration |
| 4.2 | `FutuAdapter` Implementation |
| 4.3 | Order Execution Module |
| 4.4 | Portfolio Sync (Holdings, Watchlist) |
| 4.5 | AI Analysis Integration (LLM for Market Sentiment) |
| 4.6 | Risk Management Safeguards |

### Deliverables

- 实盘信号自动下单
- AI 辅助市场分析报告
- 完整的风控机制

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.2.0 | 2026-01-21 | Roadmap 重构，新增 ATR/Slope 指标，调整 Sprint 规划 |
| 0.1.0 | Initial | 初始版本 |
