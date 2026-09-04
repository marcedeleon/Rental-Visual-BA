# 01 — Requisitos del Sistema

Caso "Rental Visual BA" — Prácticas Pre Profesionales 2026.

Este documento define los requerimientos funcionales (RF) y no funcionales (RNF) del Sistema Híbrido de Gestión de Alquileres y Catálogo Web. Complementa el análisis unificado (`docs/00-documento-analisis-unificado.pdf`).

## 1. Alcance
Sistema para un negocio de alquiler temporario de equipamiento audiovisual, con dos entornos integrados sobre una misma base de datos normalizada:
- **Entorno Cliente (Portal Web):** catálogo público con pre-reservas autogestionadas.
- **Entorno Administrador (Backoffice / Mostrador):** control total de inventario, reservas, alquileres, estados, cobros, garantías y penalidades.

El MVP cubre: inventario, disponibilidad/reservas, check-in/check-out, estados del equipo (dos ejes), mora/prórrogas/penalidades, y catálogo web con pre-reservas.

---

## 2. Módulo: Inventario de Equipos

### RF-01 Gestión del catálogo
El sistema permitirá dar de alta, modificar, consultar y dar de baja (lógica) equipos del inventario. Cada equipo tendrá un **identificador único** y datos normalizados:
- Código único (formato `EQ-###` automático)
- Nombre / descripción
- Categoría
- Precio de alquiler por día (**unidad homogénea**: configurable ARS/USD, se muestra en una sola moneda)
- Valor de reposición
- Ubicación en depósito (zona/estantería estructurada)
- **Días de revisión** (ventana de revisión técnica tras devolución; default 1)

### RF-02 Dos ejes de estado por equipo (obligatorio)
Cada equipo mantiene **dos** estados independientes:
1. **Stock / ubicación:** `EN_STOCK`, `ALQUILADO`, `EN_SERVICIO_TECNICO`, `BAJA`, `PERDIDO`.
2. **Operatividad técnica:** `OPERATIVO`, `OPERATIVO_CON_OBSERVACIONES`, `EN_REPARACION`, `SIN_REVISAR`.

### RF-03 Regla de prestabilidad
Un equipo solo se entrega a un cliente si está en estado `EN_STOCK` **y** `OPERATIVO` (o `OPERATIVO_CON_OBSERVACIONES` informadas al cliente y aceptadas). El sistema impedirá el check-out de un equipo en `EN_REPARACION`, `SIN_REVISAR` o que no esté `EN_STOCK`.

### RF-04 Historia de revisiones
El sistema registrará un historial de revisiones técnicas por equipo (fecha, técnico, resultado de operatividad, observaciones), permitiendo reconstruir la trazabilidad de cada ítem.

---

## 3. Módulo: Calendario y Disponibilidad

### RF-05 Motor de disponibilidad por rango de fechas
El sistema permitirá consultar la disponibilidad de un equipo (o conjunto) para un rango de fechas, impidiendo **estructuralmente** la doble reserva.

### RF-06 Proyección de disponibilidad probable
Al consultar, el sistema calculará y mostrará la **fecha probable de disponibilidad** de cada equipo según:
- fin del alquiler vigente (o prórroga / resolución de mora)
- + días de revisión técnica
- + retorno a estado `EN_STOCK` y `OPERATIVO`

### RF-07 Confirmación de reserva condicionada
Una reserva solo se confirma si, según la proyección, el equipo quedará `EN_STOCK` y `OPERATIVO` para el rango solicitado.

---

## 4. Módulo: Reservas y Alquileres

### RF-08 Ciclo de vida de una reserva/alquiler
Estados: `PRE-RESERVA` → `CONFIRMADA` → `EN CURSO` → `CERRADA`, con transiciones a `MORA` o `CON PRÓRROGA`.

### RF-09 Pre-reserva web (self-service)
Desde el Portal Web, un cliente registrado podrá generar una **pre-reserva** (estado `PRE-RESERVA`) y armar un presupuesto.

### RF-10 Autorización del personal (obligatorio)
Toda pre-reserva debe ser **autorizada o denegada por el personal** del depósito para pasar a `CONFIRMADA`. El sistema impedirá que un alquiler se ejecute sin esta autorización.

### RF-11 Check-out (retiro)
El personal registrará el retiro físico del equipo: el alquiler pasa a `EN CURSO`, los equipos pasan a stock `ALQUILADO`.

