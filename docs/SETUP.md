# Cómo conectar este Routine

Este Routine no necesita credenciales propias ni archivos `.env` con secretos. Todo el trabajo lo hace a través de tres conectores MCP de Composio que ya tienes vinculados a tu cuenta de Microsoft (Natalie Aguirre):

1. **Outlook** — para leer correos de `areamarketing@inelinc.com` y moverlos a la carpeta `Procesados`.
2. **Microsoft Teams** — para enviar los mensajes a los grupos y al DM de Renato Burneo.
3. **Excel** — para leer y escribir la columna "Nación" de la hoja `Testeos`.

## Pasos para crear el Routine en claude.ai/code/routines

1. Sube esta carpeta completa a un repositorio de GitHub (privado), por ejemplo `routine-clasificador-correos-marketing`.
2. Ve a `claude.ai/code/routines` → **New routine**.
3. Selecciona el repositorio que acabas de subir.
4. En **Environment**, crea uno custom:
   - Build command: `bash setup.sh`
   - Network access: **Full**
   - No necesitas agregar ninguna variable de entorno (ver `.env.example`).
5. En **Connectors**, asegúrate de tener activos (y deja solo estos, quita los que no se usen):
   - Composio — Outlook
   - Composio — Microsoft Teams
   - Composio — Excel
6. En **Trigger**, configura el cron con la cadencia que prefieras (mínimo cada 1 hora). Por ejemplo, 2 veces al día.
7. En el prompt del Routine, pega:

   ```
   Lee CLAUDE.md y ejecuta la automatización descrita ahí. Operas con autonomía
   plena: no preguntas, no pides confirmación, no esperas input — nadie puede
   contestarte. Tienes libertad total para decidir según las instrucciones y tu
   juicio (como --dangerously-skip-permissions). Si algo es genuinamente
   imposible, detente con nota clara y exit error. Respeta la asignación MCP vs
   script del CLAUDE.md. Reporta resumen al final.
   ```

8. Crea el Routine y dale **Run now** para probar el primer run.

## Verificación del primer run

Revisa:
- Que la carpeta `Procesados` se haya creado (si no existía) dentro de `areamarketing@inelinc.com`.
- Que los correos pendientes hayan llegado a los chats de Teams correctos según la tabla de ruteo de `CLAUDE.md`.
- Si un correo era TESTEO, que la columna M de la hoja `Testeos` tenga la nueva "Nación" escrita correctamente.
- Que no haya mensajes inesperados en el grupo "POD'S Operaciones (Nadie habla)" (grupo de errores).

Si algo falla, revisa el log del run — el Routine deja un mensaje de error en ese grupo de Teams con el detalle exacto del paso que falló.
