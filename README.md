# Pepesto Grocery Shopping API

API specification for the [Pepesto Grocery API](https://www.pepesto.com/ai-grocery-shopping-agent/) — turn any recipe (a URL, plain text, or a photo) into a matched basket of real supermarket products with live prices, across **28 European supermarkets in 13 countries**. Pepesto covers the ingredients people actually cook with and the workflows agents, retail-intelligence platforms, and developers actually reach for. The same surface is also exposed as an MCP server for agents — see [pepesto-mcp](https://github.com/pepesto-solutions/pepesto-mcp).

The API covers all 14 public endpoints (`/link`, `/credits`, `/oneshot`, `/predirect`, `/parse`, `/suggest`, `/products`, `/search`, `/retrieve`, `/session`, `/checkout`, `/mcheckout`, `/catalog`, `/promotions`) with request/response schemas, examples drawn from the [api-examples](https://github.com/pepesto-solutions/api-examples) repo, and the full enum of supported supermarket domains.

> _European grocery data infrastructure. 28 supermarkets, 13 countries, one schema._

## Quick start

### Get an API key

1. Subscribe to a plan — see <https://www.pepesto.com/pricing/>.
2. Mint an API key by calling `/link` with the email you used at checkout. The key is returned **only once** — store it immediately.

   ```bash
   curl -X POST https://s.pepesto.com/api/link \
     -H "Content-Type: application/json" \
     -d '{"email":"you@example.com"}'
   ```

3. Set the key in your environment:

   ```bash
   export PEPESTO_API_KEY=pep_sk_…
   ```

### Make your first call

```bash
curl -X POST https://s.pepesto.com/api/oneshot \
  -H "Authorization: Bearer $PEPESTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content_urls": ["https://www.bbcgoodfood.com/recipes/pizza-margherita-4-easy-steps"],
    "content_text": "also add sparkling water and olive oil",
    "supermarket_domain": "tesco.com"
  }'
```

You'll get back a `redirect_url` that opens the Pepesto checkout UI with the matched cart pre-filled.

## Endpoints

All endpoints live under `https://s.pepesto.com/api`. All require `Authorization: Bearer pep_sk_…` except `POST /link`.

| Endpoint | Tag | Description |
| --- | --- | --- |
| `POST /link`        | Account  | Mint an API key linked to a Stripe payment. Returned **once**. Free. |
| `POST /credits`     | Account  | Check remaining credit balance. Free. |
| `POST /oneshot`     | Cart     | One-shot recipe → checkout-ready cart. Returns a `redirect_url`. |
| `POST /predirect`   | Cart     | Hand a shopping list to the Pepesto mobile app as a deferred deep link. No API key needed, and free to you — the end user pays at checkout. |
| `POST /parse`       | Recipes  | Parse a URL/text/image recipe into structured ingredients + `kg_token`. |
| `POST /suggest`     | Recipes  | Search Pepesto's 1M+ recipe knowledge graph. Returns 3 recipes + `kg_token`s. |
| `POST /products`    | Cart     | Map `kg_token`s + supermarket to ranked product matches with prices. |
| `POST /search`      | Cart     | Start an asynchronous search of the whole store, beyond the cooking-ingredient cache. Returns a `search_session_id`. |
| `POST /retrieve`    | Cart     | Check a `/search` and collect its results once the status is `done`. Free. |
| `POST /session`     | Checkout | Create a checkout session from `/products` results or free-text items. |
| `POST /checkout`    | Checkout | Drive one iteration of the automated-checkout loop on the supermarket's site. |
| `POST /mcheckout`   | Checkout | Hand the whole checkout loop to the Pepesto app. Returns a deep link that fills the basket and sends the user back to you. |
| `POST /catalog`     | Catalog  | Full SKU dump for a supermarket (1000-2000 SKUs, focused on the most common ingredients used for cooking). Heaviest call — cache the result. |
| `POST /promotions`  | Catalog  | Promotional SKUs for a supermarket. |

The full request/response shapes live in [`openapi.yaml`](./openapi.yaml).

## Common shapes

- **Prices** are integers in the smallest currency unit (cents, pence, øre, …).
- **`kg_token`** is an opaque recipe handle returned by `/parse` and `/suggest` that you pass to `/products` to map ingredients to real SKUs.
- **`session_token`** is a per-product handle from `/products` that you pass to `/session` or `/mcheckout`.
- **Currencies**: EUR, GBP, CHF, NOK, PLN, BGN, DKK, SEK.

## Example flows

### Recipe URL → matched cart (one call)

The fastest path. `/oneshot` parses, matches, builds a session, and returns a checkout URL.

```bash
curl -X POST https://s.pepesto.com/api/oneshot \
  -H "Authorization: Bearer $PEPESTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content_urls": ["https://www.bbcgoodfood.com/recipes/pizza-margherita-4-easy-steps"],
    "supermarket_domain": "tesco.com"
  }'
```

### Browse the recipe graph → pick → match

For "find me something to cook" prompts.

1. `POST /suggest` with a query like `"vegan pasta dinner for two, ready in 20 minutes"` — returns 3 recipes, each with a `kg_token`.
2. `POST /products` with the chosen `kg_tokens` and a `supermarket_domain` — returns matched products with prices.

### Parse a specific recipe → match later

When the user already has a recipe URL or text in mind.

1. `POST /parse` with `recipe_url` or `recipe_text` — returns structured ingredients + `kg_token`.
2. `POST /products` with that `kg_token` and a `supermarket_domain`.

### Weekly meal plan → one merged cart

Pass multiple `kg_tokens` to a single `POST /products` call — Pepesto merges duplicated ingredients across recipes so the user buys 1kg of pasta, not 5.

### Compare prices across supermarkets

Run the same `kg_token` through `POST /products` once per `supermarket_domain` and compare the totals.

### Pull promotion items across supermarkets

Pull only the promotion items with `POST /promotions` once per `supermarket_domain` and share with the world.

### Catalog dump for market analysis

`POST /catalog` is the heaviest call — right for genuine market analysis or price-comparison dashboards. **Cache the result for at least a day per supermarket.** Not sure you need it? [Tell us about your use case](https://www.pepesto.com/contact) and we'll usually point you to a cheaper path.

## Supported supermarkets

| # | Country | Supermarket | Domain / ID |
| --- | --- | --- | --- |
| 1 | 🇬🇧 GB | Tesco | tesco.com |
| 2 | 🇬🇧 GB | Sainsbury's | sainsburys.co.uk |
| 3 | 🇬🇧 GB | Asda | asda.com |
| 4 | 🇬🇧 GB | Morrisons | groceries.morrisons.com |
| 5 | 🇬🇧 GB | Waitrose | waitrose.com |
| 6 | 🇬🇧 GB | Ocado | ocado.com |
| 7 | 🇳🇱 NL | Albert Heijn | ah.nl |
| 8 | 🇳🇱 NL | Jumbo | jumbo.com |
| 9 | 🇳🇱 NL | Plus | plus.nl |
| 10 | 🇩🇪 DE | Rewe | shop.rewe.de |
| 11 | 🇨🇭 CH | Coop Switzerland | coop.ch |
| 12 | 🇨🇭 CH | Migros | migros.ch |
| 13 | 🇨🇭 CH | Aldi Switzerland | aldi-now.ch |
| 14 | 🇧🇪 BE | Colruyt | colruyt.be |
| 15 | 🇧🇪 BE | Delhaize | delhaize.be |
| 16 | 🇧🇪 BE | Albert Heijn Belgium | ah.be |
| 17 | 🇮🇪 IE | Tesco Ireland | tesco.ie |
| 18 | 🇮🇪 IE | SuperValu Ireland | shop.supervalu.ie |
| 19 | 🇮🇪 IE | Dunnes Stores | dunnesstoresgrocery.com |
| 20 | 🇮🇹 IT | Esselunga | spesaonline.esselunga.it |
| 21 | 🇮🇹 IT | Conad | spesaonline.conad.it |
| 22 | 🇩🇰 DK | Nemlig | nemlig.com |
| 23 | 🇳🇴 NO | Meny | meny.no |
| 24 | 🇵🇱 PL | Frisco | frisco.pl |
| 25 | 🇵🇱 PL | Auchan Poland | zakupy.auchan.pl |
| 26 | 🇧🇬 BG | eBag | ebag.bg |
| 27 | 🇸🇪 SE | ICA Sweden | handlaprivatkund.ica.se |
| 28 | 🇵🇹 PT | Continente | continente.pt |

The authoritative list lives in the `SupermarketDomain` enum in [`openapi.yaml`](./openapi.yaml). Need a supermarket that isn't on this list? [Contact Pepesto](https://www.pepesto.com/contact).

## Where checkout actually happens

Three ways to finish the trip after `/products` has produced a matched basket:

- **Hosted checkout (recommended).** Open the `redirect_url` from `/oneshot` (or build one with `/session`) — that's where the Pepesto-hosted flow lives, including login, basket review, and (for some markets) payment.
- **Automated checkout on the supermarket's own site.** `/session` + `/checkout` form a turn-by-turn browser-automation loop that drives `tesco.com` / `coop.ch` / etc. directly (login, add-to-basket, prompt-for-CAPTCHA, etc.). This is the path for fully autonomous shopping agents.
- **Checkout in the Pepesto app.** `/mcheckout` returns a deep link that hands the loop to the Pepesto mobile app, which fills the basket and sends the user back to your `redirect_url`. The app stays locked to checkout, so your user stays your user.

## Pricing & best practices

Pepesto runs on a monthly subscription. Each payment becomes credits on your API key, every call draws those credits down at that endpoint's rate, and **unused credits roll over and never expire**. New builds can qualify for a discount on the first purchase, so [say hi](https://www.pepesto.com/contact) if that sounds like you. Full per-call pricing and volume tiers live at <https://www.pepesto.com/pricing/>.

A few tips to get the most out of every credit:

- `/link`, `/credits`, `/checkout`, `/retrieve` and `/predirect` cost nothing — call `/credits` any time for a quick balance read-out.
- `/oneshot`, `/parse`, `/suggest`, and `/products` are the everyday calls and are priced for routine agent use.
- `/products` reads the cooking-ingredient cache and answers immediately; reach for `/search` only when you need something outside it, because it is asynchronous and costs more.
- `/catalog` is the heaviest call — cache the result for at least a day per supermarket.

## Reference implementations

- **MCP server** — give an AI agent these endpoints as tools: <https://github.com/pepesto-solutions/pepesto-mcp>
- **Code examples**, one per use case: <https://github.com/pepesto-solutions/api-examples>
- This [README.md](https://github.com/pepesto-solutions/openapi-spec/blob/main/README.md) and the [`openapi.yaml`](https://github.com/pepesto-solutions/openapi-spec/blob/main/openapi.yaml) spec live in the [openapi-spec](https://github.com/pepesto-solutions/openapi-spec) repository.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for how to validate the spec locally, preview the docs in Swagger UI, and bundle to JSON or HTML.

## Support

[Contact Pepesto](https://www.pepesto.com/contact) — for missing supermarkets, custom integrations, volume pricing, or anything else.
