# gaon-app-config

Tiny public repo whose only job is serving each Gaon app's runtime config
(currently just `apiBaseUrl`) via [jsDelivr](https://www.jsdelivr.com/)'s free
CDN — a replacement for Firebase Remote Config's "domain kill-switch" use
case, without the platform-channel crash bug Remote Config had.

## Why this exists

Each app (customer, vendor, driver) needs to know which backend host to talk
to, and that host must be changeable **without shipping a new app release**
if the current domain ever goes down or migrates. Previously this used
Firebase Remote Config; this repo + jsDelivr does the same job with:

- No SDK, no native platform channel — a plain HTTPS GET, so it can't crash
  the way the Remote Config plugin did on hot-restart.
- No Firebase project dependency for this one thing.
- jsDelivr fronts this with Cloudflare **and** Fastly **and** its own edge
  network combined, not a single-CDN dependency — chosen specifically over a
  Cloudflare-only static host due to reported connectivity issues on some
  Indian ISPs.

## URLs (after this repo is pushed to GitHub as `<your-org>/gaon-app-config`)

Replace `<owner>` below with your actual GitHub username/org once pushed:

```
https://cdn.jsdelivr.net/gh/<owner>/gaon-app-config@main/config.json
https://cdn.jsdelivr.net/gh/<owner>/gaon-app-config@main/driver-config.json
```

(Add `vendor-config.json` / `customer-config.json` the same way if/when those
apps are migrated too — see `gaon_driver/lib/services/bootstrap_config_service.dart`
for the Flutter-side consumer.)

## Updating the URL

1. Edit the relevant `.json` file in this repo.
2. Commit + push to `main`.
3. jsDelivr caches files for ~7 days by default. To force it to pick up the
   change immediately instead of waiting out the cache, either:
   - Use the **purge API**: `POST https://purge.jsdelivr.net/gh/<owner>/gaon-app-config@main/config.json`
     (see https://www.jsdelivr.com/tools/purge) — do this right after every push.
   - Or reference a **tagged version** instead of `@main` (e.g. `@v2`) and cut
     a new tag each time — bypasses the cache entirely since each tag is a new
     immutable URL. Cleaner for infrequent changes; the `@main` + purge
     approach is simpler for occasional quick fixes.

## Format

Each file is flat JSON, one key so far:

```json
{ "apiBaseUrl": "https://your-real-backend.example.com/api" }
```

Keep it to primitive config values only (strings/booleans/numbers) — this is
a kill-switch, not a general remote-config system.
