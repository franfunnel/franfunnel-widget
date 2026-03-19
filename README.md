# FranFunnel Zoho CRM Widget

This repository hosts the files for the FranFunnel Conversation widget embedded inside Zoho CRM. The widget loads the FranFunnel conversation view for a contact directly inside the Zoho CRM contact record.

## How It Works

1. When a Zoho CRM user opens a Contact record, the widget initializes via the Zoho Embedded App SDK
2. The widget reads the contact's `FranFunnel_ID` field from the Zoho CRM record
3. It constructs the URL `https://app.franfunnel.com/app/lead/{FranFunnel_ID}` and loads it in an iframe

## Current Status

The Zoho integration is fully working — the widget loads, reads the correct FranFunnel ID from the CRM record, and attempts to display the conversation view. 

**The remaining issue** is on the FranFunnel app side: when loaded inside an iframe, the app redirects to the login screen even when the user is already authenticated. This is a backend fix needed on the FranFunnel server.

## Required Backend Fixes

### 1. Allow iframe Embedding

FranFunnel's server is currently blocking iframe embedding. Update the HTTP response headers on `app.franfunnel.com` to allow Zoho CRM to embed the app:

**Remove or update `X-Frame-Options`:**
```
# Remove this header entirely, or change to ALLOWFROM:
X-Frame-Options: ALLOWFROM https://crm.zoho.com
```

**Or use Content-Security-Policy (preferred modern approach):**
```
Content-Security-Policy: frame-ancestors 'self' https://crm.zoho.com https://*.zoho.com
```

### 2. Fix Session Cookies for Cross-Origin iframe

When FranFunnel is loaded inside a Zoho CRM iframe, the browser treats it as a third-party context. Session/auth cookies will be blocked unless they are explicitly configured for cross-site use.

Update all FranFunnel authentication cookies to include:
```
SameSite=None; Secure
```

For example, if using Express.js:
```javascript
res.cookie('session', value, {
  sameSite: 'none',
  secure: true,
  httpOnly: true
});
```

Without this, the browser will silently drop the auth cookie when FranFunnel is loaded in the iframe, causing the app to treat the user as unauthenticated and redirect to the login screen.

### 3. Zoho's Trusted Domain

`https://espielman23.github.io` has been added as a Trusted Domain in the Zoho CRM security settings to allow the widget to make CRM API calls.

## Files

- `index.html` — The widget UI. Initializes the Zoho SDK, fetches the FranFunnel ID from the CRM record, and renders the iframe.
- `ZohoEmbededAppSDK.min.js` — Zoho's Embedded App SDK (note: one 'd' in "Embeded" — this matches Zoho's own filename).

## Zoho Widget Configuration

| Setting | Value |
|---|---|
| Type | Related List |
| Hosting | External |
| Base URL | https://espielman23.github.io/franfunnel-widget/ |
| Entity | Contacts |
| FranFunnel ID Field | `FranFunnel_ID` |
