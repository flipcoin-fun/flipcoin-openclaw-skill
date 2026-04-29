# FlipCoin Market Monitor

Check for notable changes on FlipCoin prediction markets and notify the user if anything interesting happened.

## How to detect changes (canonical)

Use FlipCoin's **change feed** — it's cursor-based, idempotent, and far cheaper than polling portfolio + market state on every tick.

**Primary — `GET /api/agent/feed`:**

```bash
curl -s "$BASE/feed?since=$LAST_CURSOR&types=trade,market_resolved,resolution_proposed,resolution_disputed&limit=50" \
  -H "Authorization: Bearer $FLIPCOIN_API_KEY" | jq .
```

- `since` is an ISO 8601 timestamp; the response includes a `cursor` field (also ISO) — persist it and pass it back as `since` next tick.
- `hasMore: true` means paginate further with the same call before treating the tick as drained.
- On the very first tick, use a recent timestamp (e.g. now − 5 min) instead of an empty `since`.
- Cache: `s-maxage=10` — repeated polling within 10s is free.

**Streaming alternative — `GET /api/agent/feed/stream`** (Server-Sent Events, Bearer auth):

```
GET $BASE/feed/stream?channels=orderbook:0xCONDITION,trades:0xCONDITION,prices
```

Use this for low-latency monitoring (e.g. if the user opened a position and asked you to watch it). The stream auto-reconnects every 5 minutes via the `reconnect` event — resume with `Last-Event-ID`.

**Fallback (only if feed is unreachable)**: poll `GET /api/agent/portfolio?status=open` and `GET /api/agent/markets/explore?sort=deadlineSoon`. The feed is the source of truth — only use polling as a degraded-mode fallback.

## What to monitor

- **`type=trade`** — A trade hit a market the user holds. Alert if price moved significantly (>10%) since last seen.
- **`type=resolution_proposed`** — A market the user has a position in entered the 24h dispute window. Tell them which side was proposed and `finalizeAfter` time.
- **`type=resolution_disputed`** — Their proposal (if they're the creator) was disputed by a shareholder. Action: re-investigate.
- **`type=market_resolved`** — Final outcome — trigger the redeem flow if they have winning shares.

Also worth a heads-up even without feed events:
- Markets in their portfolio that resolve within 24h (`GET /portfolio?status=open` + check `resolveEndAt`).
- A trending hot market by volume, as a conversation starter when there's nothing personal to report.

## Rules

- Only alert if there is genuinely new or notable information.
- If nothing in the feed is relevant and the user has no positions, reply with `HEARTBEAT_OK`.
- Keep alerts short and conversational (this goes to Telegram/WhatsApp).
- Include actionable context: current price, position size, time remaining, txHash if relevant.
- Never make trading recommendations — just present facts.
- Persist `cursor` between ticks. Don't re-process events already seen.
