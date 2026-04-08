---
'@clerk/shared': patch
---

Fixed `isAllowedRedirect` to allow redirect URLs with non-default ports when the origin matches an `allowedRedirectOrigins` pattern.

**The bug:** String-derived glob patterns such as `https://*.example.com` failed to match origins that include a non-default port (e.g. `https://sub.example.com:5173`). `glob-to-regexp` anchors the generated regex to the end of the string, so the `:5173` suffix causes the match to fail.

This produced the warning:

```
Clerk: Redirect URL https://sub.example.com:5173 is not on one of the
allowedRedirectOrigins, falling back to the default redirect URL.
```

In practice, the Account Portal would fall back to the configured Home URL instead of honouring the `redirect_url` parameter — breaking the sign-in flow for any local dev setup that uses custom domains with non-standard ports (e.g. Vite dev servers on `:5173`, `:5174`, `:5176`).

**The fix:** When a redirect URL has an explicit non-default port and the full-origin match fails, `isAllowedRedirect` now also tests each pattern against the port-stripped origin (`protocol + hostname` only). `url.port` is an empty string for default ports (443 for https, 80 for http), so the fallback is only reached in non-standard-port contexts — typically local development. The domain must still satisfy the pattern; only the port tolerance is relaxed.

This applies to both string glob patterns and user-provided `RegExp` entries in `allowedRedirectOrigins`.
