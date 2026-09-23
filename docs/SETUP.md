# Cómo conectar este Routine

Este Routine no necesita credenciales propias ni archivos `.env` con secretos. Todo el trabajo lo hace a través de tres conectores MCP de Composio ya vinculados a la cuenta de Microsoft de Natalie Aguirre:

1. **Outlook** — para leer los correos de la bandeja (`natalieaguirre@inelinc.com`, donde también llegan los del alias `marketing@inelinc.com`) y ponerle la categoría `Alertado` a los que ya se avisaron.
2. **Microsoft Teams** — para enviar la alerta al grupo POD'S Operaciones (Nadie habla), con @menciones reales.
3. **Excel** — para leer (nunca escribir) las hojas `INTAKE 2026`, `Masterclass INTAKE ` y `Testeos`.

## Pasos para crear el Routine en claude.ai/code/routines

1. Ve a `claude.ai/code/routines` → **New routine**.
2. Selecciona el repositorio `routine-filtro-correos`.
3. En **Environment**, crea uno custom:
   - Build command: `bash setup.sh`
   - Network access: **Full**
   - Sin variables de entorno (ver `.env.example`).
4. En **Connectors**, deja activos solo estos tres:
   - Composio — Outlook
   - Composio — Microsoft Teams
   - Composio — Excel
5. En **Trigger**, configura el cron **1 vez al día**. La ventana de búsqueda es de 30 horas, así que hay 6 horas de margen si una corrida se atrasa o falla.
6. En el prompt del Routine, pega:

   ```
   Lee CLAUDE.md y ejecuta la automatización descrita ahí.

   Operas con autonomía plena: no preguntas, no pides confirmación, no esperas
   input — nadie puede contestarte. Decide con tu propio juicio dentro de las
   reglas del documento (como --dangerously-skip-permissions).

   Recordatorios de las reglas duras:
   - Solo alertas correos de PRODUCTO, y solo los que piden acción, reportan un
     bloqueo, cambian una fecha o lanzan algo nuevo. Ante la duda, NO alertes.
   - Las actualizaciones informativas ("se actualizó X", "se añadió Y", "ya
     quedó listo") NO se alertan: si nadie del área tiene que hacer algo, es
     información, no alerta.
   - Los TESTEOS sí se alertan cuando son nuevos, cambian de etapa, están
     bloqueados o no tienen responsable asignado (ahí la acción es designarlo).
   - Destino único: el chat POD'S Operaciones (Nadie habla). Ningún otro grupo,
     ningún DM.
   - El Excel es SOLO LECTURA. No escribas nada en él.
   - La bandeja de Natalie no se altera: no marcar como leído, no mover, no
     borrar, no responder. La única escritura permitida en Outlook es agregar
     la categoría "Alertado" a un correo ya notificado.
   - Nunca leas ni menciones adjuntos.

   Reporta al final, en el log de la corrida, qué correos revisaste y qué
   decidiste con cada uno.
   ```

7. Crea el Routine y dale **Run now** para probar la primera corrida.

## Verificación de la primera corrida

- Que las alertas hayan llegado al grupo POD'S Operaciones, con las @menciones en azul (no como texto suelto).
- Que los correos alertados tengan la categoría `Alertado` en Outlook.
- Que **ningún** correo haya cambiado de carpeta ni aparezca como leído.
- Que en el log estén listados los correos revisados con la decisión de cada uno — ahí se ve si el criterio de alerta vs notificación está calibrando bien.

Si la categoría `Alertado` no existe todavía en el buzón, Outlook la aplica igual pero sin color. Para darle color, créala una vez desde Outlook (clic derecho en un correo → Categorizar → Nueva categoría).