### RF-12 Check-in (devolución con revisión técnica)
Al devolver, el **técnico de recepción** registrará la revisión y actualizará **ambos** ejes de estado de cada equipo devuelto, cerrando el alquiler (`CERRADA`).

### RF-13 Presupuesto / valores
Antes de confirmar, el sistema podrá generar un presupuesto con los ítems, fechas y total estimado.

---

## 5. Módulo: Mora, Prórrogas y Penalidades

### RF-14 Marca de mora automática
Si la fecha estimada de devolución pasó y el equipo sigue en stock `ALQUILADO`, el alquiler se marca automáticamente en `MORA`.

### RF-15 Prórroga formal (obligatorio)
El personal podrá registrar una **prórroga** con nueva fecha estimada de devolución. La prórroga solo procede si el cliente **avisó** con antelación; nunca de palabra (siempre queda registrada).

### RF-16 Cálculo de penalidad por mora
Se calcula un **cargo fijo por día del +50% del valor de alquiler diario** por cada día de devolución tardía.
**Excepción:** si hay prórroga avisada y registrada, **no** se aplica recargo por mora.

### RF-17 Registro de garantías y señas
El sistema registrará depósitos en garantía (pagaré, retención de DNI, caución, seña, efectivo, etc.) vinculados al alquiler y a su estado.

### RF-18 Liberación manual de garantía (obligatorio)
Las garantías **nunca se ejecutan automáticamente**. Su liberación es un proceso **manual** del personal, previa verificación del saldo.

### RF-19 Registro de cobros y estado de cuenta
El sistema registrará cobros, señas y saldos, permitiendo consultar el estado de cuenta por cliente y el total adeudado.

---

## 6. Módulo: Portal Web Cliente

### RF-20 Catálogo público
Acceso público al catálogo con disponibilidad real proyectada por rango de fechas (sin datos sensibles).

### RF-21 Registro e inicio de sesión
Productoras, agencias y clientes freelance podrán registrarse e iniciar sesión.

### RF-22 Panel del cliente
El cliente verá sus pre-reservas y alquileres vigentes, fechas de devolución pactadas y estado de cuenta.

### RF-23 Notificación de autorización
El cliente recibirá la confirmación o denegación de su pre-reserva.

---

## 7. Módulo: Alertas y Tablero (Mostrador)

### RF-24 Panel de control diario
El mostrador verá, de un vistazo:
- Equipos que **deben volver hoy**
- Alquileres en **mora**
- Cuentas por cobrar / quién debe plata

### RF-25 Alertas
Alertas de devoluciones próximas y vencidas para el seguimiento diario.

---

## 8. Requisitos No Funcionales (RNF)

### RNF-01 Usabilidad
Interfaz simple, clara y legible, optimizada para **celular y mostrador del depósito**, con curva de aprendizaje mínima (cliente sin conocimientos informáticos).

### RNF-02 Disponibilidad del servicio
El sistema debe estar disponible durante el horario de operación del depósito; al ser una app web responsive, disponible en celular y PC.

### RNF-03 Integridad de datos
Validación obligatoria de fechas para evitar solapamientos; unicidad de códigos de equipo y contratos; transacciones que garanticen la consistencia del inventario (un equipo no puede estar en dos alquileres a la vez).

### RNF-04 Seguridad
Autenticación y roles diferenciados (cliente vs. administrador/técnico). Acceso al backoffice restringido al personal.

### RNF-05 Rendimiento
Consultas de disponibilidad con respuesta ágil incluso sobre el catálogo completo.

### RNF-06 Mantenibilidad y documentación
Documentación actualizada en `docs/` y en el `CHANGELOG.md`, con historial de cambios versionado en Git.

### RNF-07 Portabilidad
Stack simple (SQLite + Node/Express + React) que pueda ejecutarse en una sola máquina (la del depósito) y accederse desde la red local/celular.

---

## 9. Reglas de negocio clave (resumen)
| # | Regla |
|---|-------|
| RN-1 | Un equipo se presta solo si `EN_STOCK` y `OPERATIVO`. |
| RN-2 | La mora se marca automáticamente al vencer sin devolución. |
| RN-3 | Penalidad de mora = +50% del valor diario por día. Exime si hay prórroga avisada. |
| RN-4 | La prórroga siempre se registra formalmente; nunca de palabra. |
| RN-5 | Las garantías NO se ejecutan solas; liberación manual. |
| RN-6 | Toda pre-reserva web requiere autorización del personal para confirmarse. |
| RN-7 | Disponibilidad proyectada considera fin de alquiler/prórroga/mora + revisión técnica + retorno a stock operativo. |
