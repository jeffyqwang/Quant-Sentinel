# Quant Sentinel - Strategy Lab

> 策略研究与实验文档

---

## Core System: System 3.0

**状态**: 🔬 Research & Development

System 3.0 是本项目的核心交易系统，基于自定义 MACD 指标。

### 指标定义

| Parameter | Value | Note |
|-----------|-------|------|
| MACD Fast | 5 | 短周期 EMA |
| MACD Slow | 13 | 长周期 EMA |
| MACD Signal | 5 | 信号线 EMA |
| Data Source | (High + Low) / 2 | **Median Price** (非 Close) |

### 辅助指标

- **MA60**: 主趋势过滤器 (Price > MA60 = 多头区域)
- **MA120**: 辅助趋势确认

### 交易逻辑

1. **Strategic Layer (System 2.0)**
   - Input: 任意 K 线序列
   - Rule: Price > MA60 → Bullish Zone

2. **Tactical Layer (System 3.0)**
   - Multi-timeframe: 4H 定方向 + 30m 找入场
   - Patterns:
     - **Mode 1**: V-shape Reversal
     - **Mode 2**: Divergence

---

## Experimental Strategy: Trend Breakout (趋势突破)

**状态**: 🧪 Experimental  
**来源**: 基于胡曼曼 EA 策略改良

### 核心思想

捕捉趋势突破后的延续行情，使用 ATR 动态止损和 Slope 动量过滤。

### 指标组合

| Indicator | Purpose | Parameters |
|-----------|---------|------------|
| **ATR** | Volatility & Position Sizing | Period: 14 |
| **Slope** | Momentum Filter | Period: 20 |
| **MA** | Trend Direction | Period: 60 |

### 仓位管理 (ATR-Based)

```
Position Size = (Account Risk %) / (ATR × Multiplier)

Example:
- Account: $10,000
- Risk per trade: 1% ($100)
- ATR(14): 2.5
- Multiplier: 2

Position Size = $100 / (2.5 × 2) = 20 units
Stop Loss = Entry - (ATR × 2)
```

### 入场条件 (Draft)

1. Price > MA60 (趋势过滤)
2. Slope > 0 (动量确认)
3. 价格突破前高/前低
4. ATR 未处于极端值 (避免异常波动)

### 出场条件 (Draft)

- **止损**: Entry - (ATR × 2)
- **移动止损**: 跟踪 ATR trailing
- **目标**: Risk:Reward = 1:2 或 1:3

---

## Strategy Development Rules

### 验证流程 (Mandatory)

```
[Idea] → [Code] → [Backtest] → [Paper Trade] → [Live]
           ↓
      Must Pass All Stages
```

### 回测要求

| Metric | Minimum Threshold |
|--------|-------------------|
| Win Rate | > 40% |
| Risk/Reward | > 1:1.5 |
| Max Drawdown | < 20% |
| Sample Size | > 100 trades |

### 禁止事项

- ❌ 未经回测直接实盘
- ❌ 在单一品种过度优化 (Overfitting)
- ❌ 忽略手续费和滑点

---

## Indicator Implementation Checklist

### Sprint 1 Scope

- [ ] `calculateMedianPrice(candle)` - 中位价计算
- [ ] `calculateMA(data, period)` - 简单移动平均
- [ ] `calculateEMA(data, period)` - 指数移动平均
- [ ] `calculateATR(candles, period)` - 真实波幅均值
- [ ] `calculateSlope(data, period)` - 斜率/动量

### Future Scope

- [ ] `calculateMACD(data, fast, slow, signal)` - 自定义 MACD
- [ ] `detectDivergence(price, indicator)` - 背离检测
- [ ] `identifyFractal(candles)` - 分形识别

---

## Research Notes

### ATR 用途总结

1. **止损设置**: Stop = ATR × N (通常 N = 1.5~3)
2. **仓位计算**: 基于账户风险百分比
3. **波动过滤**: ATR 过高时避免入场
4. **趋势强度**: ATR 扩张 = 趋势加速

### Slope 指标说明

```typescript
// Slope = (Current MA - Previous MA) / Period
// Positive slope = Uptrend momentum
// Negative slope = Downtrend momentum
```

用于过滤假突破，只在动量方向一致时入场。
