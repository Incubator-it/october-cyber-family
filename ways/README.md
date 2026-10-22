# ways/ — october-cyber-family

`ways/v1alpha1` artifacts for this family. The root [`/way.yaml`](../way.yaml) is a **WayFamily**: it references member ways and carries the payload they share. The `ways` CLI is [`@incu/ways`](https://www.npmjs.com/package/@incu/ways).

| Artifact | Kind | What |
|---|---|---|
| `/way.yaml` | WayFamily | `october/cyber` — members plus the shared skill and the `vcs-host` slot. |
| `node-oct-2026/way.yaml` | Way (member `node-oct-2026`) | `october/node-oct-2026` — Node.js October 2026 Security Release. Driven by `skills/node-october-2026-security`. |

## Layout

A member way is installed from its own directory, so it cannot point at `../../skills/…`. This repo keeps one `skills/` tree at the root. The family declares the skill; the member manifest describes the flow that skill executes.

## Install

```bash
npm install -g @incu/ways

ways add github:Incubator-it/october-cyber-family
ways add github:Incubator-it/october-cyber-family --members node-oct-2026
```

Or only the skill, via skills.sh:

```bash
npx skills add https://github.com/Incubator-it/october-cyber-family.git --skill node-october-2026-security
```

## Validate

```bash
ways validate way.yaml ways/node-oct-2026/way.yaml
ways conformance way.yaml ways/node-oct-2026/way.yaml
```
