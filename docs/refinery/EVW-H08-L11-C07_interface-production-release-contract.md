# H08-L11-C07 · Contrato operativo de liberación Interface

**Estado:** `BLOCKED_BY_MISSING_PRODUCTION_RELEASE_CONTRACT`
**Ejecución de promoción/deploy:** `NOT_EXECUTED`
**Fecha de inspección:** 2026-10-02 (America/Santiago)

## 1. Propósito, alcance y exclusiones

Registrar la evidencia versionada disponible para planificar una futura liberación manual de `evaas-studio`. Se inspeccionaron archivos versionados y referencias Git. No se promovieron ramas, no se hizo push, no se ejecutó Vercel CLI y no se llamó a servicios ni endpoints productivos.

Este documento no autoriza una liberación. No modifica código de aplicación, infraestructura ni configuración de despliegue.

## 2. Ramas canónicas y referencias observadas

| Uso | Rama remota | SHA observada |
| --- | --- | --- |
| Integración | `develop` | `1e9c5c7948c0f1de195b86ac0dfa86a2f7a44c8b` |
| Producción | `main` | `c0f146076559b96f24d31f258ca92ebaa7f0dbc3` |
| HEAD remoto | `main` | `c0f146076559b96f24d31f258ca92ebaa7f0dbc3` |

La matriz operativa confirmada para C07 designa `develop` como integración y `main` como producción. `git remote show origin` y `git ls-remote --symref origin HEAD` confirman además `main` como HEAD del remoto. `origin/main..origin/develop` contiene catorce commits; `git diff --check origin/main..origin/develop` pasó. El árbol local inicial estaba limpio. SHA local inicial: `92f706ac86886e384ca6562b5549c952d6573c01`.

## 3. Evidencia de configuración versionada

- `README.md` documenta despliegue cloud estático en Vercel con `npx vercel --prod`, Node.js `20.x` y directorio de salida `dist/evaas-studio/browser`.
- `package.json` define `npm run build` como `ng build --configuration=production`; `build:vercel` ejecuta la misma configuración. No existe script versionado de publicación. `start:vercel` ejecuta `vercel dev` para desarrollo.
- `angular.json` sustituye el environment de desarrollo por `src/environments/environment.production.ts` en la configuración productiva. El archivo fija `production: true` y compila `apiUrl: 'https://api.evaas.lat'` dentro del bundle.
- No hay variables públicas de build requeridas por la configuración versionada. La documentación menciona los posibles nombres `API_URL`, `NG_APP_API_URL`, `VITE_API_URL` y `EVAAS_API_URL` sólo como ejemplos de variables externas que podrían existir; el repositorio no confirma que estén configuradas. Registrar nombres no implica que sean necesarias.
- La URL productiva documentada para la API dependiente es `https://api.evaas.lat`. El repositorio no identifica de manera verificable la URL pública productiva de la propia Interface ni su dominio asignado en Vercel.
- `vercel.json` declara rewrites SPA a `/index.html` y políticas de caché. No incluye identificador de proyecto, build/output settings ni asignación de dominio.
- No hay archivos versionados bajo `.github` o `.vercel` que acrediten una integración Git de Vercel o un despliegue automático al actualizar `main`. El README documenta un comando de CLI; el ajuste efectivo del proyecto Vercel (incluida la rama de producción) no es visible en este repositorio.
- `package.json` incluye el script `check:api:health`, que comprueba la respuesta HTTP de `https://api.evaas.lat/v3/api-docs`. Esa ruta pertenece a la API y no es una health/readiness check de Interface. No se ejecutó.
- `docs/operations/production-api-url-caddy.md` documenta la arquitectura esperada de la API: Caddy HTTPS `:443` → `localhost:8091`. No configura el hosting de Interface.
- No se encontró procedimiento versionado de rollback.

## 4. Secuencia propuesta — `NOT_EXECUTED`

