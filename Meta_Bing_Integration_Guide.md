# Meta Pixel & Microsoft Bing Ads (UET) — Cookie Consent Integration Guide

**Document Type:** Site Owner Integration Guide  
**Version:** 1.0  
**Date:** 2026-05-18  
**Status:** Draft — For Review

---

## 1. Overview

This guide covers everything a site owner must do to integrate Meta (Facebook) Pixel and Microsoft Bing Ads Universal Event Tracking (UET) with the Seqrite Cookie Consent Manager. It includes account setup, pixel/tag installation, and the code changes required on both the **baseline** (no enforcement) and **enforcement** branches of the test site.

**Enforcement note:** The consent signalling for Meta and Bing (i.e., blocking/unblocking cookies based on user choice) is handled on the `seqrite-enforcement` branch. The steps in this guide first cover account and tag setup, then the baseline cookie additions (already done on `main`), and finally the enforcement changes to apply on the enforcement branch.

---

## 2. Prerequisites

| Account | Purpose | URL |
|---|---|---|
| Meta Business Manager | Owns the Facebook Pixel | business.facebook.com |
| Microsoft Advertising | Owns the Bing Ads UET tag | ads.microsoft.com |
| Access to the site's codebase or GTM container | To deploy the pixel/tag | — |

---

## 3. Meta Pixel Setup

### 3.1 Create a Meta Business Account

1. Go to **business.facebook.com** and click **Create Account**.
2. Enter your business name, your name, and business email, then click **Submit**.
3. Verify your email address via the confirmation link sent by Meta.
4. Complete the Business Manager profile (business address, website, etc.).

### 3.2 Create a Facebook Pixel

1. Inside Meta Business Manager, go to **Events Manager** (left nav → All Tools → Events Manager).
2. Click **Connect data sources** → **Web** → **Get Started**.
3. Select **Meta Pixel** and click **Connect**.
4. Name the pixel (e.g., "Test Site Pixel") and enter your website URL.
5. Click **Create Pixel**. Note your **Pixel ID** (a 15-16 digit number, e.g., `123456789012345`).

### 3.3 Get the Pixel Base Code

In Events Manager → your pixel → **Set up** → **Install code manually**. You will see a base code snippet like this:

```html
<!-- Meta Pixel Code -->
<script>
  !function(f,b,e,v,n,t,s){
    if(f.fbq)return;n=f.fbq=function(){n.callMethod?
    n.callMethod.apply(n,arguments):n.queue.push(arguments)};
    if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
    n.queue=[];t=b.createElement(e);t.async=!0;
    t.src=v;s=b.getElementsByTagName(e)[0];
    s.parentNode.insertBefore(t,s)
  }(window,document,'script','https://connect.facebook.net/en_US/fbevents.js');
  fbq('init', 'YOUR_PIXEL_ID');
  fbq('track', 'PageView');
</script>
<noscript>
  <img height="1" width="1" style="display:none"
       src="https://www.facebook.com/tr?id=YOUR_PIXEL_ID&ev=PageView&noscript=1"/>
</noscript>
```

Replace `YOUR_PIXEL_ID` with the Pixel ID from step 3.2.

### 3.4 Cookies Set by Meta Pixel

| Cookie | Category | Purpose | Expiry |
|---|---|---|---|
| `_fbp` | Marketing | Browser fingerprint for retargeting audiences | 90 days |
| `_fbc` | Marketing | Click ID — set when a visitor arrives via a `fbclid` URL parameter | 90 days |

`_fbp` is always set on pixel load. `_fbc` is set only when `fbclid` appears in the URL (i.e., the visitor clicked a Meta ad).

---

## 4. Microsoft Bing Ads UET Setup

### 4.1 Create a Microsoft Advertising Account

1. Go to **ads.microsoft.com** and click **Sign up now**.
2. Sign in with a Microsoft account or create a new one.
3. Choose your country/region and currency, then complete the account wizard.
4. You do not need to add a payment method to create UET tags.

### 4.2 Create a UET Tag

1. In the Microsoft Advertising dashboard, go to **Tools** (top nav) → **UET tags**.
2. Click **Create UET tag**.
3. Name the tag (e.g., "Test Site UET") and click **Save**.
4. Note your **Tag ID** (a numeric ID, e.g., `12345678`).

### 4.3 Get the UET Base Code

After creating the tag, click **View tag** to see the installation snippet:

```html
<!-- Microsoft UET Tag -->
<script>
  (function(w,d,t,r,u){
    var f,n,i;
    w[u]=w[u]||[],f=function(){
      var o={ti:"YOUR_TAG_ID"};
      o.q=w[u],w[u]=new UET(o),w[u].push("pageLoad")
    },
    n=d.createElement(t),n.src=r,n.async=1,n.onload=n.onreadystatechange=function(){
      var s=this.readyState;
      s&&s!=="loaded"&&s!=="complete"||(f(),n.onreadystatechange=null)
    },
    i=d.getElementsByTagName(t)[0],i.parentNode.insertBefore(n,i)
  })(window,document,"script","//bat.bing.com/bat.js","uetq");
</script>
<noscript>
  <img src="//bat.bing.com/action/0?ti=YOUR_TAG_ID&Ver=2" height="0" width="0"
       style="display:none; visibility:hidden;" />
</noscript>
```

