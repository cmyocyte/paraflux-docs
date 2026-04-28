# Volatility Data

Paraflux provides on-chain realized variance data across 9 markets -- crypto, commodities, equities, and forex.

> **Important:** This is realized variance (what actually happened), not implied volatility (what the market expects). See [Realized Vol Oracle](../protocol/vol-oracle.md) for the full explanation.

## Available Data

| Method | Returns | Description |
|--------|---------|-------------|
| `getVolSurface(asset)` | `VolSurface` | Complete vol snapshot: term structure, vol-of-vol, realized vol |
| `getAllVolSurfaces()` | `Map<VolAsset, VolSurface>` | All 9 markets in parallel |
| `getCurrentVariance(asset)` | `bigint` | 1-day windowed variance (WAD) |
| `getRealizedVol(asset)` | `number` | Annualized vol as float |
| `getVolOfVol(asset)` | `bigint` | Regime detection metric (WAD) |
| `getVolTermStructure(asset)` | `VolTermStructure` | 1d/7d/30d variances + vol-of-vol |
| `getForwardVariance(asset)` | `bigint` | Forward vol between 1d and 7d |
| `getVolHistory(asset, from, to)` | `VolObservation[]` | Historical observations from chain |
| `getVolCorrelationMatrix(assets)` | `CorrelationMatrix` | Cross-asset correlation |
| `watchAllVolSurfaces(callback)` | unwatch fn | Real-time streaming |

## Available Assets

```typescript
type VolAsset = 'BTC' | 'ETH' | 'SOL' | 'HYPE' | 'XYZ100' | 'TSLA' | 'GOLD' | 'SILVER' | 'EURUSD';
```

## The VolSurface Object

```typescript
interface VolSurface {
  asset: VolAsset;
  termStructure: {
    var1d: bigint;    // 1-day variance (WAD)
    var7d: bigint;    // 7-day variance
    var30d: bigint;   // 30-day variance
    volOfVol: bigint; // regime detection
  };
  forwardVariance: bigint;     // forward vol 1d->7d
  realizedVol1d: number;       // sqrt(var1d) -- annualized
  observationCount: bigint;
  lastObservedSpot: bigint;
  isInitialized: boolean;
}
```

## Use Cases

### Cross-Asset Vol Dashboard
```typescript
const surfaces = await client.getAllVolSurfaces();
for (const [asset, s] of surfaces) {
  const vol = (s.realizedVol1d * 100).toFixed(1);
  const vov = (math.varianceToVol(s.termStructure.volOfVol) * 100).toFixed(1);
  console.log(`${asset.padEnd(8)} vol=${vol}%  VoV=${vov}%`);
}
```

### Regime Detection
```typescript
const surface = await client.getVolSurface('HYPE');
const vov = math.varianceToVol(surface.termStructure.volOfVol);
if (vov > 0.5) console.log('REGIME TRANSITION -- reduce exposure');
else if (vov > 0.2) console.log('ELEVATED -- monitor closely');
else console.log('STABLE');
```

### Funding Rate Forecasting
```typescript
const surface = await client.getVolSurface('HYPE');
// Realized vol drives the funding rate (sigma^2)
const expectedAnnualFunding = surface.realizedVol1d ** 2;
console.log(`Expected annual funding: ${(expectedAnnualFunding * 100).toFixed(1)}%`);
```

### Historical Vol Time Series
```typescript
const history = await client.getVolHistory('HYPE', fromBlock);
const vols = history.map(h => h.realizedVol * 100);
console.log(`Vol range: ${Math.min(...vols).toFixed(1)}% - ${Math.max(...vols).toFixed(1)}%`);
```

### Cross-Asset Correlation
```typescript
const corr = await client.getVolCorrelationMatrix(['BTC', 'ETH', 'HYPE', 'GOLD']);
// corr.matrix[0][1] = BTC-ETH vol correlation
```
