# AGENTS.md — Rental Visual BA

Sistema de alquiler y control de equipamiento audiovisual.
Prácticas Pre Profesionales 2026 — Universidad Nacional de Lanús (Lic. en Sistemas).

## Propósito
Gestionar integralmente un negocio de alquiler temporario de bienes audiovisuales de alto valor patrimonial ("Rental Visual BA"): garantizar disponibilidad estricta por rangos de fechas (sin doble reserva), controlar ciclos de retiro (check-out) y devolución (check-in), supervisar el estado técnico/operativo de los equipos, y gestionar garantías, señas, cobranzas y penalidades por mora.

## Contexto del cliente
Empresa que opera en Zona Norte y Capital (Bs. As.), hoy manejada con un Excel manual (archivo real: `Caso_4_Audiovisual_Rental_Datos_Equipos.xlsx`, con hojas `Equipamiento_Inventario`, `Productoras_Clientes` y `Registro_Alquileres`).
Problemáticas:
1. No saben qué hay realmente disponible (doble reservas).
2. Las devoluciones no se registran en fecha (6 de 10 alquileres sin devolución real cargada).
3. El estado técnico se describe de forma informal y no se distingue stock de operatividad.
4. Garantías, señas y cobros sin control (no se sabe quién debe mora/recargos).

## Stack tecnológico
- Base de datos: SQLite (archivo local)
- Backend: Node.js + Express (REST API)
- Frontend: React (responsive, para celular y mostrador del depósito)

## Metodología
Análisis → Diseño → Desarrollo. Primero se consolida la documentación de análisis/diseño; el código se implementa luego y contra esa base.

## Estados del equipo (dos ejes independientes)
1. Stock / ubicación:
   - `EN_STOCK`, `ALQUILADO`, `EN_SERVICIO_TECNICO`, (`BAJA` / `PERDIDO`)
2. Operatividad técnica:
   - `OPERATIVO`, `OPERATIVO_CON_OBSERVACIONES`, `EN_REPARACION`, `SIN_REVISAR`

Regla clave: un equipo solo se presta si está `EN_STOCK` y `OPERATIVO` (o `OPERATIVO_CON_OBSERVACIONES` comunicadas al cliente).
No es lo mismo estar en stock que funcionar. El técnico que recepciona/revisa actualiza ambos ejes en cada check-in o revisión.

## Disponibilidad probable y mora
- Un alquiler cuya fecha estimada de devolución pasó y el equipo sigue `ALQUILADO` se marca en **MORA** automáticamente.
- El personal puede registrar una **PRÓRROGA formal** (nueva fecha estimada; el cliente la debe avisar).
- La disponibilidad futura se proyecta: fin del alquiler (o prórroga/mora) + ventana de revisión técnica (campo "días de revisión" por equipo, default 1 día) + retorno a `EN_STOCK` y `OPERATIVO`.
- Una reserva solo se confirma si, tras la proyección, el equipo quedará `EN_STOCK` y `OPERATIVO` para las fechas pedidas.

## Reglas de negocio
- **Penalidad por mora:** cargo fijo por día de **+50%** del valor de alquiler diario. **Excepción:** si el cliente avisó con antelación (prórroga registrada), no se aplica recargo.
- **Garantías/pagarés/DNI/caución/seña:** respaldo de adeudos; liberación **manual**, nunca ejecución automática.
- **Portal web:** funciona como **pre-reserva** (`PRE-RESERVA`); requiere **autorización del personal** antes de ejecutarse como alquiler.
- **Ciclo de vida del alquiler:** `PRE-RESERVA → CONFIRMADA → EN CURSO → CERRADA`, con opción a `MORA` o `CON PRÓRROGA`.

## Requisitos funcionales clave
- Gestión de equipos e inventario (con los dos ejes de estado).
- Consulta de disponibilidad por rangos de fechas (evitar doble reserva; considerar mora y ventana de revisión técnica).
- Ciclos de check-out (retiro) y check-in (devolución con revisión técnica).
- Estado técnico/operativo de equipos y su historial de revisiones.
- Gestión de garantías, señas, cobranzas, prórrogas y penalidades por mora.
- Portal web público de catálogo con pre-reservas y autorización.
- Alertas: "qué vuelve hoy", "qué está en mora" y "quién me debe plata".

## Estructura de carpetas
```
rental-visual-ba/
├── AGENTS.md
├── README.md
├── CHANGELOG.md
├── .gitignore
├── docs/ (00..06)
└── (backend/, frontend/ en fase de desarrollo)
```

## Convenciones de código y repo
- Documentación en español (idioma del equipo/cliente).
- Commits con **Conventional Commits**: `docs:`, `feat:`, `fix:`, `chore:`, `refactor:`, `test:`.
- Backend: rutas REST con recursos en plural (`/api/equipos`, `/api/alquileres`).
- Frontend: componentes React funcionales, orientados a dispositivo móvil.
- Validaciones claras de fechas para evitar solapamiento de reservas.
- Mantener `CHANGELOG.md` actualizado a la par de cada entrega.

## Reglas del equipo
- No adivinar el modelo de datos: se diseña desde el enunciado y el Excel real, y se valida antes de codificar.
- Mantener la documentación en `docs/` actualizada a la par del código.
- No comitear secretos ni datos reales sensibles del cliente.
- Ejecutar pruebas antes de dar por terminada una funcionalidad.
