# Iterant Cloudflare Proxy Worker

A Cloudflare Worker that proxies requests from your domain to the Iterant platform, enabling seamless integration of Iterant-generated landing pages on your website.

## One-Click Deploy

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/IterantAI/cloudflare-proxy-worker)

## Features

- Edge-based reverse proxy with global distribution
- Built-in caching with configurable TTL
- Secure tenant identification headers
- Zero runtime dependencies
- TypeScript with strict mode

## Configuration

| Variable       | Required | Default            | Description                              |
| -------------- | -------- | ------------------ | ---------------------------------------- |
| `BRAND_ID`     | Yes      | -                  | Your Iterant Brand ID (set as secret)    |
| `TARGET_HOST`  | No       | `sites.iterant.ai` | Iterant platform host                    |
| `ROUTING_PATH` | No       | `/marketing`       | URL path prefix to proxy                 |
| `CACHE_TTL`    | No       | `3600`             | Cache duration in seconds (0 to disable) |

## Post-Deployment Setup

After deploying the worker, you need to configure a route:

1. Go to your [Cloudflare dashboard](https://dash.cloudflare.com)
2. Select your domain
3. Navigate to **Workers Routes** (under Workers & Pages)
4. Click **Add Route**
5. Enter route pattern: `yourdomain.com/marketing/*`
6. Select the deployed worker (`iterant-proxy`)
7. Click **Save**

> **Note:** Replace `/marketing` with your configured `ROUTING_PATH` value.

## Local Development

```bash
# Install dependencies
npm install

# Copy environment template
cp .dev.vars.example .dev.vars

# Edit .dev.vars with your BRAND_ID
# BRAND_ID=brnd_your_actual_brand_id

# Start development server
npm run dev
```

The worker will be available at `http://localhost:8787`.

## Manual Deployment

```bash
# Login to Cloudflare
wrangler login

# Set your Brand ID as a secret
wrangler secret put BRAND_ID

# Deploy the worker
npm run deploy
```

## How It Works

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Visitor       │     │  Cloudflare     │     │    Iterant      │
│                 │     │  Worker         │     │    Platform     │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         │  GET /marketing/page  │                       │
         │ ─────────────────────>│                       │
         │                       │                       │
         │                       │  GET /marketing/page  │
         │                       │  + Tenant Headers     │
         │                       │ ─────────────────────>│
         │                       │                       │
         │                       │      Response         │
         │                       │ <─────────────────────│
         │                       │                       │
         │     Response          │  (cached at edge)     │
         │ <─────────────────────│                       │
         │                       │                       │
```

1. Requests to `yourdomain.com/marketing/*` are intercepted by the worker
2. The worker adds tenant identification headers
3. Request is proxied to Iterant platform
4. Response is cached at the edge (if enabled)
5. Cached or fresh response is returned to the visitor

## Headers Added

| Header              | Value             | Purpose                   |
| ------------------- | ----------------- | ------------------------- |
| `X-Tenant-Brand-ID` | Your Brand ID     | Identifies your brand     |
| `X-Forwarded-Host`  | Original hostname | Preserves original domain |
| `X-Forwarded-Proto` | `https`           | Preserves protocol        |
| `X-Original-Path`   | Original path     | Preserves full path       |

## Troubleshooting

### "Configuration error: BRAND_ID not set"

Make sure you've set the BRAND_ID secret:

```bash
wrangler secret put BRAND_ID
```

Or when using Deploy to Cloudflare, enter your Brand ID in the deployment form.

### Pages not loading

1. Verify the route pattern matches your configuration
2. Check that `ROUTING_PATH` matches your route pattern
3. Verify your domain is verified in the [Iterant Dashboard](https://app.iterant.ai)
4. Check the worker logs in Cloudflare dashboard

### Caching issues

- Set `CACHE_TTL=0` to disable caching for debugging
- Check `X-Cache-Status` response header (`HIT` = cached, `MISS` = fresh)
- Clear cache by purging in Cloudflare dashboard

### CORS errors

The worker preserves CORS headers from the origin. If you're seeing CORS errors, ensure your Iterant content is configured correctly for your domain.

## Development

```bash
# Run linting
npm run lint

# Run TypeScript check
npm run typecheck

# Format code
npm run format
```

## Environment Variables Reference

### `BRAND_ID` (Required)

Your unique brand identifier from Iterant. Find it in your [Iterant Dashboard](https://app.iterant.ai) under **Settings > Domains**.

Format: `brnd_xxxxxxxxxxxx`

### `TARGET_HOST`

The Iterant platform hostname to proxy requests to.

Default: `sites.iterant.ai`

> Only change this if instructed by Iterant support.

### `ROUTING_PATH`

The URL path prefix that triggers the proxy. All requests starting with this path will be proxied to Iterant.

Default: `/marketing`

Examples:

- `/marketing` - Proxy `yourdomain.com/marketing/*`
- `/lp` - Proxy `yourdomain.com/lp/*`
- `/pages` - Proxy `yourdomain.com/pages/*`

### `CACHE_TTL`

Cache duration in seconds for successful responses.

Default: `3600` (1 hour)

Set to `0` to disable caching entirely.

## Support

- [Iterant Documentation](https://docs.iterant.ai)
- [GitHub Issues](https://github.com/IterantAI/cloudflare-proxy-worker/issues)

## License

MIT
