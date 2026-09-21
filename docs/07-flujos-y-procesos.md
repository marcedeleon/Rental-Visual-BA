# 07 — Flujos y Procesos del Sistema

Caso "Rental Visual BA" — Prácticas Pre Profesionales 2026.

Este documento es el **entregable de flujo** solicitado por el cliente: representa gráficamente cómo opera el sistema, de punta a punta, desde la pre-reserva web hasta la devolución del equipo, incluyendo disponibilidad, mora y prórrogas. Es la contraparte visual de los requerimientos (`01-requerimientos.md`): cada flujo referencia los RF/RN que implementa.

> Los diagramas usan sintaxis **Mermaid**. Se visualizan directamente en GitHub / VS Code y en la versión `07-flujos-y-procesos.html` (exportable a PDF).

## Índice de flujos

| # | Flujo | Qué responde |
|---|-------|--------------|
| 1 | Contexto y actores | ¿Quién usa el sistema y para qué? |
| 2 | Ciclo de vida de una reserva/alquiler | ¿Qué estados atraviesa un alquiler? |
| 3 | Pre-reserva web y autorización | ¿Cómo pide un cliente y cómo autoriza el personal? |
| 4 | Motor de disponibilidad | ¿Cómo se evita la doble reserva? |
| 5 | Check-out (retiro) | ¿Cómo sale el equipo del depósito? |
| 6 | Check-in (devolución + revisión técnica) | ¿Cómo vuelve y cómo se controla su estado? |
| 7 | Mora y prórroga | ¿Qué pasa si no devuelve a tiempo? |
| 8 | Estados doble eje | ¿En qué estado puede alquilarse un equipo? |

---

## 1. Contexto y actores

El sistema es una única plataforma con dos entornos (Portal Cliente y Backoffice) sobre una misma base de datos normalizada. Tres actores interactúan con él.

```mermaid
flowchart LR
    C["Cliente / Productora / Agencia"] -->|"Portal web: pre-reserva y presupuesto"| S["Sistema Rental Visual BA"]
    M["Personal de mostrador"] -->|"Backoffice: reservas, cobros, garantías"| S
    T["Técnico de recepción"] -->|"Check-in y revisión técnica"| S
    S --> DB[("Base de datos SQLite")]
    S -->|"Alertas: vuelve hoy / en mora / debe plata"| M
```

**Referencias:** RF-20 a RF-25, RNF-04.

---

## 2. Ciclo de vida de una reserva / alquiler

Estados oficiales: `PRE-RESERVA` → `CONFIRMADA` → `EN CURSO` → `CERRADA`, con ramas a `MORA` y `CON PRÓRROGA`.

```mermaid
stateDiagram-v2
    [*] --> PRE_RESERVA: Cliente crea pre-reserva web
    PRE_RESERVA --> CONFIRMADA: Personal autoriza
    PRE_RESERVA --> RECHAZADA: Personal deniega
    RECHAZADA --> [*]
    CONFIRMADA --> EN_CURSO: Check-out / retiro
    EN_CURSO --> CON_PRORROGA: Cliente avisa y se registra prórroga
    EN_CURSO --> MORA: Vence sin devolver
    CON_PRORROGA --> CERRADA: Check-in
    CON_PRORROGA --> MORA: Vence la prórroga
    MORA --> CERRADA: Check-in con penalidad
    EN_CURSO --> CERRADA: Check-in en término
    CERRADA --> [*]
```

**Referencias:** RF-08, RN-2, RN-4, RN-6.

---

## 3. Pre-reserva web y autorización del personal

El portal **nunca** confirma un alquiler por sí solo: genera una pre-reserva que el personal debe autorizar o denegar.

```mermaid
flowchart TD
    A["Cliente navega el catálogo público"] --> B["Consulta disponibilidad por rango de fechas"]
    B --> C{"¿Hay disponibilidad proyectada?"}
    C -- No --> B
    C -- Sí --> D["Cliente arma presupuesto y genera PRE-RESERVA"]
    D --> E["Sistema notifica al personal del depósito"]
    E --> F{"El personal revisa"}
    F -- Deniega --> G["Notifica el rechazo al cliente"]
    F -- Autoriza --> H["Alquiler pasa a CONFIRMADA"]
    H --> I["Cliente recibe la confirmación"]
```

**Referencias:** RF-09, RF-10, RF-13, RF-23, RN-6.

---

## 4. Motor de disponibilidad (evitar la doble reserva)

Es el corazón del sistema. Combina los dos ejes de estado con la proyección de disponibilidad probable (fin de alquiler / prórroga / mora + ventana de revisión técnica).

```mermaid
flowchart TD
    A["Consulta de disponibilidad para un rango"] --> B{"¿Stock = EN_STOCK?"}
    B -- No --> N["No disponible"]
    B -- Sí --> C{"¿Operatividad = OPERATIVO u OPERATIVO_CON_OBSERVACIONES?"}
    C -- No --> N
    C -- Sí --> D{"¿Solapa con un alquiler o reserva vigente?"}
    D -- No --> Y["Disponible en el rango pedido"]
    D -- Sí --> E["Proyectar fecha probable = fin alquiler / prórroga / mora + días de revisión"]
    E --> F{"¿Queda libre y operativo dentro del rango pedido?"}
    F -- Sí --> Y2["Disponible a partir de la fecha proyectada"]
    F -- No --> N
```

