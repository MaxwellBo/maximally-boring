# IMC Prosperity 2 visualizer

```js
const logFile = FileAttachment("data/28eb68e2-818d-46e4-9a45-44f936224284.log").text()
```

```js
view(logFile)
```


```js
const kMovingAverage = view(Inputs.range([1, 1000], {
  label: "Moving average window (k)",
  step: 10
}))
```


```js
const unsettledTimestampRange = view(interval(
  [0, d3.max(prices, (d) => d.timestamp)],
  {
    label: "timestamp range (delayed by 1 sec)",
    step: 1
  }
))
```

```js
const showTrades = view(Inputs.toggle({ label: "Show log trades", value: true }))
```

```js
const scopedZippedPrices = zippedPrices.filter(
  (d) => timestampRange[0] <= d.timestamp && d.timestamp <= timestampRange[1]
);

view(Plot.plot({
  title: "ETF diff chart",
  color: {
    legend: true
  },
  y: {
    label: "price"
  },
  marginLeft: 50,
  width,
  marks: [
    Plot.differenceY(scopedZippedPrices, {
      x: "timestamp",
      y1: "unit_mid_price",
      y2: "aggregate_mid_price",
      positiveFill: "green",
      negativeFill: "purple"
    }),
    Plot.lineY(scopedZippedPrices, {
      x: "timestamp",
      y: "unit_mid_price",
      strokeWidth: 3,
      stroke: (d) => "GIFT_BASKET",
      curve: "step-after",
      tip: true
    }),
    // Plot.lineY(scopedZippedPrices, {
    //   x: "timestamp",
    //   y: (d) => d.unit_mid_price - d.aggregate_mid_price
    //   // stroke: "6 * STRAWBERRIES + 4 * CHOCOLATE + ROSES"
    // }),
    Plot.lineY(scopedZippedPrices, {
      x: "timestamp",
      y: "aggregate_mid_price",
      strokeWidth: 3,
      stroke: (d) => "6 * STRAWBERRIES + 4 * CHOCOLATE + ROSES",
      tip: true,
      curve: "step-after"
    })
  ]
}))
```

```js
{
  const scopedZippedPrices = zippedPrices.filter(
    (d) => timestampRange[0] <= d.timestamp && d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    color: {
      legend: true
    },
    y: {
      label: "diff"
    },
    marginLeft: 50,
    width,
    marks: [
      Plot.lineY(scopedZippedPrices, {
        x: "timestamp",
        y: (d) => d.unit_mid_price - d.aggregate_mid_price,
        curve: "step-after"
      })
    ]
  })
}
```

```js echo
{
  const product = "GIFT_BASKET";
  const scopedPrices = prices.filter(
    (d) =>
      d.product == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );
  const scopedTrades = trades.filter(
    (d) =>
      d.symbol == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );
  const scopedTidyPrices = tidyPrices.filter(
    (d) =>
      d.product == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );
  const scopedZippedPrices = zippedPrices.filter(
    (d) => timestampRange[0] <= d.timestamp && d.timestamp <= timestampRange[1]
  );

  return Plot.plot({
    title: `Custom ${product} price and order depth`,
    width,
    caption:
      "Line is [bid/ask]_price_1, off-line dots are [bid/ask]_price_[2/3]",
    color: {
      label: "volume",
      legend: true,
      scheme: "BuRd"
    },
    symbol: {
      domain: ["volume", "trades"],
      legend: true
    },
    marks: [
      Plot.lineY(scopedPrices, {
        x: "timestamp",
        y: "bid_price_1",
        z: null,
        stroke: "bid_volume_1"
      }),
      Plot.lineY(scopedPrices, {
        x: "timestamp",
        y: "ask_price_1",
        z: null,
        stroke: (d) => -d.ask_volume_1
      }),
      Plot.dot(scopedTidyPrices, {
        x: "timestamp",
        y: "price",
        fill: "volume",
        symbol: (d) => "volume"
      }),
      Plot.lineY(scopedZippedPrices, {
        x: "timestamp",
        y: (d) => d.unit_mid_price - d.aggregate_mid_price + 70000
      }),
      showTrades
        ? Plot.dot(scopedTrades, {
            x: "timestamp",
            y: "price",
            stroke: (d) => (d.quantity > 0 ? "green" : "orange"),
            tip: true,
            fill: "quantity",
            symbol: (d) => "trades"
          })
        : undefined,
      Plot.tickX(scopedTrades, {
        x: "timestamp",
        stroke: (d) => (d.quantity > 0 ? "green" : "orange"),
        symbol: (d) => "trades"
      }),
      Plot.lineY(
        scopedPrices,
        Plot.windowY({
          x: "timestamp",
          y: "mid_price",
          k: kMovingAverage,
          reduce: "mean"
        })
      )
    ]
  });
}
```

