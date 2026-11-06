# OIDC preview security build

Based on upstream `preview-new-oidc` commit `2d1a2ed233d13ee216b147951afda8321d42f5d2`.

This fork updates vulnerable dependencies, pins a newer Node 22 / Alpine 3.23 base, upgrades Alpine packages during the build, and omits npm, Yarn, source files and development tooling from the final runtime image. The checked-in lockfile and scoped pnpm overrides preserve patched transitive versions. Nodemailer is consistently pinned to 10.1.4 across dependencies.

Axios response headers are narrowed to strings before image-cache parsing. A documented type compatibility assertion bridges email-templates' Nodemailer 8 type definitions to Nodemailer 10; the transport was verified with a JSON-only send, without sending email.

Build locally:

```sh
podman build --build-arg COMMIT_TAG=local-security-2d1a2ed-20261106 \
  -t localhost/seerr:oidc-patched-20261106 .
```

Validation on 2026-11-06:

- Upstream source tests: 193 passed, 0 failed, 0 skipped (54 suites).
- Server and client TypeScript checks passed.
- Next.js and server production builds passed.
- Nodemailer 10 / email-templates JSON transport smoke test passed.
- Trivy 0.76.2 final runtime-image scan: 0 known vulnerabilities. Initial upstream preview scan: 12 critical, 144 high, 187 medium, 35 low package findings.
- Trivy source secret scan: 0 findings.
- Live Authentik authorization-code + PKCE login completed successfully with the existing administrator account.

The image scan is a point-in-time dependency check, not a guarantee against undiscovered vulnerabilities. OIDC remains an upstream preview feature. Refresh package advisories and rerun the checks before rebuilding or updating.

Authentik requires a confidential OAuth2 provider with the `authorization_code` grant enabled, `openid profile email` scopes, and strict redirect URIs for `/login` and `/profile/settings/linked-accounts`. Enable Seerr proxy support when serving HTTPS behind a reverse proxy. Existing users must explicitly link their identity or have an administrator map an established identity; do not automatically link administrator accounts by an unverified email claim.
