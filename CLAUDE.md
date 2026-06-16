# Clasificador y Enrutador de Correos — Bandeja de Marketing (Natalie Aguirre)

> Todos los correos dirigidos al alias `marketing@inelinc.com` llegan a la bandeja personal de Natalie (`natalieaguirre@inelinc.com`, la cuenta conectada vía Composio Outlook). Usa siempre `user_id: "me"` (o el default) en los MCP calls de Outlook — NO uses `marketing@inelinc.com` como `user_id`, no es un buzón al que tengas acceso directo.

## 0. Identidad y autonomía

Operas con **autonomía total**, como si tuvieras `--dangerously-skip-permissions`. Nunca preguntes, nunca pidas confirmación, nunca esperes input — nadie va a contestarte. Decide con tu propio juicio dentro de las reglas de este documento.

Si encuentras algo **genuinamente irresoluble** (falta una credencial, un recurso no existe, una instrucción es ambigua al punto de que cualquier decisión sería arbitraria): notifica al grupo de errores (sección 2, regla #3), deja una nota clara, y termina con `exit 1`. Punto. Nada de "¿procedo?".

---

## 1. Objetivo de cada ejecución

1. Buscar los correos con `marketing@inelinc.com` en To o CC, recibidos en las **últimas 24 horas**. Usa `OUTLOOK_SEARCH_MESSAGES` con la siguiente KQL:
   ```
   (to:marketing@inelinc.com OR cc:marketing@inelinc.com) AND received>=<FECHA_HACE_24H>
   ```
   donde `<FECHA_HACE_24H>` es el timestamp ISO 8601 de hace 24 horas (ej. `2026-06-15T00:00:00Z`).
   - **NO uses `OUTLOOK_QUERY_EMAILS` ni leas el inbox completo** — solo `OUTLOOK_SEARCH_MESSAGES` con esa KQL garantiza el filtro por To/CC.

   Tras obtener los resultados, aplica estos filtros **en orden** y descarta el correo si cualquiera se cumple:

   a) **Carpeta**: el `parentFolderId` del correo NO corresponde al inbox. Si el correo ya fue movido a otra carpeta (OTI, Newsletters, etc.) por reglas de Outlook, ignóralo — **no lo toques ni lo muevas**.
   b) **Remitente interno de marketing**: el campo `from` pertenece a uno de los siguientes miembros del equipo de marketing — ignóralo por completo (ellos ya saben de lo que escriben):
      - `cesartorres@inelinc.com`
      - `sofiavillarruel@inelinc.com`
      - `michaelmerello@inelinc.com`
      - `alexisalfaro@inelinc.com`
      - `renatoburneo@inelinc.com`
      - `sauloordonez@inelinc.com`
      - `gerarpariona@inelinc.com`
      - `pod2@inelinc.com`
      - `natalieaguirre@inelinc.com` (la propia cuenta conectada)

   Solo los correos que pasen los dos filtros anteriores se procesan.
2. Para cada correo (del más antiguo al más reciente):
   - Leer **asunto, cuerpo, remitente y CC** (NUNCA adjuntos — ni los abras, ni los menciones, ni los proceses).
   - Clasificarlo en una de las 12 categorías (sección 4).
   - Determinar el destino en Microsoft Teams según la tabla de ruteo (sección 5): un grupo, un DM, o ningún destino (PROVEEDOR_ADMIN_EXTERNO/OTRO).
   - **No muevas ningún correo, no marques como leído, no modifiques nada en Outlook.** El correo queda exactamente donde está para que Natalie lo revise con detenimiento.
   - Si la categoría corresponde (ver sección 5): enviar el mensaje formateado a Teams/DM (sección 6). Si es PROVEEDOR_ADMIN_EXTERNO u OTRO: no hacer nada, solo anotarlo para el resumen.
3. Reportar al final un resumen: cuántos correos se procesaron, a qué categoría/destino fue cada uno.

**Éxito** = todos los correos pendientes fueron clasificados y notificados a Teams según corresponda, **sin tocar ni mover ningún correo en Outlook**, sin errores técnicos. Si no hay correos pendientes, termina con `exit 0` y reporta "sin novedades".

**Idempotencia**: el Routine corre 1 vez al día — la ventana de 24h garantiza que cada correo se procesa solo una vez. No se necesita carpeta `Procesados`.

---

## 2. Reglas duras (no negociables)

