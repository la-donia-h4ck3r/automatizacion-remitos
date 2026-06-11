# Pipeline de Generación Automática de Remitos

Automatización en producción para Berta, agencia de marketing de Leones, Córdoba.
Reemplaza la creación manual mensual de remitos en Zoho Books con un pipeline
que corre solo el día 1 de cada mes.

## Problema

Berta presta servicios mensuales a +25 clientes con contratos de monto y fecha
de vencimiento fijos. El proceso era 100% manual: la administrativa entraba a
Zoho Books, seleccionaba cada cliente, cargaba monto, descripción, fecha de
vencimiento y publicaba — repitiendo esto para cada cliente todos los meses.
Consumía tiempo operativo recurrente, era propenso a errores y dependía de que
la persona lo hiciera en el momento correcto.

## Solución

Workflow en n8n que corre automáticamente el **día 1 de cada mes a las 08:00 hs**
(`cron: 0 8 1 * *`). Lee la tabla de contratos activos en Google Sheets, calcula
las fechas de vencimiento, crea cada remito en Zoho Books vía API REST y lo
publica en estado `Enviado` — sin intervención humana.

## Arquitectura

Cron: día 1 · 08:00 hs
↓
Google Sheets "Clientes"
(contact_id, monto, fecha_vencimiento, activo, periodo_facturado)
↓
Filtrar activos + Anti-duplicado
(compara periodo_facturado vs mes actual "Mayo de 2026")
↓
Calcular fecha de vencimiento
(día fijo del mes o fecha exacta)
↓
Loop por cliente (con wait 10s entre requests)
↓
POST /estimates → Zoho Books API
↓
Validar respuesta (estimate_id)
↓
┌──────────────────┐
ÉXITO              ERROR
│                  │
POST /status/sent    Log → Sheets "Errores_API"
│
Log → Sheets "Log_Creacion"
│
Update → Sheets "Clientes" (periodo_facturado)



## Stack técnico

- **n8n** (self-hosted) — orquestador del workflow
- **Zoho Books API** — creación de estimates via REST (`POST /estimates`)
- **Google Sheets** — fuente de contratos + log de resultados
- **Docker + EasyPanel + VPS Contabo** — infraestructura self-hosted

## Detalles técnicos

**Anti-duplicado**: antes de crear un remito, el workflow verifica si el campo
`periodo_facturado` del cliente ya contiene el período actual (ej: `"Mayo de 2026"`).
Si coincide, lo saltea. Esto permite reejecutar el workflow sin riesgo de
facturación doble.

**Cálculo de fechas de vencimiento**: soporta tres formatos en la columna
`fecha_vencimiento` de Sheets:
- Número de día (`10` → calcula el próximo día 10 del mes)
- Fecha fija (`15/05/2026`)
- Vacío → usa fecha del día de ejecución

**Cobro a mes vencido**: el período facturado corresponde al mes anterior
al de ejecución (`mesFacturar = today.minus({ months: 1 })`).

**Rate limiting**: 10 segundos de espera entre clientes para respetar los
límites de la API de Zoho.

**Manejo de errores**: continúa al siguiente cliente ante fallos de API.
Los errores se clasifican en dos hojas separadas según su origen.

## Estructura de Google Sheets

### Hoja "Clientes" (fuente de contratos)
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `contact_id` | text | ID del cliente en Zoho Books |
| `cliente_nombre` | text | Nombre del cliente |
| `monto` | number | Monto mensual del contrato |
| `fecha_vencimiento` | text | Día del mes o fecha fija |
| `activo` | boolean | `TRUE` para incluir en el ciclo |
| `Whatsapp` | text | Número en formato `549XXXXXXXXXX` |
| `periodo_facturado` | text | Último período procesado (ej: `Mayo de 2026`) |

### Hojas de log
- **Log_Creacion**: timestamp, cliente, número de remito, monto, fecha_vencimiento
- **Errores**: errores de validación de datos por cliente
- **Errores_API**: errores de respuesta de Zoho Books

## Estructura del proyecto

/
├── workflow-remitos.json   # Flujo exportado de n8n
└── README.md



## Resultados en producción

- **25+ remitos generados** por ciclo en ~5 minutos
- Última ejecución: 4 de mayo 2026 · 11:14 hs
- Remitos del ciclo: REM-000098 al REM-000119+
- **0 errores** en producción (hojas Errores y Errores_API vacías)
- 0 intervención manual requerida desde implementación

## Variables de entorno necesarias

ZOHO_CLIENT_ID=
ZOHO_CLIENT_SECRET=
ZOHO_ORGANIZATION_ID=
GOOGLE_SHEETS_SPREADSHEET_ID=
GOOGLE_SERVICE_ACCOUNT_EMAIL=
GOOGLE_SERVICE_ACCOUNT_KEY=



> Las credenciales reales no están incluidas en este repositorio.
