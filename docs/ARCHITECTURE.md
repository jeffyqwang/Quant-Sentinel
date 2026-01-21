# Quant Sentinel - Architecture Overview

> 系统架构与设计决策文档

---

## Application Type

**CLI Application** - 命令行工具，非库 (Library)。

```bash
# Usage Examples
pnpm start fetch --symbol AAPL --interval 1d
pnpm start analyze --symbol HSTECH
pnpm start backtest --strategy system3
```

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Runtime | Node.js (LTS) | JavaScript 运行时 |
| Language | TypeScript (Strict) | 类型安全 |
| Package Manager | pnpm | 快速、节省磁盘 |
| TS Executor | tsx | 直接运行 .ts 文件 |
| CLI Framework | cac | 轻量级命令行解析 |
| Data (Sprint 1) | yahoo-finance2 | 公开市场数据 |
| Data (Sprint 2) | PostgreSQL + Prisma | 数据持久化 |
| Data (Sprint 4) | Futu OpenD | 券商 API |
| Containerization | Docker | 本地开发环境 |
| Process Manager | PM2 | 生产环境进程管理 |

---

## Project Structure

```
quant-sentinel/
├── docs/                    # 项目文档
│   ├── ARCHITECTURE.md      # 本文件
│   ├── ROADMAP.md           # 开发路线图
│   └── STRATEGY_LAB.md      # 策略研究
│
├── src/
│   ├── cli.ts               # ⭐ 唯一入口点
│   │
│   ├── commands/            # CLI 命令实现
│   │   ├── fetch.ts         # 数据获取命令
│   │   ├── analyze.ts       # 分析命令
│   │   └── backtest.ts      # 回测命令
│   │
│   ├── core/                # 核心业务逻辑
│   │   ├── indicators/      # 技术指标
│   │   │   ├── ma.ts
│   │   │   ├── atr.ts
│   │   │   └── slope.ts
│   │   └── strategies/      # 交易策略
│   │       └── system3.ts
│   │
│   ├── adapters/            # 外部数据适配器
│   │   ├── yahoo.adapter.ts
│   │   ├── db.adapter.ts    # Sprint 2
│   │   └── futu.adapter.ts  # Sprint 4
│   │
│   ├── interfaces/          # TypeScript 类型定义
│   │   ├── candle.ts        # K线数据结构
│   │   ├── market-provider.ts
│   │   └── strategy.ts
│   │
│   └── utils/               # 通用工具函数
│       └── median-price.ts
│
├── prisma/                  # Sprint 2: 数据库 Schema
│   └── schema.prisma
│
├── docker/                  # Sprint 2: 容器配置
│   └── docker-compose.yml
│
├── package.json
├── tsconfig.json
└── .cursorrules
```

---

## Design Patterns

### 1. Adapter Pattern (数据层)

所有外部数据源都实现统一的 `IMarketProvider` 接口。

```typescript
interface IMarketProvider {
  getCandles(symbol: string, interval: string, limit?: number): Promise<Candle[]>;
  getQuote(symbol: string): Promise<Quote>;
}

// Implementations
class YahooAdapter implements IMarketProvider { ... }
class DbMarketProvider implements IMarketProvider { ... }
class FutuAdapter implements IMarketProvider { ... }
```

**好处**:
- 数据源可随时切换
- 策略代码无需修改
- 便于测试 (Mock)

### 2. Service Layer (业务层)

业务逻辑封装在 Service 中，与数据获取解耦。

```typescript
class AnalysisService {
  constructor(private provider: IMarketProvider) {}
  
  async analyzeSymbol(symbol: string): Promise<AnalysisResult> {
    const candles = await this.provider.getCandles(symbol, '1d');
    // ... 指标计算和分析逻辑
  }
}
```

### 3. Interface First (类型优先)

永远先定义接口，再写实现。

```typescript
// Step 1: Define interface
interface IStrategy {
  name: string;
  analyze(candles: Candle[]): Signal[];
  backtest(candles: Candle[]): BacktestResult;
}

// Step 2: Implement
class System3Strategy implements IStrategy { ... }
```

---

## Data Flow

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   CLI       │────▶│   Service    │────▶│  Adapter    │
│  (cac)      │     │   Layer      │     │  (Yahoo/DB) │
└─────────────┘     └──────────────┘     └─────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Indicators  │
                    │  (MA/ATR)    │
                    └──────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Strategy   │
                    │  (System 3)  │
                    └──────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Output     │
                    │  (Console)   │
                    └──────────────┘
```

---

## Key Decisions

### 为什么选择 CLI 而非 Web/API？

1. **简单直接**: 量化分析核心是计算，不需要复杂 UI
2. **脚本友好**: 可配合 cron/PM2 定时执行
3. **资源轻量**: 无需 HTTP 服务器开销
4. **开发高效**: 快速迭代，专注业务逻辑

### 为什么用 pnpm 而非 npm/yarn？

1. **磁盘效率**: 硬链接复用，节省空间
2. **安装速度**: 比 npm 快 2-3 倍
3. **严格依赖**: 避免幽灵依赖 (phantom dependencies)
4. **Workspace**: 未来 monorepo 友好

### 为什么用 tsx 而非 ts-node？

1. **更快**: 基于 esbuild，冷启动快
2. **零配置**: 无需额外 tsconfig 调整
3. **ESM 友好**: 原生支持 ES Modules
4. **Watch Mode**: 内置热重载支持

---

## Error Handling Strategy

```typescript
// 使用 Result 模式而非 throw
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

async function fetchData(): Promise<Result<Candle[]>> {
  try {
    const data = await provider.getCandles(...);
    return { success: true, data };
  } catch (error) {
    return { success: false, error };
  }
}
```

---

## Future Considerations

### Sprint 2: Database Integration

- PostgreSQL 存储历史 K 线
- Prisma ORM 类型安全查询
- 增量同步策略

### Sprint 4: Broker Integration

- Futu OpenD 本地网关
- WebSocket 实时行情
- 订单状态同步