```js
plotPrice("GIFT_BASKET")
```

```js
{
  const scopedPrices = prices.filter(
    (d) => timestampRange[0] <= d.timestamp && d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    title: "Product profit and loss",
    color: {
      legend: true
    },
    width,
    marginLeft: 50,
    marks: [
      Plot.ruleY([0]),
      Plot.lineY(
        scopedPrices.filter((d) => d.product != "COCONUT_COUPON"),
        {
          x: "timestamp",
          y: "profit_and_loss",
          stroke: "product",
          curve: "step-after"
        }
      )
    ]
  });
}
```

```js echo
plotPrice("STARFRUIT")
```

```js echo
listTrades("STARFRUIT")
```

```js echo
plotVolume("STARFRUIT")
```

```js echo
plotRootedMidVolume("STARFRUIT")
```

```js echo
plotPrice("AMETHYSTS")
```

```js echo
plotVolume("AMETHYSTS")
```

```js echo
plotRootedMidVolume("AMETHYSTS")
```

```js echo
plotPrice("ORCHIDS")
```

```js
viewof table = Inputs.table(trades.filter((d) => d.symbol === "ORCHIDS"))
```

```js echo
plotRootedMidVolume("ORCHIDS")
```

---

## Implementation

### Timestep filtering

```js
import { interval } from "@mootari/range-slider"
```

```js
import { settle } from "@mjbo/settling-input"
```

We need to "settle" the input because immediately filtering with the new input tanks the browser. This will delay plot recomputation by 1 second.

```js echo
// const timestampRange = settle(viewof unsettledTimestampRange)
```

```js echo
const timestampRange = [0, 99900]
```

### Data cleaning

Now lets the load the selected log files. Reminder, the selected log files are:

```js echo
const chunks_ = logFile.split("\n\n\n\n");
const sandbox = chunks[0].split("Sandbox logs:\n")[1];
const activities = chunks[1].split("Activities log:\n")[1];
const trades = JSON.parse(chunks[2].split("\nTrade History:\n")[1]);

const chunks = ({
    sandbox,
    activities,
    trades
});

const logger = JSON.parse(`[${chunks.sandbox.replaceAll("}\n{", "},{")}]`).map(
  (d) => ({
    ...d
    
  })
)
```

They're stored with `;` separators instead of `,`s, so we replace the `;` with `,`s and then pass that modified file to the **COMMA** separated value parser (fucking IMC).

```js echo
const prices = d3.csvParse(chunks.activities.replace(/;/g, ","), d3.autoType)
```

```js echo
const trades = chunks.trades
```

Now let's create some ["tidy data"](https://tidyr.tidyverse.org/) - a single row per observation. This will make it easy to visualize in some situations.

```js echo
const tidyPrices = prices.flatMap(
  ({
    timestamp,
    product,
    bid_price_1,
    bid_volume_1,
    bid_price_2,
    bid_volume_2,
    bid_price_3,
    bid_volume_3,
    ask_price_1,
    ask_volume_1,
    ask_price_2,
    ask_volume_2,
    ask_price_3,
    ask_volume_3,
    mid_price
  }) => [
    {
      type: "bid_3",
      timestamp,
      product,
      price: bid_price_3,
      volume: bid_volume_3,
      mid_price
    },
    {
      type: "bid_2",
      timestamp,
      product,
      price: bid_price_2,
      volume: bid_volume_2,
      mid_price
    },
    {
      type: "bid_1",
      timestamp,
      product,
      price: bid_price_1,
      volume: bid_volume_1,
      mid_price
    },
    {
      type: "ask_1",
      timestamp,
      product,
      price: ask_price_1,
      volume: -ask_volume_1,
      mid_price
    },
    {
      type: "ask_2",
      timestamp,
      product,
      price: ask_price_2,
      volume: -ask_volume_2,
      mid_price
    },
    {
      type: "ask_3",
      timestamp,
      product,
      price: ask_price_3,
      volume: -ask_volume_3,
      mid_price
    }
  ]
)
```