Replace `YOUR_TAG_ID` with the Tag ID from step 4.2.

### 4.4 Cookies Set by Bing Ads UET

| Cookie | Category | Purpose | Expiry |
|---|---|---|---|
| `_uetsid` | Marketing | UET session identifier | 30 minutes |
| `_uetvid` | Marketing | UET visitor identifier — persists across sessions | 180 days |
| `MUID` | Marketing | Microsoft User ID — shared across Microsoft properties | 1 year |

> **Important:** `_uetsid` and `_uetvid` are Bing Ads UET cookies, **not** Microsoft Clarity cookies. Clarity uses `_clck` and `_clsk`. Ensure they are mapped to the **Marketing** consent category in the Seqrite config, not Analytics.

---

## 5. Code Changes — Baseline (`main` branch)

These changes add Meta and Bing cookies to the test site so they appear in the browser and can be observed before enforcement is applied. No consent gating is applied at this stage.

### 5.1 `index.html` — Add cookie-setting scripts

Added to `<head>` alongside the existing marketing cookie scripts:

```html
<!-- Meta Pixel cookies: browser ID + click ID -->
<script>
  document.cookie = '_fbc=fb.1.' + Date.now() + '.IwAR0test123; path=/; max-age=7776000';
  console.log('Meta Pixel click-ID cookie set: _fbc');
</script>

<!-- Bing Ads UET cookies: session, visitor, Microsoft User ID -->
<script>
  document.cookie = '_uetsid=uet-sess-' + Math.random().toString(36).slice(2) + '; path=/; max-age=1800';
  document.cookie = '_uetvid=uet-vid-'  + Math.random().toString(36).slice(2) + '; path=/; max-age=15552000';
  document.cookie = 'MUID=MUID-' + Math.random().toString(36).slice(2).toUpperCase() + '; path=/; max-age=31536000';
  console.log('Bing Ads UET cookies set: _uetsid, _uetvid, MUID');
</script>
```

`_fbp` was already present from the earlier marketing cookie additions.

---

## 6. Code Changes — Enforcement (`seqrite-enforcement` branch)

> Apply these changes on the `seqrite-enforcement` branch only. Do not apply to `main`.

### 6.1 `index.html` — Block Meta and Bing scripts until consent

Wrap the Meta Pixel and Bing UET script tags with `type="text/plain"` and `data-name="marketing"` so Klaro blocks them until marketing consent is granted:

```html
<!-- Meta Pixel — blocked until "marketing" consent granted -->
<script type="text/plain" data-type="text/javascript" data-name="marketing">
  !function(f,b,e,v,n,t,s){
    if(f.fbq)return;n=f.fbq=function(){n.callMethod?
    n.callMethod.apply(n,arguments):n.queue.push(arguments)};
    if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
    n.queue=[];t=b.createElement(e);t.async=!0;
    t.src=v;s=b.getElementsByTagName(e)[0];
    s.parentNode.insertBefore(t,s)
  }(window,document,'script','https://connect.facebook.net/en_US/fbevents.js');
  fbq('init', 'YOUR_PIXEL_ID');
  fbq('track', 'PageView');
</script>

<!-- Bing Ads UET — blocked until "marketing" consent granted -->
<script type="text/plain" data-type="text/javascript" data-name="marketing"
        data-src="//bat.bing.com/bat.js">
</script>
<script type="text/plain" data-type="text/javascript" data-name="marketing">
  window.uetq = window.uetq || [];
  window.uetq.push('pageLoad');
</script>
```

### 6.2 `seqrite-enforcement.js` — Add Meta and Bing consent default + update functions

Add the following at the **top** of `seqrite-enforcement.js`, before the `PrivacyConsent` wrapper, so defaults are set before any pixel loads:

```javascript
// ── Meta Pixel Consent Mode ─────────────────────────────────────────────
// Create fbq stub and immediately revoke consent (LDU mode default).
// When fbevents.js loads it reads this queue and starts in Limited Data Use mode.
window.fbq = window.fbq || function() {
  (window.fbq.q = window.fbq.q || []).push(arguments);
};
window.fbq('consent', 'revoke');

// ── Microsoft Bing Ads (UET) Consent Mode ──────────────────────────────
// Initialise uetq queue and push consent-denied default before bat.js loads.
window.uetq = window.uetq || [];
window.uetq.push('set', { convConsentMode: false });
```

