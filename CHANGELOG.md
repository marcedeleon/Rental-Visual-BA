# Changelog — Rental Visual BA

Todas las actualizaciones y cambios notables del proyecto se registran aquí.
Formato basado en [Keep a Changelog](https://keepachangelog.com/es/1.1.0/). Cada entrega va acompañada de su commit correspondiente.

## [0.1.0] — Fundación del proyecto (Análisis y diseño)

### Agregado
- Documento de análisis unificado (`docs/00-documento-analisis-unificado.{pdf,html}`) con el relevamiento del caso, diagnóstico del Excel, solución propuesta, benchmarking y alcance MVP.
- Definición del stack tecnológico: SQLite + Node.js/Express + React.
- Convención de commits y metodología del proyecto (ver `AGENTS.md`).
- Estructura base del repositorio y `.gitignore`.

### Decisiones de negocio incorporadas
- **Dos ejes de estado** por equipo: Stock/ubicación (`EN_STOCK`, `ALQUILADO`, `EN_SERVICIO_TECNICO`, `BAJA/PERDIDO`) y Operatividad (`OPERATIVO`, `OPERATIVO_CON_OBSERVACIONES`, `EN_REPARACION`, `SIN_REVISAR`). Un equipo solo se alquila si está `EN_STOCK` y `OPERATIVO`.
- **Penalidad por mora:** cargo fijo por día del **+50%** del valor de alquiler diario. Excepción si el cliente avisó (prórroga registrada formalmente → no aplica recargo).
- **Garantías/pagarés:** respaldo de adeudos, liberación **manual**, nunca ejecución automática.
- **Portal web:** funciona como **pre-reserva** self-service que requiere **autorización del personal** para confirmarse.
- **Disponibilidad probable:** fecha de disponibilidad proyectada = fin de alquiler/prórroga/mora + ventana de revisión técnica + retorno a `EN_STOCK` y `OPERATIVO`.
- **Ciclo de vida del alquiler:** `PRE-RESERVA → CONFIRMADA → EN CURSO → CERRADA` (con opción a `MORA` / `CON PRÓRROGA`).

## [0.2.0] — Requerimientos

### Agregado
- `docs/01-requerimientos.md` — Requisitos funcionales (RF-01 a RF-25) por módulo (Inventario, Calendario/Disponibilidad, Reservas/Alquileres, Mora/Prórrogas/Penalidades, Portal Web, Alertas) y requisitos no funcionales (RNF-01 a RNF-07), con reglas de negocio resumidas (RN-1 a RN-7).

## [0.3.0] — Entregables para el cliente (flujos y presupuesto)

### Agregado
- `docs/07-flujos-y-procesos.{md,html,pdf}` — Entregable de flujo con 8 diagramas (contexto/actores, ciclo de vida del alquiler, pre-reserva web y autorización, motor de disponibilidad, check-out, check-in con revisión técnica, mora/prórroga y estados doble eje). Versión `.html` con Mermaid, exportable a PDF.
- `docs/08-presupuesto.{md,html,pdf}` — Presupuesto de tiempo y costos en ARS para 4 integrantes: 960 horas-hombre en 16 semanas, tarifa de referencia ARS 15.000/HH, costo de desarrollo ARS 14.400.000, infraestructura año 1 ARS 500.000, total ARS 14.900.000, con hitos de pago y análisis de sensibilidad. Versión `.html` exportable a PDF con cronograma Gantt.

### Cambiado
- `README.md` — Se incorporan los nuevos entregables y la estructura actualizada de `docs/`.

## [Próximos pasos]
- `02-modelo-de-dominio.md` — Entidades y relaciones.
- `03-casos-de-uso.md` — Casos de uso principales.
- `04-der-esquema-db.sql` — DER y script SQLite con datos normalizados.
- `05-diseno-tecnico.md` — Arquitectura.
- `06-ui-pantallas.md` — Wireframes.