**Referencias:** RF-05, RF-06, RF-07, RN-1, RN-7.

---

## 5. Check-out (retiro del equipo)

```mermaid
flowchart TD
    A["Reserva en estado CONFIRMADA"] --> B["El cliente se presenta en el mostrador"]
    B --> C{"¿El equipo está EN_STOCK y OPERATIVO?"}
    C -- No --> X["Se bloquea la entrega / se ofrece alternativa"]
    C -- Sí --> D["Registrar seña y garantía: pagaré, DNI, caución, efectivo"]
    D --> E["Firmar el contrato de alquiler"]
    E --> F["Entregar el equipo: check-out"]
    F --> G["Alquiler EN CURSO / equipo ALQUILADO"]
```

**Referencias:** RF-03, RF-11, RF-17, RN-1.

---

## 6. Check-in (devolución con revisión técnica)

El técnico de recepción actualiza **ambos** ejes de estado en cada devolución. La garantía se libera siempre de forma **manual**.

```mermaid
flowchart TD
    A["El cliente devuelve el equipo"] --> B["El técnico recepciona y revisa"]
    B --> C["Actualiza eje operatividad: OPERATIVO / CON_OBSERVACIONES / EN_REPARACION"]
    C --> D["Actualiza eje stock: EN_STOCK o EN_SERVICIO_TECNICO"]
    D --> E{"¿Hay mora o daños?"}
    E -- "Devolución tardía" --> F["Calcular penalidad: +50% del valor diario por día"]
    E -- "Prórroga avisada" --> G["Sin recargo de mora"]
    E -- "Sin novedad" --> H["Sin cargo"]
    F --> I["Cerrar el alquiler: CERRADA"]
    G --> I
    H --> I
    I --> J["Liberar la garantía: proceso manual"]
```

**Referencias:** RF-04, RF-12, RF-16, RF-18, RN-3, RN-5.

---

## 7. Mora y prórroga

Regla de oro: si el cliente **avisó** con antelación, se registra prórroga formal y **no** se cobra recargo. Si no avisó, la mora se marca sola y se aplica la penalidad.

```mermaid
flowchart TD
    A["Fecha estimada de devolución"] --> B{"¿Pasó la fecha y el equipo sigue ALQUILADO?"}
    B -- No --> Z["Alquiler en curso normal"]
    B -- Sí --> C{"¿El cliente avisó con antelación?"}
    C -- Sí --> D["Registrar PRÓRROGA formal con nueva fecha"]
    D --> E["Sin recargo de mora"]
    C -- No --> F["Marcar MORA automáticamente"]
    F --> G["Penalidad: +50% del valor de alquiler diario por día"]
    G --> H["Impactar en la cuenta corriente del cliente"]
```

**Referencias:** RF-14, RF-15, RF-16, RF-19, RN-2, RN-3, RN-4.

---

## 8. Estados doble eje

### Eje 1 — Stock / ubicación

```mermaid
stateDiagram-v2
    [*] --> EN_STOCK
    EN_STOCK --> ALQUILADO: check-out
    ALQUILADO --> EN_STOCK: check-in sin falla
    ALQUILADO --> EN_SERVICIO_TECNICO: check-in con falla
    EN_STOCK --> EN_SERVICIO_TECNICO: enviar a revisión
    EN_SERVICIO_TECNICO --> EN_STOCK: reparado
    EN_STOCK --> BAJA: baja lógica
    EN_STOCK --> PERDIDO: extravío
    BAJA --> [*]
    PERDIDO --> [*]
```

### Eje 2 — Operatividad técnica

```mermaid
stateDiagram-v2
    [*] --> SIN_REVISAR
    SIN_REVISAR --> OPERATIVO: revisión OK
    SIN_REVISAR --> OPERATIVO_CON_OBSERVACIONES: revisión con detalle
    SIN_REVISAR --> EN_REPARACION: falla detectada
    OPERATIVO --> OPERATIVO_CON_OBSERVACIONES: aparece un detalle
    OPERATIVO_CON_OBSERVACIONES --> OPERATIVO: detalle resuelto
    OPERATIVO_CON_OBSERVACIONES --> EN_REPARACION: el detalle empeora
    EN_REPARACION --> OPERATIVO: reparado
    EN_REPARACION --> OPERATIVO_CON_OBSERVACIONES: reparado con detalle
```

> **Regla clave:** un equipo solo se alquila si está `EN_STOCK` **y** `OPERATIVO` (o `OPERATIVO_CON_OBSERVACIONES` informadas y aceptadas por el cliente). Estar en stock no es lo mismo que funcionar.

**Referencias:** RF-02, RF-03, RN-1.