```js echo
const scaledPrices = prices
  .map((p) => {
    if (p.product === "STRAWBERRIES") {
      return { ...p, scaled_mid_price: p.mid_price * 6 };
    }

    if (p.product === "CHOCOLATE") {
      return { ...p, scaled_mid_price: p.mid_price * 4 };
    }

    if (p.product === "ROSES") {
      return { ...p, scaled_mid_price: p.mid_price * 1 };
    }
  })
  .filter((d) => d)
```

```js echo
const aggregatePrices = Object.entries(
  _.groupBy(scaledPrices, (d) => d.timestamp)
).map(([k, v]) => ({
  timestamp: Number(k),
  aggregate_mid_price: _.sumBy(v, (d) => d.scaled_mid_price)
}))
```

```js echo
const zippedPrices = aggregatePrices.map((d) => {
  const pairedGiftBasket = prices.find(
    (p) => p.timestamp === d.timestamp && p.product === "GIFT_BASKET"
  );

  return {
    ...d,
    unit_mid_price: pairedGiftBasket.mid_price,
    pairedGiftBasket
  };
})
```

### Plots

```js echo
function plotPrice(product) {
  const scopedPrices = prices.filter(
    (d) =>
      d.product == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );
  const scopedTrades = trades.filter(
    (d) =>
      d.symbol == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );
  const scopedTidyPrices = tidyPrices.filter(
    (d) =>
      d.product == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );

  return Plot.plot({
    title: `${product} price and order depth`,
    width,
    caption:
      "Line is [bid/ask]_price_1, off-line dots are [bid/ask]_price_[2/3]",
    color: {
      label: "volume",
      legend: true,
      scheme: "BuRd"
    },
    symbol: {
      domain: ["volume", "trades"],
      legend: true
    },
    marks: [
      Plot.lineY(scopedPrices, {
        x: "timestamp",
        y: "bid_price_1",
        z: null,
        stroke: "bid_volume_1"
      }),
      Plot.lineY(scopedPrices, {
        x: "timestamp",
        y: "ask_price_1",
        z: null,
        stroke: (d) => -d.ask_volume_1
      }),
      Plot.dot(scopedTidyPrices, {
        x: "timestamp",
        y: "price",
        fill: "volume",
        symbol: (d) => "volume"
      }),
      showTrades
        ? Plot.dot(scopedTrades, {
            x: "timestamp",
            y: "price",
            stroke: "green",
            tip: true,
            fill: "quantity",
            symbol: (d) => "trades"
          })
        : undefined,
      Plot.lineY(
        scopedPrices,
        Plot.windowY({
          x: "timestamp",
          y: "mid_price",
          k: kMovingAverage,
          reduce: "mean"
        })
      )
    ]
  });
}
```

```js echo
function listTrades(product) {
  const scopedTrades = trades.filter(
    (d) =>
      d.symbol == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );

  return Inputs.table(scopedTrades);
}
```

```js echo
function plotPnl() {
  const scopedPrices = prices.filter(
    (d) => timestampRange[0] <= d.timestamp && d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    title: "Product profit and loss",
    color: {
      legend: true
    },
    width,
    marks: [
      Plot.ruleY([0]),
      Plot.lineY(scopedPrices, {
        x: "timestamp",
        y: "profit_and_loss",
        stroke: "product"
      })
    ]
  });
}
```

```js echo
function plotVolume(product) {
  const scopedPrices = tidyPrices.filter(
    (d) =>
      d.product == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    title: `${product} volume`,
    width,
    height: 300,
    y: {
      label: "abs(volume)"
    },
    color: {
      label: "volume",
      legend: true,
      scheme: "BuRd"
    },
    marks: [
      Plot.dot(scopedPrices, {
        x: "timestamp",
        y: (d) => Math.abs(d.volume),
        tip: true,
        fill: "volume",
        fx: "type"
      }),
      Plot.ruleY([0])
    ]
  });
}
```

```js echo
function plotRootedMidVolume(product) {
  const scopedPrices = tidyPrices.filter(
    (d) =>
      d.product == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    title: `${product} volume around mid price`,
    width,
    y: {
      label: "price - mid_price",
      domain: [-7, 7]
    },
    color: {
      label: "volume",
      legend: true,
      scheme: "BuRd"
    },
    marks: [
      Plot.dot(scopedPrices, {
        x: "timestamp",
        y: (d) => d.price - d.mid_price,
        tip: true,
        fill: "volume",
        fx: "type"
      }),
      Plot.ruleY([0])
    ]
  });
}
```

