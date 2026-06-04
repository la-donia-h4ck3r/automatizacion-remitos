# 🧾 Automatización de Creación de Remitos

Sistema automatizado que conecta las fuentes de datos operativas con el sistema de facturación, eliminando la generación manual de remitos y reduciendo errores administrativos.

---

## 🧩 Problema

La generación de remitos se realizaba de forma completamente manual. Cada vez que se registraba una operación, una persona debía cargar los datos del cliente, completar la información comercial, crear el remito, verificar errores y guardar la documentación.

Esto generaba:

- Alto tiempo administrativo en tareas repetitivas
- Errores frecuentes de carga manual
- Dependencia total de intervención humana
- Imposibilidad de escalar sin aumentar personal

---

## ✅ Solución

Sistema de automatización que detecta nuevos registros comerciales, valida los datos necesarios y genera el remito automáticamente en el sistema de facturación, registrando evidencia para auditoría.

---

## 🔁 Flujo del sistema

```
Google Sheets (nuevo registro comercial)
        ↓
    n8n (motor de automatización)
        ↓
Validación de datos obligatorios
        ↓
API de facturación → Creación del remito
        ↓
PostgreSQL + Google Sheets (log y auditoría)
```

---
## 📸 Workflow en n8n

![Workflow de creación de remitos](Worflow_creacion_de_remitos.png)

## ⚙️ Stack tecnológico

| Herramienta | Rol |
|---|---|
| **n8n** | Motor de automatización (self-hosted) |
| **API de facturación** | Creación del remito en el sistema |
| **Google Sheets** | Fuente de datos y registro de resultados |
| **PostgreSQL** | Base de datos para auditoría y trazabilidad |
| **Docker / EasyPanel** | Infraestructura self-hosted en VPS |

---

## 📋 Funcionalidades principales

- Detección automática de nuevos registros comerciales
- Obtención y validación de datos obligatorios antes de procesar
- Generación automática del remito en el sistema de facturación vía API
- Registro del resultado de cada operación
- Almacenamiento de evidencia para auditoría

---

## 📈 Resultados

- ✅ Reducción drástica del tiempo administrativo
- ✅ Eliminación de errores de carga manual
- ✅ Estandarización del proceso de generación de remitos
- ✅ Capacidad de escalar sin aumentar personal administrativo

---

## 📁 Estructura del repositorio

```
├── workflows/          # Exportaciones JSON del workflow de n8n
├── docs/               # Diagramas y documentación técnica
└── README.md
```

---

## ⚠️ Nota sobre credenciales

Este repositorio no contiene credenciales, tokens ni datos de clientes reales. Las variables de entorno sensibles se gestionan mediante `.env` y no se suben al repositorio.

---

*Desarrollado como solución productiva en entorno real — Berta Multimedios, Leones, Córdoba, Argentina.*
