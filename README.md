# ChainSight

Mobile inventory-planning tool for people with no supply-chain background. It works out, for every part you stock:

- **Order size** — Economic Order Quantity: `sqrt( (2 × annual_demand × ordering_cost) / (unit_cost × holding_cost_pct) )`
- **Buffer** — safety stock: `z(service level) × demand_std_dev × sqrt(lead_time_days)`
- **Order at** — reorder point: `average daily demand × lead_time_days + safety stock`
- Annual storage cost, ordering cost and total cost at that order size

It then runs a 180-day simulation with random daily demand, charting stock on hand, order triggers and any stockouts, and rolls the whole portfolio up into a dashboard (cash tied up, yearly cost, parts at risk). The service level is a live slider — everything recalculates as you drag.

Everything runs client-side. No backend, no build step, no dependencies to install.

## Run it

Open `index.html` in a browser. That file is fully self-contained (design system, fonts, logo and app logic inlined).

## Using it

1. **Data** tab → *Load sample data* for 15 demo parts, or upload/paste a CSV.
2. **Dash** → portfolio totals, cost breakdown, at-risk parts, service-level slider.
3. **Parts** → sortable table; tap a row for that part's plan and simulation.
4. **Learn** → every term in plain English with the arithmetic underneath.

### CSV format

```
SKU,annual_demand,lead_time_days,unit_cost,holding_cost_pct,ordering_cost,demand_std_dev,description
FST-1042,42000,14,0.42,0.22,85,38,Hex bolt M8x40
```

A header row is optional, `description` is optional, and `holding_cost_pct` accepts either `0.22` or `22`. See `chainsight-sample.csv`. Rows with missing or zero values are excluded and listed under the table with the reason.

## Languages

Any language: the language button in the header translates the whole interface on the fly (Claude API when running inside a host that provides it) and caches translations in `localStorage`. Right-to-left languages flip the layout.

## Source layout

```
index.html                      self-contained build — open this
chainsight-sample.csv           demo data
src/ChainSight Mobile.dc.html   the authored source (template + logic class)
src/support.js                  Design Component runtime
src/ios-frame.jsx               iPhone frame used for the mobile presentation
src/_ds/…                       design tokens and component CSS
src/assets/chainsight-mark.png  logo mark
```

Edit `src/ChainSight Mobile.dc.html` and re-bundle to regenerate `index.html`.

## Notes on the maths

- Average on-hand is taken as `EOQ/2 + safety stock`, so holding cost includes the buffer.
- The z-score comes from a Hastings approximation of the inverse normal CDF (95% → 1.64).
- Simulated daily demand is normal around `annual_demand / 365` with `demand_std_dev`, clipped at zero; deliveries arrive `lead_time_days` after the trigger, and orders are placed off inventory *position* (on hand + on order) so it never double-orders while waiting.
