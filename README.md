# october-cyber-family

A family of security ways ([ways.sh](https://www.ways.sh/)), in the style of [incu-way](https://github.com/Incubator-it/incu-way). Each incident is a member way with its own skill; the family installs as one.

## Install

The whole family:

```bash
npm install -g @incu/ways

ways add github:Incubator-it/october-cyber-family
```

The skill only:

```bash
npx skills add https://github.com/Incubator-it/october-cyber-family.git --skill node-october-2026-security
```

## Skills

| Skill | Member way | What it does |
|---|---|---|
| `node-october-2026-security` | `october/node-oct-2026` | Audits a Node.js repo against the October 2026 Security Release (CVE-2026-111497 and related issues). Inventory and analysis are read-only, the report comes before any edit, and a patch PR opens only when something is affected. |

The procedure is in [`skills/node-october-2026-security/SKILL.md`](skills/node-october-2026-security/SKILL.md). The manifests are in [`way.yaml`](way.yaml) and [`ways/`](ways/README.md).

## Versions

- `VERSION` and the frontmatter of each `skills/*/SKILL.md` are the skill-bundle version.
- `metadata.version` on each `way.yaml` versions that Ways artifact on its own.

## Validate

```bash
ways validate way.yaml ways/node-oct-2026/way.yaml
ways conformance way.yaml ways/node-oct-2026/way.yaml
```
