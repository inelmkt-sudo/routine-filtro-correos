# Alerta de Correos de Producto — Área de Marketing INEL

Routine de Claude Code que corre periódicamente y:

1. Revisa los correos nuevos de la bandeja de Natalie Aguirre (`natalieaguirre@inelinc.com`), donde llegan también los correos enviados al alias `marketing@inelinc.com`.
2. Se queda solo con los que hablan de un **producto**: programas, cursos, masterclasses, testeos, lanzamientos y summits.
3. Distingue lo que **pide acción o cambia el plan** (alerta) de lo que solo **informa un avance o comparte un recurso** (no alerta).
4. Para lo que amerita alerta: busca el producto y sus responsables en el Excel "REGISTRAR PROGRAMAS WORKSHOPS.xlsx" (hojas `INTAKE 2026`, `Masterclass INTAKE ` y `Testeos`, **solo lectura**).
5. Manda **una sola alerta** al grupo de Teams **POD'S Operaciones (Nadie habla)**, con @mención real a los responsables — o `@all` cuando el producto no tiene responsable asignado.
6. Mueve el correo alertado a la carpeta `Procesados` de Outlook.

Toda la lógica detallada está en [`CLAUDE.md`](./CLAUDE.md) — esas son las instrucciones que el Routine ejecuta en cada corrida.

## Alcance del MVP

- Un único destino: POD'S Operaciones. No se escribe en POD 1/2/3 ni en DMs.
- No se escribe nada en el Excel (la lógica de "Naciones" con round-robin fue eliminada).
- Los correos que no son de producto se ignoran por completo: ni alerta, ni movimiento de carpeta.
- Nunca se leen adjuntos, y los correos no se marcan como leídos.

## Requisitos

- Sin dependencias (`requirements.txt` vacío, `setup.sh` no hace nada) y sin variables de entorno (`.env.example`).
- Requiere 3 connectors de Composio activos: **Outlook**, **Microsoft Teams**, **Excel** (todos sobre la cuenta de Microsoft de Natalie Aguirre).

## Cómo desplegarlo

Ver [`docs/SETUP.md`](./docs/SETUP.md) para el paso a paso de cómo conectar este repo como Routine en `claude.ai/code/routines`.

## Cadencia de ejecución

La frecuencia (cron) se configura directamente en el Routine al crearlo — no está fijada en este repo.