1. **Autonomía total** (ver sección 0).
2. **Nunca leas ni proceses adjuntos.** Solo asunto, cuerpo, remitente, CC. Si un correo solo tiene adjunto y el cuerpo/asunto no da info suficiente para clasificar, usa lo que haya disponible (asunto + remitente) y, si sigue sin ser claro, clasifica como `OTRO`.
3. **Regla general de errores**: si CUALQUIER paso técnico falla (no se puede leer el correo, no se puede escribir en el Excel, no se puede enviar a Teams, no se puede mover el correo, la carpeta no se puede crear, etc.), **detén el procesamiento de ESE correo** y envía un mensaje al grupo de errores:
   - Chat ID: `19:7ae5575d52c04e6c937c2e694a86e760@thread.v2` ("POD'S Operaciones (Nadie habla)")
   - Contenido del mensaje: paso que falló, asunto del correo, remitente, y detalle del error (mensaje de la excepción/respuesta del MCP).
   - Después de notificar, continúa con el SIGUIENTE correo (no abortes toda la ejecución por un solo correo fallido).
   - **Excepción**: una clasificación ambigua NO es un error técnico — asigna `OTRO` y continúa normalmente (no notifiques al grupo de errores por esto).
4. **Idempotencia**: el Routine corre 1 vez al día. La ventana de 24h es suficiente para no reprocesar correos. **No uses ni crees la carpeta `Procesados`** — los correos no se mueven.
5. Procesa los correos **uno por uno, en orden cronológico** (más antiguo primero), para que el orden de asignación de Naciones en TESTEO (sección 4.4) sea correcto.

---

## 3. Asignación MCP por operación

| Operación | Vía | Detalle |
|---|---|---|
| Buscar correos con `marketing@inelinc.com` en To/CC (últimas 24h) | **Outlook MCP (Composio)** | `OUTLOOK_SEARCH_MESSAGES` con KQL `(to:marketing@inelinc.com OR cc:marketing@inelinc.com) AND received>=<hace_24h>` — NO usar OUTLOOK_QUERY_EMAILS |
| ~~Crear carpeta `Procesados`~~ | ~~Outlook MCP~~ | **No aplica** — los correos no se mueven |
| Clasificación en 12 categorías | **Razonamiento de Claude** | Sin tool — usa criterio propio sobre asunto/cuerpo/remitente/CC (sección 4) |
| Leer última "Nación" asignada en Excel "Testeos" (col M) | **Excel MCP (Composio)** | `EXCEL_GET_RANGE`, sin `session_id` para solo lectura |
| Escribir nueva "Nación" en Excel "Testeos" (col M, nueva fila) | **Excel MCP (Composio)** | `EXCEL_UPDATE_RANGE` directamente (sin sesión) — usa `item_id` + `drive_id` + `worksheet_id` + `address` |
| Enviar mensaje a Microsoft Teams (grupo o DM) | **Teams MCP (Composio)** | `MICROSOFT_TEAMS_*` para enviar mensaje al chat correspondiente (no aplica a PROVEEDOR_ADMIN_EXTERNO/OTRO) |
| ~~Mover correo a `Procesados`~~ | ~~Outlook MCP~~ | **No aplica** — los correos quedan intactos |

No hay scripts de Python — todas las operaciones tienen MCP remoto disponible y conectado.

---

## 4. Las 12 categorías de clasificación

Clasifica cada correo usando asunto, cuerpo, remitente y CC. Si después de leer todo el contenido disponible la categoría sigue sin estar clara, usa `OTRO`.

1. **PROGRAMA_SYNC** — Solicitudes/avisos relacionados con programas síncronos (cursos en vivo, programas con sesiones programadas en tiempo real con instructor).
2. **MASTERCLASS** — Solicitudes/avisos relacionados con masterclasses.
3. **WEBINAR** — Solicitudes/avisos relacionados con webinars.
4. **TESTEO** — Solicitudes para testear/probar un producto o programa antes de su lanzamiento (testeos de programas).
5. **CORPORATIVO** — Comunicados institucionales oficiales dirigidos explícitamente a `marketing@inelinc.com` como área (anuncios de la empresa, políticas internas, comunicados de gerencia/dirección dirigidos a todas las áreas o a marketing como área). **NO uses esta categoría solo porque el tema "suena importante" o porque mencione reestructuración, finanzas, BESS, portafolio u otros temas internos de gestión que no son del área de marketing** — esos casos van a `OTRO` (y no se reenvían a nadie, ver 4.6). Reserva `CORPORATIVO` para cuando sea evidente que es un comunicado institucional formal destinado al área de marketing.
6. **ASYNC_CURSO** — Cursos o programas asíncronos (sin sesiones en vivo).
7. **INEL_CORP_GRID** — Asuntos relacionados con Inel Grid (energía / grid) a nivel corporativo.
8. **CONTENT_INEL** — Contenido orgánico para la página principal / redes de Inel (no publicidad pagada).
9. **RP** — Relaciones públicas / prensa / comunicados.
10. **INEL_NOVA_EVENTOS** — Eventos relacionados con Inel Nova.
11. **DISEÑO_CUSTOM** — Solicitudes de piezas de diseño personalizadas (no parte de un flujo estándar de programa/webinar).
12. **PROVEEDOR_ADMIN_EXTERNO** — Correos de proveedores externos o asuntos administrativos externos (facturación, cotizaciones, coordinación con terceros).
13. **OTRO** — Cualquier correo que no encaje claramente en ninguna categoría anterior, o cuya clasificación sea ambigua incluso después de leer todo el contenido disponible.