1. Confirmar con el operador el proyecto Vercel, dominio productivo de Interface, vínculo del repositorio, rama de producción configurada y si `main` despliega automáticamente.
2. Confirmar que `origin/main` contiene únicamente el conjunto aprobado de cambios H08; revisar el diff, árbol limpio y `git diff --check`.
3. Ejecutar en el checkout aprobado `npm test -- --watch=false` y aceptar sólo las dos fallas heredadas aprobadas (`ObjectCardComponent should create` y `ObjectsGridComponent should create`); ejecutar `npm run build` y revisar el artefacto `dist/evaas-studio/browser`.
4. Integrar los commits aprobados de `develop` a `main` mediante la revisión autorizada por el operador. Este contrato no ejecuta merges ni push.
5. Publicar el build de ese mismo SHA en el proyecto Vercel confirmado. El README menciona `npx vercel --prod`, pero no se ejecutó: primero deben confirmarse el project link, el target y el comportamiento de `main` para evitar un segundo despliegue.
6. Verificar la URL de Interface que entregue el operador y la disponibilidad de la API según el procedimiento autorizado. La respuesta de `/v3/api-docs` sólo comprueba respuesta HTTP de API; no sustituye readiness.
7. Con una sesión administrativa ya autorizada, hacer un smoke de sólo lectura: abrir el panel, cargar la lista de Resources y la evidencia LIORA, revisar consola/red y confirmar cero `PATCH` u otras mutaciones. No hacer login ni llamadas productivas desde este procedimiento documental.

Toda la secuencia queda `NOT_EXECUTED`.

## 5. Checks y resultado de inspección

- Preflight Git: árbol limpio; `develop=1e9c5c79…`, `main=c0f14607…`; `diff --check` pasó.
- Evidencia previa R3: `docs/refinery/EVW-H08-L11-C03_R3_governed-e2e-gate.md` reporta 139 pruebas correctas y dos fallas heredadas permitidas, build productivo aprobado y smoke E2E aislado. Esa evidencia no acredita una liberación a Vercel ni salud de producción.
- Build y pruebas de C07: no ejecutados; esta cápsula sólo registra el contrato operativo.
- Health de Interface: no hay endpoint o probe de hosting documentado en el repositorio.
- Health de API: existe el script no ejecutado `check:api:health` contra `/v3/api-docs`; no es readiness.
- Smoke productivo: no ejecutado. No hubo login, llamadas de producción ni mutaciones.

## 6. Rollback

No hay un rollback de Interface verificable en los archivos versionados. El repositorio no documenta cómo seleccionar una versión previa en Vercel, cuánto se conservan deployments, quién puede reasignar tráfico ni cómo validar el rollback. La disponibilidad de controles de rollback en la cuenta Vercel no se infiere. Se requiere un procedimiento confirmado por el operador que identifique el deployment anterior y su restauración sin rebuild ambiguo.

## 7. Secretos y datos prohibidos en el registro

No registrar valores de tokens, credenciales, secretos de Vercel, variables privadas, contenido de archivos `.env`, URLs con credenciales, datos personales ni sesiones administrativas. Las variables públicas del frontend quedan embebidas en el bundle y nunca deben contener secretos. Este documento conserva nombres y URL pública documentada, no valores secretos.

## 8. Bloqueos y datos requeridos del operador

- Proyecto Vercel ligado al repositorio, configuración de producción y responsable con permisos de deploy.
- Evidencia de si `main` activa despliegue automático; si lo hace, confirmación del mecanismo único que debe monitorearse.
- URL/dominio productivo de Interface y verificación de su mapeo al deployment correcto.
- Build/output settings efectivos y lista segura de variables públicas requeridas, sólo por nombre.
- Procedimiento de rollback validado, deployment previo conservado y responsable de ejecutarlo.
- Sesión administrativa autorizada para el smoke de sólo lectura y criterios de consola/red aceptables.

## 9. Decisión

`BLOCKED_BY_MISSING_PRODUCTION_RELEASE_CONTRACT`

No marcar `READY_FOR_MANUAL_PRODUCTION_RELEASE` hasta confirmar la configuración externa de Vercel, el dominio, el mecanismo de despliegue de `main` y un rollback verificable.
