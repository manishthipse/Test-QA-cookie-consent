# Cookie Consent Test Site

Baseline test site for cookie consent enforcement development.

## Pages

- `index.html` — direct HTML/JS scripts + tracking pixel
- `gtm-page.html` — GTM container scenario
- `spa-page.html` — dynamic script loading scenario
- `iframe-page.html` — YouTube + Google Maps iframes
- `privacy-policy.html` — preferences link test

## Local dev

```
npx serve .
```

Then open http://localhost:3000

## Deployment

Every push to `main` auto-deploys to Netlify.

## WARNING

This site has NO consent enforcement by default.
All tracking scripts load freely. Enforcement is added separately.
