---
name: node-october-2026-security
version: 0.1.0
description: Audits a Node.js repository against the October 2026 Security Release (CVE-2026-111497 Brotli out-of-bounds write/RCE, TLS hostname bypass, dns.resolve NUL, and repeated randomUUID/randomInt entropy in snapshots and SEA). Inventories every declared Node version, analyzes exposed attack surface, reports before any change, and opens a patch pull request only when something is affected. Use when the user mentions the Node.js October 2026 security release, CVE-2026-111497, CVE-2026-105483, CVE-2026-106069, CVE-2026-105482, Brotli decoder RCE, Node 22.23.4, 24.22.1, 26.12.1, or asks to patch Node.js for that advisory.
---

Actuá como auditor de seguridad para este repositorio Node.js.

CONTEXTO
Node.js publicó el October 2026 Security Release (21/10/2026). El fix está en las versiones 22.23.4, 24.22.1 y 26.12.1. La falla más grave es CVE-2026-111497 (high): una escritura fuera de límites en el decodificador Brotli que permite RCE. También hay otros high: CVE-2026-105483 y CVE-2026-106069 (bypass del hostname TLS) y CVE-2026-105482 (dns.resolve con NUL). Los medium son CVE-2026-105481, CVE-2026-106072 y CVE-2026-106074 (entropía de randomUUID/randomInt repetida en snapshots y SEA). Además se actualizan c-ares, OpenSSL, Undici, SQLite y los CA certs.
Node 20 y anteriores están en EOL y NO reciben el fix.
Referencias: https://nodejs.org/en/blog/vulnerability/october-2026-security-releases y https://vercel.com/kb/bulletin/node-js-security-advisory

FASE 1: INVENTARIO (solo lectura)
1. Buscá todas las versiones de Node que declara el repo:
   .nvmrc, .node-version, .tool-versions, volta en package.json, engines.node, Dockerfile*/docker-compose* (FROM node:..., imágenes distroless/alpine), .github/workflows/* (setup-node, node-version, matrix), .gitlab-ci.yml, Jenkinsfile, azure-pipelines, .devcontainer/, vercel.json, netlify.toml, serverless.yml, template.yaml (SAM), app.yaml, Procfile, infra IaC (Terraform/CDK/Pulumi: runtime nodejsXX.x).
2. Detectá el gestor de paquetes (npm/yarn/pnpm/bun) y en el caso de monorepo, los workspaces.
3. Listá los checks de seguridad que ya tiene el repo: Dependabot/Renovate (config y PRs abiertos), CodeQL/code scanning, npm audit / audit-ci en CI, Snyk, Trivy/Grype para imágenes, OSV-Scanner, secret scanning, SECURITY.md. Si hay alertas abiertas de Dependabot o code scanning, leelas.

FASE 2: ANÁLISIS
1. Por cada lugar donde se declara Node: versión actual, si está afectado y versión objetivo (la misma línea parcheada, o migrar a 22.x/24.x si es la 20 o anterior).
2. Superficie expuesta en el código:
   - zlib.createBrotliDecompress, brotliDecompress(Sync), zlib.decompress con br
   - Middleware que descomprime bodies entrantes (body-parser con inflate, Content-Encoding: br, proxies)
   - fetch/undici/axios/got/node-fetch que descomprimen respuestas de terceros automáticamente
   - Image optimization o proxies en Next.js self-hosted
   - Uso de TLS con IPs como host (tls.connect, https.request con servername/IP, checkServerIdentity personalizado)
   - Uso de dns.resolve* con entrada externa
   - crypto.randomUUID()/randomInt() en startup snapshots (--build-snapshot) o SEA
   - Permission Model (--permission / --experimental-permission)
3. Corré las herramientas disponibles sin instalar nada global: npm audit --omit=dev --json (o yarn npm audit / pnpm audit), y npx osv-scanner si está accesible. Revisá undici directo/transitivo.
4. Corré los checks de seguridad que ya tenga el repo (scripts de package.json como audit, lint:security, etc.).

FASE 3: REPORTE (antes de modificar)
Tabla: archivo | qué declara | versión actual | afectado (sí/no) | versión objetivo | riesgo del cambio.
Segunda tabla: superficie expuesta (archivo:línea, patrón, explicación de riesgo).
Tercera: resultado de audit y checks existentes (CVE/GHSA, paquete, severidad, fix).

FASE 4: FIX (solo si hay algo afectado)
1. Rama: security/node-oct-2026
2. Actualizá cada declaración a la versión de patch exacta (22.23.4 / 24.22.1 / 26.12.1). Fijá las imágenes Docker a ese tag exacto (por ejemplo node:22.23.4-alpine); si usan digest, actualizalo.
3. Si hay Node ≤20: migrá a 22.x o 24.x, actualizá engines y CI, y marcalo en el PR como cambio con riesgo de compatibilidad.
4. Actualizá undici si es dependencia directa; para vulnerabilidades transitivas, usá overrides/resolutions/pnpm.overrides.
5. Regenerá los lockfiles con el mismo gestor.
6. Ejecutá install limpio, build, tests, lint y los checks de seguridad del repo. Si algo falla, intentá arreglarlo; si no se puede, documentalo y no lo escondas.
7. Si el repo no tiene checks de seguridad automáticos, proponé en el mismo PR (en commits separados): dependabot.yml (npm, docker, github-actions), un paso de npm audit --audit-level=high en CI y, si hay Dockerfile, un escaneo con Trivy.
8. Abrí un PR con título "security: actualizar Node.js por October 2026 security release". En la descripción poné las tablas de la fase 3, los cambios, el resultado de tests y checks, los riesgos y la sección de acciones manuales (abajo).

RESTRICCIONES
- No toques lógica de negocio salvo para mitigar una vulnerabilidad, y en ese caso justificalo.
- No hagas push a main, merge ni deploy.
- No borres ni desactives checks existentes.
- Si no hay nada afectado, no abras el PR: devolvé solo el reporte.

FASE 5: ACCIONES MANUALES (incluir siempre en el PR o el reporte)
Generá un checklist adaptado a lo que encontraste, que cubra como mínimo:
- [ ] Hacer rebuild y redeploy de todas las imágenes/servicios después del merge (el PR solo no parchea producción).
- [ ] Verificar en cada entorno (prod, staging, dev) que process.version sea la versión parcheada (log al arranque o endpoint de health).
- [ ] Revisar el runtime configurado fuera del repo: panel de Vercel/Netlify, Lambda/Cloud Functions/Cloud Run/App Service, PaaS.
- [ ] Actualizar Node en servidores/VMs propios, runners self-hosted de CI y en las máquinas de desarrollo (nvm/volta/fnm).
- [ ] Revisar imágenes base compartidas en otros repos o registries internos.
- [ ] Si se está en Vercel con 20.x: cambiar a 22.x/24.x y redeployar.
- [ ] Revisar logs/WAF por requests sospechosos con Content-Encoding: br desde el 21/10/2026 o antes.
- [ ] Si hay backends detrás de un CDN/WAF: limitar el acceso directo para que solo entre tráfico por el proxy.
- [ ] Si se usaban snapshots/SEA con randomUUID: regenerar los binarios y evaluar si algún ID/token generado es predecible.
- [ ] Listar los entornos que el agente no pudo verificar y asignarles un responsable.
