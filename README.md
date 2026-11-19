# github-actions-basics

Repo de práctica para entender lo básico de **GitHub Actions**: un workflow que corre los tests de un proyecto Node mínimo en cada push y pull request a `master`. El código (`add` / `subtract`) es solo un pretexto para tener algo que testear.

## Requisitos

- Node `>=18` en local (para tener `node:test`); el CI usa Node 24
- Sin dependencias: no hay `npm install` ni `package-lock.json`

## Cómo correrlo

```bash
npm test     # node --test index.spec.js
```

## Estructura

```
.github/workflows/test.yml  # workflow de CI
index.js                    # funciones add y subtract
index.spec.js               # tests con node:test + node:assert
package.json                # solo define el script "test"
```

## Workflow (`.github/workflows/test.yml`)

| Parte | Valor |
| --- | --- |
| Disparadores | `push` y `pull_request` a `master` |
| Runner | `ubuntu-latest` |
| Pasos | `actions/checkout@v6` → `actions/setup-node@v6` (Node 24) → `npm test` |

El resultado se ve en la pestaña **Actions** del repo y como check (✔/✖) en cada commit y PR.

## Hallazgos (lo que falló en el camino)

- **`npm ci` falla sin `package-lock.json`**: exige el lockfile. Como el proyecto no tiene dependencias, se quitó el paso. Si se agregan dependencias, hay que commitear el lockfile y volver a poner `npm ci` antes de `npm test`.
- **Glob en el script de test**: `node --test **/*.spec.js` no funcionaba en el CI (con Node 20 `--test` no acepta globs, y el `sh` de npm no expande `**` recursivamente). Se cambió a la ruta explícita `index.spec.js`. Desde Node 21 `--test` sí acepta globs, así que con Node 24 se podría volver a usar un patrón (entre comillas para que lo resuelva Node y no el shell).
- **Versiones de las actions**: se pasó de `@v4` a `@v6` y de Node 20 a Node 24 para no usar versiones antiguas.

## Referencias

- [Workflow syntax – GitHub Actions](https://docs.github.com/actions/writing-workflows/workflow-syntax-for-github-actions)
- [Node.js test runner (`node:test`)](https://nodejs.org/api/test.html)