Then add the two update functions inside the `PrivacyConsent` IIFE, alongside the existing `updateGoogleConsentMode`:

```javascript
// ── updateMetaConsentMode ─────────────────────────────────────────────
function updateMetaConsentMode(consents) {
  if (typeof window.fbq !== 'function') return;
  window.fbq('consent', consents['marketing'] === true ? 'grant' : 'revoke');
}

// ── updateMicrosoftConsentMode ────────────────────────────────────────
function updateMicrosoftConsentMode(consents) {
  window.uetq = window.uetq || [];
  window.uetq.push('set', { convConsentMode: consents['marketing'] === true });
}
```

Call both from the existing `watch` callback alongside `updateGoogleConsentMode`:

```javascript
// Inside the update: function(obj, name, data) { ... } callback:
updateGoogleConsentMode(data);
updateMetaConsentMode(data);       // add this line
updateMicrosoftConsentMode(data);  // add this line
```

Export both from the return object:

```javascript
return {
  // ... existing exports ...
  updateMetaConsentMode:      updateMetaConsentMode,
  updateMicrosoftConsentMode: updateMicrosoftConsentMode
};
```

### 6.3 `seqrite-config.js` — Add `_fbc`, `_uetsid`, `_uetvid`, `MUID` to marketing service

In the `services` array, update the `marketing` entry's `cookies` array:

```javascript
{
  "name": "marketing",
  "cookies": [
    "_fbp",
    "_fbc",        // add
    "_gcl_au",
    "_uetsid",     // add
    "_uetvid",     // add
    "MUID"         // add
  ]
}
```

This ensures Klaro deletes all Meta and Bing cookies when the user rejects marketing consent.

---

## 7. GTM Implementation (Alternative to Direct Embed)

If the site loads Meta Pixel or Bing UET via Google Tag Manager instead of direct embed, no changes to the pixel/tag code are needed. Use the `dataLayer` events that `seqrite-enforcement.js` already pushes:

| Event Name | Use For |
|---|---|
| `privacyconsent-marketing-accepted` | Fire the Meta Pixel tag / Bing UET tag |
| `privacyconsent-marketing-rejected` | Block / pause the Meta Pixel tag / Bing UET tag |

In GTM: create two **Custom Event triggers** on the above event names and assign them as fire/block conditions on the respective tags.

---

## 8. Validation Checklist

| # | Check | How to Verify |
|---|---|---|
| 1 | **Meta — Default denied** | On page load, open browser console and run `fbq.q`. It should contain `["consent", "revoke"]` before any user interaction. `connect.facebook.net` must not appear in the Network tab. |
| 2 | **Meta — Grant on accept** | Accept marketing consent. Run `fbq.q` again — should contain `["consent", "grant"]`. Verify with the **Meta Pixel Helper** Chrome extension. |
| 3 | **Meta — Revoke on decline** | Decline marketing. `fbq.q` should show `["consent", "revoke"]`. |
| 4 | **Meta cookies visible** | In DevTools → Application → Cookies, both `_fbp` and `_fbc` should be present on the baseline site (`main`). |
| 5 | **Bing — Default denied** | On page load, run `window.uetq` in console. Should contain `{ convConsentMode: false }`. |
| 6 | **Bing — Signal on consent change** | Accept/decline marketing. `window.uetq` should update to `convConsentMode: true` or `false`. |
| 7 | **Bing cookies visible** | `_uetsid`, `_uetvid`, and `MUID` present in DevTools → Application → Cookies on the baseline site. |
| 8 | **Cookie categorisation** | In the consent banner (enforcement branch), `_uetsid` and `_uetvid` must appear under **Marketing**, not Analytics. |
| 9 | **Cookies deleted on reject** | After rejecting marketing consent (enforcement branch), `_fbp`, `_fbc`, `_uetsid`, `_uetvid`, `MUID` should all be removed from the browser. |
| 10 | **GTM dataLayer events** | Run `window.dataLayer` in console after a consent interaction. Should contain `privacyconsent-marketing-accepted` or `privacyconsent-marketing-rejected`. |

---

## 9. Summary of Changes by Branch

| File | `main` (baseline) | `seqrite-enforcement` |
|---|---|---|
| `index.html` | Add `_fbc`, `_uetsid`, `_uetvid`, `MUID` cookie scripts (plain, no gating) | Wrap Meta Pixel and Bing UET scripts with `type="text/plain" data-name="marketing"` |
| `seqrite-enforcement.js` | Not present | Add `fbq` stub + `uetq` default at top; add `updateMetaConsentMode()`, `updateMicrosoftConsentMode()`; call both from watch callback |
| `seqrite-config.js` | Not present | Add `_fbc`, `_uetsid`, `_uetvid`, `MUID` to `marketing.cookies` array |

---

*Seqrite Cookie Consent Manager · Site Owner Integration Guide · v1.0 · 2026-05-18*