```js echo
plotSpread("STARFRUIT")
```

```js echo
function plotSpread(product) {
  const scopedPrices = prices.filter(
    (d) =>
      d.product == product &&
      timestampRange[0] <= d.timestamp &&
      d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    title: `${product} volume around mid price`,
    width,
    y: {
      label: "price - mid_price"
    },
    color: {
      label: "volume",
      legend: true,
      scheme: "BuRd"
    },
    marks: [
      Plot.dot(scopedPrices, {
        x: "timestamp",
        y: (d) => d.ask_price_1 - (d.ask_price_2 || d.ask_price_1),
        tip: true
        // fill: "volume",
      })
    ]
  });
}
```

---

## WIP

The following is just the JSON dumped from `TradingState.toJson()` dumped into a JS array.

```js
states = [{"listings": {"PRODUCT1": {"denomination": "SEASHELLS", "product": "PRODUCT1", "symbol": "PRODUCT1"}, "PRODUCT2": {"denomination": "SEASHELLS", "product": "PRODUCT2", "symbol": "PRODUCT2"}}, "market_trades": {"PRODUCT1": [{"buyer": "", "price": 11, "quantity": 4, "seller": "", "symbol": "PRODUCT1", "timestamp": 900}], "PRODUCT2": []}, "observations": {}, "order_depths": {"PRODUCT1": {"buy_orders": {"9": 5, "10": 7}, "sell_orders": {"11": -4, "12": -8}}, "PRODUCT2": {"buy_orders": {"141": 5, "142": 3}, "sell_orders": {"144": -5, "145": -8}}}, "own_trades": {"PRODUCT1": [], "PRODUCT2": []}, "position": {"PRODUCT1": 3, "PRODUCT2": -5}, "timestamp": 1000, "traderData": ""}, {"listings": {"PRODUCT1": {"denomination": "SEASHELLS", "product": "PRODUCT1", "symbol": "PRODUCT1"}, "PRODUCT2": {"denomination": "SEASHELLS", "product": "PRODUCT2", "symbol": "PRODUCT2"}}, "market_trades": {"PRODUCT1": [], "PRODUCT2": []}, "observations": {}, "order_depths": {"PRODUCT1": {"buy_orders": {"9": 5, "10": 7}, "sell_orders": {"12": -5, "13": -3}}, "PRODUCT2": {"buy_orders": {"141": 5, "142": 3}, "sell_orders": {"144": -5, "145": -8}}}, "own_trades": {"PRODUCT1": [{"buyer": "SUBMISSION", "price": 11, "quantity": 4, "seller": "", "symbol": "PRODUCT1", "timestamp": 1000}, {"buyer": "SUBMISSION", "price": 12, "quantity": 3, "seller": "", "symbol": "PRODUCT1", "timestamp": 1000}], "PRODUCT2": [{"buyer": "", "price": 143, "quantity": 2, "seller": "SUBMISSION", "symbol": "PRODUCT2", "timestamp": 1000}]}, "position": {"PRODUCT1": 10, "PRODUCT2": -7}, "timestamp": 1100, "traderData": ""}]
```

```js echo
tidyStates = {
  const acc = [];

  for (const state of states) {
    const timestamp = state.timestamp;
    for (const [symbol, position] of Object.entries(state.position ?? {})) {
      acc.push({
        timestamp,
        symbol,
        position
      });
    }

    for (const [symbol, orderDepth] of Object.entries(state.order_depths)) {
      for (const [price, depth] of Object.entries(
        orderDepth.buy_orders ?? {}
      )) {
        acc.push({
          symbol,
          timestamp,
          price: Number(price),
          depth,
          depth_type: "BUY"
        });
      }

      for (const [price, depth] of Object.entries(
        orderDepth.sell_orders ?? {}
      )) {
        acc.push({
          symbol,
          timestamp,
          price: Number(price),
          depth,
          depth_type: "SELL"
        });
      }
    }
  }

  return acc;
}
```

With our tidy data, we can chart the product holdings over time:

Or the order depth over time for given products.

```js echo
orderDepthForSymbol("PRODUCT1")
```

```js echo
orderDepthForSymbol("PRODUCT2")
```

