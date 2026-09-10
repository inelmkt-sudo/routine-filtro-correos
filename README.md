# Alerta de Correos de Producto — Área de Marketing INEL

Routine de Claude Code que corre periódicamente y:

1. Revisa los correos de las últimas 30 horas en la bandeja de Natalie Aguirre (`natalieaguirre@inelinc.com`), donde llegan también los correos enviados al alias `marketing@inelinc.com`. Corre 1 vez al día; las 6 horas de solape son a propósito, para que nada se pierda si el cron se atrasa.
2. Se queda solo con los que hablan de un **producto**: programas, cursos, masterclasses, testeos, lanzamientos y summits.
3. Distingue lo que **pide acción o cambia el plan** (alerta) de lo que solo **informa un avance o comparte un recurso** (no alerta).
4. Para lo que amerita alerta: busca el producto y sus responsables en el Excel "REGISTRAR PROGRAMAS WORKSHOPS.xlsx" (hojas `INTAKE 2026`, `Masterclass INTAKE ` y `Testeos`, **solo lectura**).
5. Manda **una sola alerta** al grupo de Teams **POD'S Operaciones (Nadie habla)**, con @mención real a los responsables — o `@all` cuando el producto no tiene responsable asignado.
6. Le pone al correo alertado la categoría `Alertado` de Outlook — es la única memoria de la rutina, y lo que evita avisar dos veces lo mismo.

Toda la lógica detallada está en [`CLAUDE.md`](./CLAUDE.md) — esas son las instrucciones que el Routine ejecuta en cada corrida.

## Alcance del MVP

- Un único destino: POD'S Operaciones. No se escribe en POD 1/2/3 ni en DMs.
- No se escribe nada en el Excel (la lógica de "Naciones" con round-robin fue eliminada).
- Los correos que no son de producto se ignoran por completo.
- **La bandeja queda como estaba**: no se marca nada como leído, no se mueve ni se borra nada, no se leen adjuntos. La única huella es la categoría `Alertado` en los correos ya avisados.

## Requisitos

- Sin dependencias (`requirements.txt` vacío, `setup.sh` no hace nada) y sin variables de entorno (`.env.example`).
- Requiere 3 connectors de Composio activos: **Outlook**, **Microsoft Teams**, **Excel** (todos sobre la cuenta de Microsoft de Natalie Aguirre).

## Cómo desplegarlo

Ver [`docs/SETUP.md`](./docs/SETUP.md) para el paso a paso de cómo conectar este repo como Routine en `claude.ai/code/routines`.

## Cadencia de ejecución

La frecuencia (cron) se configura directamente en el Routine al crearlo — no está fijada en este repo.
