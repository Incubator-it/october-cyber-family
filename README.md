# october-cyber-family

Familia de ways de seguridad ([ways.sh](https://www.ways.sh/)), al estilo de [incu-way](https://github.com/Incubator-it/incu-way). Cada incidencia vive como un member way con su skill; la familia se instala de una vez.

## Instalación

Como familia:

```bash
npm install -g @incu/ways

ways add github:Incubator-it/october-cyber-family
```

Solo la skill:

```bash
npx skills add https://github.com/Incubator-it/october-cyber-family.git --skill node-october-2026-security
```

## Skills

| Skill | Member way | Qué hace |
|---|---|---|
| `node-october-2026-security` | `october/node-oct-2026` | Audita un repo Node.js contra el October 2026 Security Release (CVE-2026-111497 y relacionados). Inventario y análisis en solo lectura, reporte antes de modificar, y PR de parche solo si hay algo afectado. |

El procedimiento está en [`skills/node-october-2026-security/SKILL.md`](skills/node-october-2026-security/SKILL.md). Los manifests están en [`way.yaml`](way.yaml) y [`ways/`](ways/README.md).

## Versiones

- `VERSION` y el frontmatter de cada `skills/*/SKILL.md` son la versión del bundle de skills.
- `metadata.version` de cada `way.yaml` versiona el artefacto Ways por separado.

## Validar

```bash
ways validate way.yaml ways/node-oct-2026/way.yaml
ways conformance way.yaml ways/node-oct-2026/way.yaml
```