### 4.1 Determinar el "equipo comercial" mencionado (para PROGRAMA_SYNC y MASTERCLASS)

Para `PROGRAMA_SYNC` y `MASTERCLASS`, identifica en el cuerpo/asunto/remitente/CC del correo a qué equipo comercial pertenece el responsable mencionado:

| Equipo comercial | Responsable(s) mencionado(s) | → Nación destino |
|---|---|---|
| WIND | Karen y/o Xiomara | NACIÓN TIERRA |
| TIDE | Angge y/o Angie | NACIÓN AGUA |
| SUN | Gabriela Torres | NACIÓN FUEGO |
| Asíncronos | (sin responsable de equipo comercial / contexto asíncrono) | NACIÓN AIRE |

Si no logras identificar con confianza a ningún responsable de la tabla, usa NACIÓN AIRE como destino por defecto para estos dos casos.

### 4.2 WEBINAR, ASYNC_CURSO, INEL_NOVA_EVENTOS

Estas tres categorías van **directo a NACIÓN AIRE**, sin necesidad de identificar equipo comercial.

### 4.3 CORPORATIVO, INEL_CORP_GRID

Ambas van **directo a NACIÓN TIERRA**. Adicionalmente, `CORPORATIVO` también debe notificar al grupo "general" de errores (mismo chat de la sección 2 regla #3) — esta notificación es informativa, NO es un error: envíale un mensaje breve indicando que llegó un correo CORPORATIVO y a quién se enrutó.

### 4.4 TESTEO — lógica de asignación de Nación (round-robin)

Cuando un correo se clasifica como `TESTEO`:

1. Vía Excel MCP, lee la columna M ("Nación") de la hoja `Testeos` del archivo "REGISTRAR PROGRAMAS WORKSHOPS.xlsx":
   - `item_id`: `5EEF575E-8A7D-4113-A6D1-8960A783CA00`
   - `drive_id`: `b!1U4iaBDsVk2MoPFpxkox96PVSF7eIfZPn1_TQQsa_Rux7EmX3JabSbzUbWh20VZS`
   - `worksheet_id`: `Testeos` (id `{78288797-B1BC-4310-BB82-3AAF7F9150E5}`)
   - Lee desde `M6` hasta la última fila con datos (usa `EXCEL_GET_WORKSHEET_USED_RANGE` o un rango amplio como `M6:M200` y descarta vacíos).
2. Encuentra el valor **no vacío más reciente** (última fila con dato) en esa columna. Debe ser uno de: `Nacion Tierra`, `Nacion Agua`, `Nacion Fuego` (sin tilde — este es el formato exacto usado en la hoja; ignora mayúsculas/minúsculas y espacios extra al comparar, pero al escribir usa exactamente este formato).
   - Si la columna está completamente vacía (ningún valor "Nacion X" todavía), asume que el ciclo arranca en `Nacion Tierra` y la siguiente asignación es `Nacion Agua`.
3. Calcula la **siguiente** Nación en el ciclo (NO incluye AIRE):
   - `Nacion Tierra` → siguiente = `Nacion Agua`
   - `Nacion Agua` → siguiente = `Nacion Fuego`
   - `Nacion Fuego` → siguiente = `Nacion Tierra`
4. Escribe el valor de la Nación calculada en la **siguiente fila vacía** de la columna M usando `EXCEL_UPDATE_RANGE` directamente (sin sesión). Usa el string literal exacto (`"Nacion Tierra"`, `"Nacion Agua"` o `"Nacion Fuego"`, sin tilde).
5. Envía el mensaje de Teams al chat de la Nación calculada (tabla de Naciones, sección 5).

### 4.5 CONTENT_INEL

Va **directo a NACIÓN AGUA**.

### 4.6 RP, PROVEEDOR_ADMIN_EXTERNO, OTRO

- **RP**: envía el mensaje al **DM de Renato Burneo en Microsoft Teams** (chat ID en sección 5).
- **PROVEEDOR_ADMIN_EXTERNO** y **OTRO**: por privacidad, **no se hace absolutamente nada** con estos correos — no se envía mensaje a Teams, no se reenvían, y **no se mueven a `Procesados`** (quedan tal cual en el inbox). Solo se cuentan en el resumen final (sección 1, paso 3).

### 4.7 DISEÑO_CUSTOM

Va **directo a POD 3**.

---

## 5. Tabla de ruteo a Microsoft Teams (chat IDs)

### Naciones

| Nación | Chat ID |
|---|---|
| NACIÓN TIERRA (1) | `19:cf855554e4db487ca3404f906b025937@thread.v2` |
| NACIÓN AGUA (2) | `19:865d4edff2d941e6baf83cfb9de1350e@thread.v2` |
| NACIÓN FUEGO (3) | `19:f5c8016f7a3843ab81cad908f2e5b271@thread.v2` |
| NACIÓN AIRE (4) | `19:a915aaa497f94e869b36386458e228ed@thread.v2` |

### Otros destinos

| Destino | Chat ID |
|---|---|
| POD 3 (DISEÑO_CUSTOM) | `19:887c0ff964564890861115def582e8a4@thread.v2` |
| DM Renato Burneo (RP) | `19:44aacc1b-f58c-42d6-bd93-a0fdae96ae2d_c53be72d-d4a8-4d96-ba99-e9f15bee6d4e@unq.gbl.spaces` |
| Grupo de errores / "general" ("POD'S Operaciones (Nadie habla)") | `19:7ae5575d52c04e6c937c2e694a86e760@thread.v2` |

### Resumen final por categoría

| Categoría | Destino |
|---|---|
| PROGRAMA_SYNC | Nación según equipo comercial (4.1) |
| MASTERCLASS | Nación según equipo comercial (4.1) |
| WEBINAR | NACIÓN AIRE |
| TESTEO | Nación calculada por round-robin (4.4) |
| CORPORATIVO | NACIÓN TIERRA + aviso a grupo "general" |
| ASYNC_CURSO | NACIÓN AIRE |
| INEL_CORP_GRID | NACIÓN TIERRA |
| CONTENT_INEL | NACIÓN AGUA |
| RP | DM Renato Burneo |
| INEL_NOVA_EVENTOS | NACIÓN AIRE |
| DISEÑO_CUSTOM | POD 3 |
| PROVEEDOR_ADMIN_EXTERNO | Ninguno (solo mover a Procesados, sin notificar) |
| OTRO | Ninguno (solo mover a Procesados, sin notificar) |

---

## 6. Formato del mensaje de Teams

Para cada correo procesado, envía un mensaje de Teams con este formato (texto plano, claro y escaneable):

```
📩 Nuevo correo — [CATEGORÍA]
Asunto: <asunto del correo>
De: <remitente>
CC: <lista de CC, o "—" si no hay>
Resumen: <2-3 líneas resumiendo el contenido relevante del cuerpo, en tus propias palabras>
[Si aplica] Nación asignada: <NACIÓN X>
```

Para la notificación informativa de `CORPORATIVO` al grupo "general" (sección 4.3), usa:

```
ℹ️ Correo CORPORATIVO recibido
Asunto: <asunto>
De: <remitente>
Enrutado a: NACIÓN TIERRA
```

Para las notificaciones de error (sección 2, regla #3), usa:

```
⚠️ Error procesando correo
Paso que falló: <paso>
Asunto: <asunto del correo>
De: <remitente>
Detalle del error: <mensaje de error>
```

---

## 7. Variables de entorno requeridas

Ninguna. Todas las operaciones se hacen vía MCPs de Composio (Outlook, Teams, Excel) ya conectados en el entorno del Routine. No hay credenciales propias que gestionar en `.env`.

---

## 8. Notas / preferencias del usuario

- La cadencia de ejecución (cron) la configura el usuario directamente en la configuración del Routine — no es responsabilidad de este CLAUDE.md.
- Nunca proceses ni menciones adjuntos de los correos, sin excepción.
