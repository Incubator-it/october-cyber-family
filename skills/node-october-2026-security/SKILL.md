---
name: node-october-2026-security
version: 0.1.0
description: Audits a Node.js repository against the October 2026 Security Release (CVE-2026-111497 Brotli out-of-bounds write/RCE, TLS hostname bypass, dns.resolve NUL, and repeated randomUUID/randomInt entropy in snapshots and SEA). Inventories every declared Node version, analyzes exposed attack surface, reports before any change, and opens a patch pull request only when something is affected. Use when the user mentions the Node.js October 2026 security release, CVE-2026-111497, CVE-2026-105483, CVE-2026-106069, CVE-2026-105482, Brotli decoder RCE, Node 22.23.4, 24.22.1, 26.12.1, or asks to patch Node.js for that advisory.
---

Act as a security auditor for this Node.js repository.

CONTEXT
Node.js published the October 2026 Security Release (2026-10-21). The fix is in versions 22.23.4, 24.22.1, and 26.12.1. The most severe issue is CVE-2026-111497 (high): an out-of-bounds write in the Brotli decoder that allows RCE. Other highs: CVE-2026-105483 and CVE-2026-106069 (TLS hostname bypass) and CVE-2026-105482 (dns.resolve with NUL). The mediums are CVE-2026-105481, CVE-2026-106072, and CVE-2026-106074 (repeated randomUUID/randomInt entropy in snapshots and SEA). The release also updates c-ares, OpenSSL, Undici, SQLite, and the CA certs.
Node 20 and earlier are EOL and do NOT receive the fix.
References: https://nodejs.org/en/blog/vulnerability/october-2026-security-releases and https://vercel.com/kb/bulletin/node-js-security-advisory

PHASE 1: INVENTORY (read-only)
1. Find every Node version the repo declares:
   .nvmrc, .node-version, .tool-versions, volta in package.json, engines.node, Dockerfile*/docker-compose* (FROM node:..., distroless/alpine images), .github/workflows/* (setup-node, node-version, matrix), .gitlab-ci.yml, Jenkinsfile, azure-pipelines, .devcontainer/, vercel.json, netlify.toml, serverless.yml, template.yaml (SAM), app.yaml, Procfile, IaC (Terraform/CDK/Pulumi: runtime nodejsXX.x).
2. Detect the package manager (npm/yarn/pnpm/bun) and, for a monorepo, the workspaces.
3. List the security checks the repo already has: Dependabot/Renovate (config and open PRs), CodeQL/code scanning, npm audit / audit-ci in CI, Snyk, Trivy/Grype for images, OSV-Scanner, secret scanning, SECURITY.md. If there are open Dependabot or code scanning alerts, read them.

PHASE 2: ANALYSIS
1. For each place Node is declared: current version, whether it is affected, and the target version (the patched release on the same line, or migrate to 22.x/24.x if it is 20 or earlier).
2. Exposed surface in the code:
   - zlib.createBrotliDecompress, brotliDecompress(Sync), zlib.decompress with br
   - Middleware that decompresses incoming bodies (body-parser with inflate, Content-Encoding: br, proxies)
   - fetch/undici/axios/got/node-fetch that automatically decompress third-party responses
   - Image optimization or proxies in self-hosted Next.js
   - TLS with IPs as the host (tls.connect, https.request with servername/IP, a custom checkServerIdentity)
   - dns.resolve* with external input
   - crypto.randomUUID()/randomInt() in startup snapshots (--build-snapshot) or SEA
   - Permission Model (--permission / --experimental-permission)
3. Run the tools already available without installing anything globally: npm audit --omit=dev --json (or yarn npm audit / pnpm audit), and npx osv-scanner if it is reachable. Check undici, direct and transitive.
4. Run the security checks the repo already has (package.json scripts such as audit, lint:security, and so on).

PHASE 3: REPORT (before modifying anything)
Table: file | what it declares | current version | affected (yes/no) | target version | change risk.
Second table: exposed surface (file:line, pattern, risk explanation).
Third: audit and existing check results (CVE/GHSA, package, severity, fix).

PHASE 4: FIX (only if something is affected)
1. Branch: security/node-oct-2026
2. Update each declaration to the exact patch version (22.23.4 / 24.22.1 / 26.12.1). Pin Docker images to that exact tag (for example node:22.23.4-alpine); if they use a digest, update it.
3. If Node is ≤20: migrate to 22.x or 24.x, update engines and CI, and call out the compatibility risk in the PR.
4. Update undici if it is a direct dependency; for transitive vulnerabilities, use overrides/resolutions/pnpm.overrides.
5. Regenerate the lockfiles with the same package manager.
6. Run a clean install, build, tests, lint, and the repo's security checks. If something fails, try to fix it; if you cannot, document it and do not hide it.
7. If the repo has no automated security checks, propose them in the same PR (in separate commits): dependabot.yml (npm, docker, github-actions), an npm audit --audit-level=high step in CI, and, if there is a Dockerfile, a Trivy scan.
8. Open a PR titled "security: update Node.js for the October 2026 security release". In the description include the phase 3 tables, the changes, test and check results, the risks, and the manual-actions section (below).

CONSTRAINTS
- Do not touch business logic except to mitigate a vulnerability, and in that case justify it.
- Do not push to main, merge, or deploy.
- Do not delete or disable existing checks.
- If nothing is affected, do not open the PR: return only the report.

PHASE 5: MANUAL ACTIONS (always include in the PR or the report)
Generate a checklist adapted to what you found, covering at least:
- [ ] Rebuild and redeploy every image/service after merge (the PR alone does not patch production).
- [ ] In each environment (prod, staging, dev), verify that process.version is the patched version (startup log or health endpoint).
- [ ] Review the runtime configured outside the repo: Vercel/Netlify dashboard, Lambda/Cloud Functions/Cloud Run/App Service, PaaS.
- [ ] Update Node on your own servers/VMs, self-hosted CI runners, and development machines (nvm/volta/fnm).
- [ ] Review shared base images in other repos or internal registries.
- [ ] If you are on Vercel with 20.x: switch to 22.x/24.x and redeploy.
- [ ] Review logs/WAF for suspicious requests with Content-Encoding: br on or before 2026-10-21.
- [ ] If backends sit behind a CDN/WAF: restrict direct access so only proxied traffic gets in.
- [ ] If snapshots/SEA used randomUUID: regenerate the binaries and assess whether any generated ID/token is predictable.
- [ ] List the environments the agent could not verify and assign an owner to each.