```js echo
orderDepthForSymbol = (symbol) =>
  Plot.plot({
    color: {
      legend: true,
      scheme: "BuRd"
    },
    marks: [
      Plot.dot(
        tidyStates.filter((d) => d.symbol === symbol),
        {
          x: "timestamp",
          y: "price",
          r: 20,
          fill: "depth"
        }
      ),
      Plot.text(
        tidyStates.filter((d) => d.symbol === symbol),
        {
          x: "timestamp",
          y: "price",
          text: "depth",
          fill: "black",
          stroke: "white"
        }
      )
    ]
  })
```

```js echo
Plot.plot({
  color: {
    legend: true
  },
  marks: [
    Plot.ruleY([0]),
    Plot.line(
      tidyStates.filter((d) => d.position != null),
      {
        x: "timestamp",
        y: "position",
        stroke: "symbol"
      }
    )
  ]
})
```

---

```js echo
tidyTraderTrades = trades.flatMap((d) => [
  {
    ...d,
    trader: d.buyer,
    position: d.quantity,
    delta: -(d.quantity * d.price)
  },
  {
    ...d,
    trader: d.seller,
    position: -d.quantity,
    delta: d.quantity * d.price
  }
])
```

```js
traders = new Set(tidyTraderTrades.map((d) => d.trader))
```

```js
plotPositionCumSum = (symbol) => {
  const scopedTrades = tidyTraderTrades.filter(
    (d) => timestampRange[0] <= d.timestamp && d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    title: `${symbol} position per trader`,
    color: {
      domain: [...traders],
      legend: true
    },
    fx: {
      domain: [...traders]
    },
    width,
    marginRight: 100,
    marks: [
      Plot.frame(),
      Plot.ruleY([0]),
      Plot.tickX(
        scopedTrades.filter(
          (d) => d.trader == "SUBMISSION" && d.symbol === symbol
        ),
        {
          x: "timestamp",
          stroke: (d) => (d.quantity > 0 ? "purple" : "orange")
        }
      ),
      Plot.lineY(
        scopedTrades.filter((d) => d.symbol === symbol),
        Plot.mapY("cumsum", {
          stroke: "trader",
          fx: facetByTrader ? "trader" : undefined,
          x: "timestamp",
          y: "position",
          // stroke: "green",
          curve: "step-after",
          tip: true
          // fill: "quantity",
        })
      )
    ]
  });
}
```

```js
{
  const scopedTrades = tidyTraderTrades.filter(
    (d) => timestampRange[0] <= d.timestamp && d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    title: "Trader position per product",
    color: {
      domain: [...traders],
      legend: true
    },
    width,
    marginRight: 100,
    marks: [
      Plot.frame(),
      Plot.ruleY([0]),
      Plot.lineY(
        scopedTrades,
        Plot.mapY("cumsum", {
          stroke: "trader",
          fy: "symbol",
          fx: "trader",
          x: "timestamp",
          y: "position",
          // stroke: "green",
          tip: true,
          // fill: "quantity",
          symbol: (d) => "trades"
        })
      )
    ]
  });
}
```

```js
{
  const scopedTrades = tidyTraderTrades.filter(
    (d) => timestampRange[0] <= d.timestamp && d.timestamp <= timestampRange[1]
  );
  return Plot.plot({
    title: "Balance per trader",
    color: {
      domain: [...traders],
      legend: true
    },
    width,
    marginLeft: 100,
    marginRight: 100,
    marks: [
      Plot.frame(),
      Plot.ruleY([0]),
      Plot.lineY(
        scopedTrades,
        Plot.mapY("cumsum", {
          fx: "trader",
          fy: "symbol",
          x: "timestamp",
          y: "delta",
          stroke: "trader",
          tip: true,
          // fill: "quantity",
          symbol: (d) => "trades"
        })
      )
    ]
  });
}
```

```js
viewof facetByTrader = Inputs.toggle({ label: "Facet by trader", value: false })
```

```js
plotPositionCumSum("AMETHYSTS")
```

```js
plotPositionCumSum("CHOCOLATE")
```

```js
plotPositionCumSum("COCONUT")
```

```js
plotPositionCumSum("COCONUT_COUPON")
```

```js
plotPositionCumSum("GIFT_BASKET")
```

```js
plotPositionCumSum("ORCHIDS")
```

```js
plotPositionCumSum("ROSES")
```

```js
plotPositionCumSum("STARFRUIT")
```

```js
plotPositionCumSum("STRAWBERRIES")
```
