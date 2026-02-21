type-safe Deno client for the Serper.dev Google Search API. ten endpoints, zero dependencies, strict TypeScript throughout.

```ts
import { SerperClient } from "jsr:@yigitkonur/serper-deno-sdk";

const client = new SerperClient({ apiKey: Deno.env.get("SERPER_API_KEY")! });
const results = await client.search("your query");
```

[![JSR](https://jsr.io/badges/@yigitkonur/serper-deno-sdk)](https://jsr.io/@yigitkonur/serper-deno-sdk)
[![deno](https://img.shields.io/badge/deno-2.x-93450a.svg?style=flat-square)](https://deno.land/)
[![license](https://img.shields.io/badge/license-MIT-grey.svg?style=flat-square)](https://opensource.org/licenses/MIT)

---

## what it does

- **10 search endpoints** — web, images, news, videos, shopping, maps, reviews, scholar, patents, autocomplete
- **zero runtime dependencies** — only web standard APIs (`fetch`, `AbortController`)
- **strict types** — readonly result arrays, `noUncheckedIndexedAccess`, true private fields via `#`
- **typed error hierarchy** — auth, rate limit, validation, server errors with proper `instanceof` chains
- **timeout + abort** — configurable per-client, defaults to 30s
- **default merging** — set country/language once on the client, override per-request

## install

```bash
deno add jsr:@yigitkonur/serper-deno-sdk
```

or import directly without install:

```ts
import { SerperClient } from "jsr:@yigitkonur/serper-deno-sdk@1.0.2";
```

## usage

### web search

```ts
const results = await client.search("rust async runtime", {
  num: 5,
  gl: "us",
  tbs: "qdr:w", // past week
});
```

### images

```ts
const images = await client.searchImages("aurora borealis", { safe: "active" });
```

### news / videos / shopping

```ts
const news = await client.searchNews("deno 2.0");
const videos = await client.searchVideos("systems programming");
const shopping = await client.searchShopping("mechanical keyboard");
```

### maps + reviews

```ts
const places = await client.searchMaps("coffee shops", { location: "Brooklyn, NY" });
const reviews = await client.getReviews({ placeId: "ChIJ..." });
```

### scholar + patents

```ts
const papers = await client.searchScholar("transformer architecture", {
  as_ylo: 2023, // published after 2023
});
const patents = await client.searchPatents("solid state battery");
```

### autocomplete

```ts
const suggestions = await client.autocomplete("how to");
```

## endpoints

| method | endpoint | description |
|:---|:---|:---|
| `search` | `/search` | web search (organic, knowledge graph, answer box, people also ask) |
| `searchImages` | `/images` | image results with dimensions and source |
| `searchNews` | `/news` | news articles with source and date |
| `searchVideos` | `/videos` | video results (YouTube, etc.) |
| `searchShopping` | `/shopping` | products with price and seller |
| `searchMaps` | `/maps` | local businesses with lat/lng, rating, phone |
| `searchPlaces` | `/maps` | alias for `searchMaps` |
| `getReviews` | `/reviews` | place reviews with ratings and pagination |
| `searchScholar` | `/scholar` | academic papers with optional PDF links |
| `searchPatents` | `/patents` | patent filings with abstract and publication date |
| `autocomplete` | `/autocomplete` | search suggestions |

## client options

```ts
const client = new SerperClient({
  apiKey: "...",              // required
  baseUrl: "...",             // default: https://google.serper.dev
  defaultCountry: "us",      // applied as `gl` to all requests
  defaultLanguage: "en",     // applied as `hl` to all requests
  timeout: 15_000,           // ms, default: 30_000
});
```

per-request options override client defaults.

## error handling

```ts
import {
  SerperAuthError,
  SerperRateLimitError,
  SerperValidationError,
  SerperServerError,
} from "jsr:@yigitkonur/serper-deno-sdk";

try {
  await client.search("query");
} catch (e) {
  if (e instanceof SerperRateLimitError) {
    // 429 — back off
  } else if (e instanceof SerperAuthError) {
    // 401/403 — bad key
  } else if (e instanceof SerperValidationError) {
    // 4xx or empty query
  } else if (e instanceof SerperServerError) {
    // 5xx, timeout, network
  }
}
```

all errors extend `SerperError` which extends `Error`. each carries a `.status` number.

## supabase edge functions

the repo includes 10 ready-to-deploy Supabase Edge Functions in `supabase/functions/`, one per endpoint. import map points to JSR:

```json
{ "imports": { "@yigitkonur/serper-deno-sdk": "jsr:@yigitkonur/serper-deno-sdk@1.0.2" } }
```

```bash
supabase functions deploy serper-web-search
```

## development

```bash
deno task all         # fmt + lint + check + test
deno task test        # run tests
deno task bump:patch  # bump version, commit, tag
```

## license

MIT
