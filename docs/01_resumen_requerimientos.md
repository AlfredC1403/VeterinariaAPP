# Resumen de requerimientos

## Fuentes

- **[A12]–[A17]:** los 6 audios que el cliente le mandó a Moises (WA0012 a WA0017, 4 min 27 s). Los transcribí con dos modelos de voz automáticos (ver `00_transcripciones.md`).
- **[Llamada]:** lo que Moises habló con el cliente en una llamada aparte.
- **[Moises]:** lo que Moises indica como requisito o decisión.

Las tres fuentes valen igual. Donde Moises aclara algo, esa es la versión que usa el sistema.

## 1. En una frase

Un sistema **sencillo y que se vea bien** para una **veterinaria con 3 sucursales**, que reemplace el libro manual: registra **ventas de productos y servicios** con su método de pago, lleva el **inventario con fechas de vencimiento**, avisa con **alertas en rojo** (por vencer y por acabarse), permite el **cierre de caja** de cada sucursal y da **reportes completos**. Cada sucursal **trabaja sin internet** y sincroniza con un **servidor central** donde la oficina ve todo. **No imprime ni emite factura fiscal.**

## 2. Contexto

- **Negocio:** veterinaria con 3 sucursales [A12, Llamada]. Venden **productos** (por ejemplo, medicamentos) y dan **servicios** (por ejemplo, consultas) [A13, Llamada].
- **Hoy:** todo se lleva **a mano**, las ventas en un libro [A14, Llamada].
- **Lo que más les preocupa:** la fecha de vencimiento *"representa un problema bastante grande"* [A13].
- **Personas que mencionan:** **David** [A13] y **Julio** (un modelo de voz oye "Yule") [A16]. No sé qué rol tiene cada uno.

## 3. Requerimientos

### Sucursales y usuarios

| ID | Requerimiento | Fuente |
|---|---|---|
| R-01 | Sistema sencillo, pero que "se mire bien". | A12 |
| R-02 | Se usa en las **3 sucursales**. | A12 |
| R-03 | Cada sucursal tiene **su propio usuario** (sucursal 1, 2, 3, o con el nombre de cada una). | A12, A15 |
| R-04 | Cada sucursal es **autónoma**: su propio inventario, su propio proceso de venta y su propio cierre. Los datos **no se mezclan**. | A15, A17 |
| R-05 | **Información centralizada:** desde la **oficina principal** se ven las ventas, los cierres y el inventario de todas las sucursales. | A12, A16, Moises |
| R-06 | **Servidor central solo para la base de datos.** | Moises |
| R-07 | Cada sucursal **trabaja sin internet** y **sincroniza** sus datos en línea cuando vuelve la conexión. | Moises |
| R-08 | El cliente **paga el servidor por transferencia a Moises**, quien lo administra. No necesitan tarjeta de crédito. | Moises |

### Productos, servicios e inventario

| ID | Requerimiento | Fuente |
|---|---|---|
| R-09 | El inventario se **actualiza automáticamente**: una venta lo rebaja y una entrada lo suma al existente. | A13 |
| R-10 | Cada producto lleva **nombre, precio de costo, precio de venta, código y fecha de vencimiento** (esta última "muy importante"). | A13 |
| R-11 | **Alerta en rojo** cuando un producto está **cerca de vencer**. | A13 |
| R-12 | **Alerta en rojo** cuando un producto está **por acabarse** (stock bajo). Necesita un stock mínimo por producto. | Moises |
| R-13 | Ver los productos **disponibles en cada sucursal**, con el inventario al día. | A15 |
| R-14 | **Servicios:** se registran con **nombre, código y precio de venta**. El **costo es opcional**: si el cliente quiere registrar lo que le cuesta el servicio, se guarda; si no, solo el precio. Sin inventario ni vencimiento. | Moises, Llamada |

### Ventas y caja

| ID | Requerimiento | Fuente |
|---|---|---|
| R-15 | Registrar la venta **directo en el sistema** en vez del libro: productos, servicios y total. Una misma venta puede mezclar ambos. | A14, Moises |
| R-16 | Registrar el **método de pago**. Se proponen **efectivo, transferencia y tarjeta**; falta confirmar con el cliente cuáles acepta. | A14, Moises |
| R-17 | Dejar todo listo para el **cierre y cuadre de caja** del día, **por sucursal**. | A14, A15, A17 |
| R-18 | **No se imprime nada y no hay factura fiscal ni SAR.** El sistema es de uso interno, para reportería. | Llamada |

### Reportes

| ID | Requerimiento | Fuente |
|---|---|---|
| R-19 | **Módulo de reportes:** al final del día, la venta total; al mes, las ventas totales acumuladas. | A17 |
| R-20 | **Reportes completos:** costos, utilidad ("valor neto"), ventas por producto y por servicio, inventario valorizado, comparativo entre sucursales. | Moises |

## 4. Puntos de diseño que salen de estos requisitos

1. **Sucursales autónomas + oficina que ve todo (R-04, R-05, R-07):** base local en cada sucursal y base central en línea. Como cada sucursal escribe solo lo suyo, la sincronización no genera conflictos entre sucursales (`03_propuesta.md` §6.2).
2. **Servidor "solo base de datos" (R-06):** se puede. Recomiendo una API pequeña delante, en el mismo servidor y con el mismo costo, para no exponer la base a internet (`03_propuesta.md` §6.2).
3. **Dos alertas en rojo (R-11, R-12):** para distinguirlas, cada una lleva su etiqueta ("Por vencer" y "Por acabarse") y se muestran en el mismo panel de inicio.
4. **Servicios sin costo (R-14):** en el reporte de ganancia aparecen como "sin costo registrado" y no se cuentan como 100 % de ganancia.
5. **"Costos" (R-20):** puede significar el costo de lo vendido, lo comprado en el período, o los gastos del negocio (alquiler, salarios). Los dos primeros salen con los datos del MVP; los gastos requieren un módulo aparte. Está en las preguntas.
6. **Alternativa del audio A16** (entrar desde la PC de la sucursal 1 a las otras): queda reemplazada por el servidor central; ya no se necesita.

## 5. Aún sin definir (van en las preguntas)

Cuántas computadoras venden a la vez en cada sucursal (clave para el modo sin internet) · qué internet y qué equipos tienen · productos a granel (alimento por libra) · lista de servicios y precios, y si el precio cambia al cobrar · quién presta el servicio · roles y permisos · anulaciones y descuentos · ventas a crédito · datáfono y pagos combinados · fondo de caja y gastos de caja · cantidad de productos · registro de medicamentos veterinarios · fecha de arranque.

Están en `04_resumen_y_preguntas_cliente.md` §4, y la lista larga con el porqué de cada una en `02_preguntas_cliente.md`.
