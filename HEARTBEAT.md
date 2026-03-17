# FlipCoin Market Monitor

Check for notable changes on FlipCoin prediction markets and notify the user if anything interesting happened.

## Checklist

1. **Portfolio check** — If the user has open positions (`GET /api/agent/portfolio?status=open`), check if any market's price moved significantly since last check (>10% change). Alert: "Your position in [market] moved — YES now at X% (was Y%)."

2. **Expiring markets** — Check for markets resolving within 24 hours (`GET /api/agent/markets/explore?status=open&sort=deadlineSoon&limit=5`). Alert: "[market] expires in X hours — your position: Z shares."

3. **Resolution results** — Check for newly resolved markets where the user had a position (`GET /api/agent/portfolio?status=resolved`). Alert: "Market resolved! [market] — outcome: YES. You won/lost $X."

4. **Hot markets** — If no portfolio alerts, mention the top trending market by volume as a conversation starter: "Trending on FlipCoin: [market title] — YES at X%, $Y volume."

## Rules

- Only alert if there is genuinely new or notable information
- If the user has no positions and nothing trending, reply with HEARTBEAT_OK
- Keep alerts short and conversational (this goes to Telegram/WhatsApp)
- Include actionable context: current price, user's position size, time remaining
- Never make trading recommendations — just present facts
