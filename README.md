# Clasificador y Enrutador de Correos — Área de Marketing INEL

Routine de Claude Code que corre periódicamente y:

1. Revisa los correos nuevos de `marketing@inelinc.com`.
2. Clasifica cada uno en una de 12 categorías (PROGRAMA_SYNC, MASTERCLASS, WEBINAR, TESTEO, CORPORATIVO, ASYNC_CURSO, INEL_CORP_GRID, CONTENT_INEL, RP, INEL_NOVA_EVENTOS, DISEÑO_CUSTOM, PROVEEDOR_ADMIN_EXTERNO, OTRO).
3. Envía un mensaje resumen al grupo o persona de Microsoft Teams correspondiente.
4. Para correos de TESTEO, calcula y registra la "Nación" (TIERRA/AGUA/FUEGO) que le toca por turno en el Excel de Testeos.
5. Mueve el correo procesado a la carpeta `Procesados` de Outlook.
6. Si algo falla técnicamente, avisa al grupo de errores con el detalle.

Toda la lógica detallada está en [`CLAUDE.md`](./CLAUDE.md) — esas son las instrucciones que el Routine ejecuta en cada corrida.

## Requisitos

- No requiere instalación de dependencias (`requirements.txt` vacío, `setup.sh` no hace nada).
- No requiere variables de entorno (`.env.example`).
- Requiere 3 connectors de Composio activos: **Outlook**, **Microsoft Teams**, **Excel** (todos sobre la cuenta de Microsoft de Natalie Aguirre / `marketing@inelinc.com`).

## Cómo desplegarlo

Ver [`docs/SETUP.md`](./docs/SETUP.md) para el paso a paso de cómo conectar este repo como Routine en `claude.ai/code/routines`.

## Cadencia de ejecución

La frecuencia (cron) se configura directamente en el Routine al crearlo — no está fijada en este repo.
