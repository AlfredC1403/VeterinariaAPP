# Propuesta: sistema de ventas, inventario y reportes para 3 tiendas

> Estado: **borrador técnico para ti**. Se basa en los 6 audios del cliente y en lo que Moises habló con él por llamada (`01_resumen_requerimientos.md`). Lo marcado como *propuesta* son valores o soluciones que sugiero para el diseño (por ejemplo, los días de las alertas).
> **Actualizado tras la llamada:** es una **veterinaria**; además de productos venden **servicios**; no imprimen nada; y cada sucursal **debe poder trabajar sin internet y sincronizar después**. Eso último convierte el modo sin internet en requisito del MVP y cambia la arquitectura (§6).
> Para enviar al cliente usa `04_resumen_y_preguntas_cliente.md`; este documento es la referencia técnica.
> Las cifras de costo y tiempo son **estimaciones gruesas**; se ajustan cuando el cliente responda.

## 1. Resumen ejecutivo

El cliente es una **veterinaria con 3 sucursales** que hoy lleva todo a mano en un libro. Quiere un sistema **sencillo y agradable** donde cada sucursal:

1. registre sus ventas de **productos y servicios** (con método de pago),
2. tenga su propio inventario, que se actualiza solo con cada venta o entrada de mercadería,
3. reciba **alertas de productos por vencer** (rojo) y de stock bajo,
4. haga su **cierre de caja** diario, y
5. **siga trabajando aunque se caiga el internet**,

y donde la **oficina principal** pueda ver ventas, cierres e inventario de las tres sucursales, con reportes diarios y mensuales.

**Mi recomendación:** arquitectura **híbrida**: cada sucursal trabaja sobre una base de datos **local** y se **sincroniza** con una base de datos **central** en línea cuando hay internet. Como cada sucursal es dueña de sus propios datos, la sincronización es mucho más sencilla de lo normal (§6.2). El servidor en línea tiene un costo mensual bajo, que el cliente puede pagar por transferencia a ti, que lo administras. Si el cliente no quiere pagarlo, el plan B es mucho más limitado (§6.3).

**Fuera de alcance:** facturación fiscal (SAR/CAI), contabilidad, nómina. El cliente confirmó por llamada que **no emite factura fiscal y no imprime nada**; el sistema es solo de control interno y reportes.

## 2. Alcance

### Incluido en el MVP (primera entrega)

| Módulo | Qué hace |
|---|---|
| **Acceso y sucursales** | Inicio de sesión, 3 sucursales, usuarios por persona, roles (§9). |
| **Productos** | Alta/edición de productos: código, nombre, precio de costo, precio de venta, stock mínimo, activo/inactivo. Si se venden por peso (alimento a granel), cantidades decimales. |
| **Servicios** | Código, nombre, precio de venta y **costo opcional**. Sin inventario ni vencimiento. Se agregan a la venta como un producto más; una misma venta puede mezclar productos y servicios. |
| **Sincronización y modo sin internet** | Cada sucursal guarda todo en una base local y envía/recibe cambios del servidor cuando hay conexión. Muestra "última sincronización" por sucursal (§6.2). |
| **Entradas de mercadería** | Registrar lo que llega: producto, cantidad, **lote, fecha de vencimiento y costo**. Suma al inventario. |
| **Inventario por sucursal** | Existencias por producto y por lote. Cada tienda solo ve y mueve lo suyo. |
| **Alertas** | Vencimiento (amarillo/rojo/vencido) y stock bajo, en un panel de inicio y marcadas en las listas. |
| **Punto de venta** | Buscar o escanear producto o servicio, armar la venta, método de pago (efectivo, transferencia, tarjeta). Rebaja el inventario sola. **No imprime**; el comprobante interno queda como opcional. |
| **Anulaciones y ajustes** | Anular una venta (revierte inventario) y ajustar inventario por daño, pérdida o vencimiento, siempre con motivo y usuario. |
| **Caja y cierre del día** | Apertura con fondo, cierre con totales por método de pago, efectivo esperado vs contado, diferencia. |
| **Reportes** | Ventas del día y del mes (separando productos y servicios), utilidad bruta, inventario valorizado, por vencer, stock bajo, comparativo entre sucursales (§10). Exportar a PDF/Excel. |

### Después del MVP (mejoras, según respuestas)

Traslados entre sucursales (rompen la regla de "cada sucursal escribe solo lo suyo", ver §6.2) · compras y proveedores · gastos operativos y utilidad neta · ventas a crédito · **ficha de cliente y mascota** · servicios que descuentan producto automáticamente (por ejemplo, vacuna) · comisiones o reporte por veterinario (si lo piden, el campo "quién prestó el servicio" conviene incluirlo desde el MVP) · notificaciones por correo/WhatsApp · vista para celular del dueño.

### No incluido

Factura fiscal SAR/CAI y correlativos · contabilidad · planilla/nómina · tienda en línea · integración con bancos o datáfonos (la tarjeta se registra, no se cobra desde el sistema) · historial clínico de las mascotas.

## 3. Cómo se usaría en un día normal

```
Apertura        El encargado abre caja e indica el fondo inicial.
Mañana          Llega mercadería → el encargado registra la entrada con lote y vencimiento.
Todo el día     El cajero vende productos y servicios: busca → elige método de pago → cobra.
                El inventario baja solo. El panel avisa si algo está por vencer o se acaba.
                Si se cae el internet, todo sigue igual; al volver, la sucursal envía sola lo pendiente.
Cierre          Sistema muestra: ventas por método, efectivo esperado.
                El encargado cuenta el efectivo, el sistema calcula la diferencia y cierra.
Oficina         Ve el cierre y las ventas de las 3 tiendas, y el reporte del mes.
```

## 4. Reglas de negocio propuestas

| Regla | Detalle |
|---|---|
| **Inventario por lote** | Cada entrada crea un lote con su vencimiento y su costo. El stock de un producto es la suma de sus lotes. |
| **FEFO al vender** | Al vender, el sistema descuenta **primero del lote que vence antes** (First Expired, First Out), sin que el cajero elija. Si una venta toma de dos lotes, se parte automáticamente. |
| **Utilidad exacta** | La ganancia de cada venta se calcula con el **costo del lote del que salió**, no con un costo promedio. Así el costo es correcto aunque el proveedor cambie el precio. |
| **Semáforo de vencimiento** | *Propuesta (configurable):* verde > 90 días · amarillo 31–90 · rojo ≤ 30 · vencido. El cliente decide los días y si el vencido se bloquea para venta. |
| **Stock bajo** | Alerta cuando el stock total de la sucursal ≤ stock mínimo del producto. |
| **Anular, no borrar** | Las ventas y movimientos no se borran: se anulan con motivo, usuario y fecha. Anular una venta devuelve las unidades al lote original. |
| **Ajustes con motivo** | Toda baja o corrección de inventario pide motivo (daño, vencimiento, robo, conteo) y deja rastro. |
| **Cierre inmutable** | Un cierre hecho no se edita. Solo el administrador puede reabrirlo, y queda registrado. |
| **Autonomía por sucursal** | Todo registro lleva su sucursal. Un usuario de tienda no puede ver ni tocar datos de otra. |
| **Servicios** | No tienen lote, vencimiento ni stock. El costo es opcional: si está vacío, el reporte de ganancia muestra el servicio como **"sin costo registrado"** en lugar de contarlo como 100 % de ganancia, para no inflar la utilidad. |
| **Precio al vender** | El precio sale del catálogo. Si el cliente confirma que algunos servicios varían (p. ej. consulta), se permite cambiar el precio al cobrar con permiso del encargado, y queda registrado. |

## 5. Flujo de datos (modelo)

Entidades principales (nombres tentativos):

| Entidad | Campos clave | Nota |
|---|---|---|
| Sucursal | id, nombre, activa | 3 filas al inicio. |
| Usuario | id, nombre, usuario, clave (hash), rol, sucursal (nula = todas) | Un usuario por persona. |
| Artículo (producto o servicio) | id, **tipo** (producto/servicio), código, nombre, categoría, precio_venta, costo_referencia (opcional en servicios), stock_mínimo, controla_inventario, activo | Una sola tabla para productos y servicios; los servicios llevan `controla_inventario = no`. El catálogo puede ser compartido por las tres sucursales (pregunta 9 del cliente). |
| Lote | id, producto, **sucursal**, código_lote, fecha_vencimiento, cantidad_inicial, cantidad_actual, costo_unitario | Aquí vive el inventario. |
| MovimientoInventario | id, lote, tipo (entrada, venta, anulación, ajuste, traslado), cantidad, usuario, fecha, motivo, referencia | Historial completo (kardex). |
| Venta | id, sucursal, usuario, fecha, total, estado (activa/anulada), cierre | Una por transacción. |
| VentaDetalle | venta, artículo, lote (vacío en servicios), cantidad (decimal), precio_unitario, costo_unitario (puede ser vacío en servicios), prestado_por (opcional) | Guarda precio y costo **del momento**, para que reportes viejos no cambien. |
| Pago | venta, método, monto, referencia | Permite pago mixto. |
| CierreCaja | id, sucursal, fecha, fondo_inicial, esperado por método, contado, diferencia, usuario, estado | Uno por sucursal y día (o turno). |
| Auditoría | usuario, acción, entidad, fecha, antes/después | Para anulaciones, ajustes y cambios de precio. |

**Importante:** el precio y el costo se **copian** al detalle de la venta. Si mañana cambia el precio de un producto, los reportes de ayer no se alteran.

**Para la sincronización:** todas las tablas usan identificadores únicos (UUID) generados en la sucursal, no autonuméricos, y llevan `sucursal_origen`, fecha de creación y estado de sincronización. Cada sucursal numera sus ventas con su propio prefijo (p. ej. `S1-000123`).

## 6. Arquitectura: la decisión del servidor

Esta es la pregunta clave. Hay tres caminos:

### 6.1 Opciones

| | **A. Cada tienda sola** (sin servidor) | **B. Servidor en línea** | **C. Híbrida** (local + en línea) — **recomendada** |
|---|---|---|---|
| Cómo funciona | Programa instalado en la PC de cada tienda con su propia base de datos. | Un sistema web con una base de datos central. Las tiendas entran por navegador. | Cada tienda trabaja local (aunque no haya internet) y sincroniza con un servidor cuando hay conexión. |
| ¿La oficina ve las 3 tiendas? | **No.** Solo si cada tienda exporta un archivo y alguien lo junta. | **Sí**, en tiempo real. | **Sí**, con algo de retraso. |
| ¿Necesita internet? | No. | **Sí, siempre**, para vender. | Solo para sincronizar. |
| Costo mensual (aprox.) | Ninguno. | US$10–40 | US$10–40 |
| Costo de desarrollo | Bajo. | Medio. | **Alto** (sincronizar sin perder ni duplicar datos es difícil). |
| Respaldo de datos | Manual, en cada PC. Si la PC se daña, se pierde. | Automático en el servidor. | Automático en el servidor y copia local. |
| Riesgo principal | Datos aislados; reporte consolidado manual. | Si cae el internet, no se vende. | Complejidad y errores de sincronización. |

```
 OPCIÓN B
 Tienda 1 ─┐
 Tienda 2 ─┼── Internet ──►  Servidor en línea  ◄── Oficina principal
 Tienda 3 ─┘                (sistema + base de datos
                             + copias automáticas)
```

### 6.2 Recomendación: híbrida (opción C), con "cada sucursal escribe solo lo suyo"

Tras la llamada con el cliente, el requisito es **centralizado y, a la vez, que cada sucursal trabaje sin internet y sincronice después**. Eso es la opción C, y ya no es "Fase 2": es parte del MVP. En el cuadro de §6.1 la califiqué de costo alto porque sincronizar bases de datos suele ser difícil. Aquí el propio negocio lo simplifica: **cada sucursal tiene su propio inventario, sus propias ventas y su propio cierre**, así que casi nadie escribe los mismos datos que otro.

**Quién escribe qué, y hacia dónde viaja:**

| Dato | Lo escribe | Sincronización | ¿Conflictos? |
|---|---|---|---|
| Ventas, pagos, entradas, ajustes, lotes y cierres de la sucursal X | **Solo la sucursal X** | Sucursal → servidor. Solo se **añaden** registros; una anulación es un registro nuevo, no una edición. | **Ninguno posible**: nadie más escribe esos datos. |
| Catálogo (productos, servicios, precios), usuarios, parámetros de alerta | **Solo la oficina** (administrador) | Servidor → sucursales. La sucursal solo lee. | Solo si dos administradores editan lo mismo a la vez: gana el último cambio. |

**Consecuencias de diseño:**

- Identificadores únicos (UUID) creados en cada sucursal, no autonuméricos (ver §5).
- En el MVP, las **correcciones** (reabrir un cierre, ajustar inventario) se hacen **en la sucursal** con un usuario administrador, no desde la oficina; así se respeta la regla de que cada sucursal escribe lo suyo.
- Los **traslados entre sucursales** tocan dos inventarios y rompen la regla. Van después del MVP, modelados como dos registros separados: "envío" (lo escribe la que manda) y "recepción" (lo escribe la que recibe).
- La oficina ve el stock **tal como estaba en la última sincronización** de cada sucursal. Hay que mostrar esa hora junto a cada una.

**Lo que sí queda delicado** (para hablarlo con el cliente y para tu diseño):

1. **Cuántas computadoras venden en cada sucursal.** Con una sola, la base local vive en esa PC y es simple. Con dos o más que comparten inventario sin internet, hace falta que una PC haga de servidor local en la red de la tienda. Es la pregunta 1 del cliente.
2. **Lo no sincronizado vive solo en la PC de la sucursal.** Si se daña antes de enviar, se pierde. Mitigación: sincronizar cada pocos minutos cuando haya internet, copia local diaria y un aviso de "hay N ventas sin enviar desde hace más de X horas".
3. **Reloj de la PC.** Ventas, cierres y vencimientos usan la fecha del equipo; si está mal, salen mal. Compararla con el servidor al sincronizar y avisar si hay diferencia.
4. **Actualizar el sistema.** Si hay una base local en cada PC, cada actualización con cambios de datos hay que aplicarla en todas, y el servidor debe aceptar versiones viejas por un tiempo. Conviene que la app se actualice sola.

**Cómo construir la parte local** (tu decisión; yo me inclinaría por la primera):

| Opción | A favor | En contra |
|---|---|---|
| **App de escritorio** con base local (por ejemplo SQLite) y proceso de sincronización | La más robusta sin internet; los datos no dependen del navegador. | Se instala y actualiza en cada PC; solo en el sistema operativo para el que la hagas. |
| **Web instalable (PWA)** con almacenamiento del navegador | No hay que instalar; la misma interfaz sirve a sucursales y oficina. | El navegador puede borrar los datos locales (limpiar caché, poco espacio). Con ventas aún sin enviar, es el punto más frágil. |
| **Escritorio con interfaz web** (por ejemplo Electron o Tauri) | Una sola interfaz para sucursal y oficina, con base local real. | Más piezas que mantener. |

La **oficina** no necesita trabajar sin internet: puede ser una página web que lee la base central.

**Sobre "el servidor es solo la base de datos".** Se puede, pero si cada computadora se conecta directo a la base central por internet, hay que abrir la base a internet y dejar sus credenciales en cada PC: es el punto más débil de seguridad. Recomiendo una **API pequeña** delante de la base, en el **mismo servidor**, de modo que el costo mensual no cambia. Cada sucursal tendría su propio acceso y solo podría enviar datos de su sucursal; eso además hace cumplir la regla de "cada sucursal escribe lo suyo" en el servidor y no solo en la app.

### 6.3 Si el cliente no quiere pagar servidor

Sin servidor central **no hay sincronización**: el modo sin internet pierde sentido porque cada sucursal ya sería local y nada llegaría a la oficina. Quedan dos caminos, ambos con límites claros:

| Camino | Cómo | Límites |
|---|---|---|
| **Opción A pura** | Cada tienda con su sistema y su base de datos. Cada mes o cada día exportan un archivo (Excel) y la oficina los junta. | No hay vista consolidada en tiempo real. Los reportes entre tiendas son manuales. |
| **Una PC como servidor** (la "alternativa" del audio A16) | Una computadora de la oficina o de la sucursal 1 se queda encendida como servidor, con un acceso remoto gratuito (por ejemplo un túnel o VPN privada). | Si se apaga, hay corte de luz o se daña, **ninguna tienda puede trabajar**. Requiere que alguien la mantenga, haga copias de seguridad y la proteja. Es más frágil que pagar US$10–40 al mes. |

Mi consejo es explicarle al cliente que la mensualidad es **más barata que el riesgo de perder los datos de un día de ventas**.

### 6.4 Costos recurrentes de la opción híbrida (aproximados, verificar precios vigentes)

El cliente puede pagarte por **transferencia** y tú contratar y administrar el servidor, sin que ellos necesiten tarjeta de crédito. Si la cuenta del proveedor queda a tu nombre, **ponlo por escrito**: qué incluye la mensualidad, qué pasa si dejas de dar el servicio, y que ellos pueden pedir una copia de sus datos.

| Concepto | Orden de magnitud |
|---|---|
| Servidor pequeño (VPS) con la base central y la API de sincronización | US$5–20 al mes |
| Base de datos administrada con respaldo (solo si no va en el mismo servidor) | US$0–25 al mes |
| Dominio propio | US$10–20 al año |
| Copias de seguridad externas | US$0–5 al mes |
| **Total** | **≈ US$10–40 al mes** |

Para 3 tiendas y un uso modesto, el extremo bajo debería bastar. No incluye tu tarifa de soporte.

## 7. Stack técnico sugerido (a confirmar según lo que domines)

| Capa | Sugerencia | Por qué |
|---|---|---|
| Sucursales | **App con base de datos local** (ver §6.2 para las tres variantes) | Debe vender sin internet. |
| Oficina | Página web que lee la base central | No necesita funcionar sin internet; se abre desde cualquier navegador. |
| Servidor | **API de sincronización + base de datos central** en el mismo servidor | Autentica a cada sucursal y evita exponer la base a internet. |
| Lenguaje | ASP.NET Core 8 (C#) *o* Node.js con TypeScript | Si ya trabajas en .NET, ASP.NET Core te evita aprender otra pila. |
| Base de datos | Local: **SQLite**. Central: **PostgreSQL** | Gratis, baratas de hospedar. SQL Server solo si ya cuentas con licencia; local y central deben poder intercambiar los mismos datos. |
| Interfaz | Con una librería de componentes (para "que se mire bien" sin diseñar desde cero) | Prioridad: pocas pantallas, botones grandes, rojo/amarillo/verde claro. |
| Reportes | Pantalla + exportación a PDF y Excel | Es lo que se pide con más frecuencia. |
| Respaldo | Servidor: copia automática diaria, retención de 30 días. Sucursal: copia local diaria. | El dato más valioso del cliente son sus ventas e inventario, y lo no sincronizado solo existe en la PC. |
| Impresión | **No hay.** Comprobante interno opcional más adelante. | El cliente no imprime nada. |

## 8. Pantallas principales (para validar con el cliente antes de programar)

1. **Inicio / panel de alertas:** ventas de hoy, por vencer (rojo/amarillo), stock bajo.
2. **Vender:** buscador grande para productos y servicios, lista de la venta, total, botones de efectivo/transferencia/tarjeta. Un indicador fijo de **conexión y de ventas pendientes de enviar**.
3. **Inventario:** tabla con filtros y colores por vencimiento.
4. **Entrada de mercadería:** formulario corto (producto, cantidad, lote, vencimiento, costo).
5. **Cierre de caja:** resumen del día, cuadro de efectivo contado y diferencia.
6. **Reportes:** elegir reporte, rango de fechas, sucursal, exportar.
7. **Administración:** productos y servicios, usuarios, sucursales, parámetros de alertas.

Recomiendo hacer **bocetos de las pantallas 2, 3 y 5** y mostrárselos antes de escribir código; es lo que más reduce retrabajo.

## 9. Roles y permisos (propuesta)

| Acción | Cajero | Encargado de tienda | Administrador / oficina |
|---|:---:|:---:|:---:|
| Vender | ✔ | ✔ | ✔ |
| Ver alertas e inventario de su tienda | ✔ | ✔ | ✔ |
| Registrar entradas de mercadería | ✘ | ✔ | ✔ |
| Anular una venta (con motivo) | ✘ | ✔ | ✔ |
| Ajustar inventario (con motivo) | ✘ | ✔ | ✔ |
| Ver costos y utilidad | ✘ | a definir | ✔ |
| Cerrar caja | ✘ | ✔ | ✔ |
| Reabrir un cierre | ✘ | ✘ | ✔ |
| Crear/editar productos y precios | ✘ | a definir | ✔ |
| Ver otras sucursales y el consolidado | ✘ | ✘ | ✔ |
| Gestionar usuarios | ✘ | ✘ | ✔ |

## 10. Reportes propuestos

| # | Reporte | Para qué |
|---|---|---|
| 1 | **Cierre de caja** (por sucursal y día) | Cuadre: ventas por método, esperado vs contado, diferencia, anulaciones. |
| 2 | **Ventas del día / mes / rango** | Total vendido, número de ventas, ticket promedio, por método de pago, **separando productos y servicios**. *(R-19.)* |
| 3 | **Utilidad** | Ventas − costo de lo vendido, y margen %, por sucursal, artículo y período. Los servicios sin costo salen como "sin costo registrado". |
| 4 | **Inventario actual** | Existencias por producto/lote y su valor a costo y a precio de venta. |
| 5 | **Por vencer y vencidos** | Qué lotes vencen pronto y cuánto dinero hay en riesgo. |
| 6 | **Stock bajo / por reponer** | Qué pedir. |
| 7 | **Más vendidos y sin movimiento** | Qué rota y qué se queda parado. |
| 8 | **Mermas y ajustes** | Cuánto se pierde y por qué motivo. |
| 9 | **Comparativo entre sucursales** | Para la oficina. |
| 10 | **Kardex por producto** | Todos los movimientos de un producto. |

Si el cliente quiere "valor neto" restando gastos, hace falta además un módulo de **gastos** (después del MVP).

## 11. Plan por fases y estimación

> Estimación gruesa (±40 %), para **una persona a tiempo completo**. A medio tiempo, duplica el calendario. Se afina cuando haya respuestas.

| Fase | Contenido | Duración aprox. |
|---|---|---|
| **0. Descubrimiento y diseño** | Respuestas del cliente, bocetos de pantallas, modelo de datos, decisión del servidor. | 1–1.5 semanas |
| **1. MVP local de una sucursal** | Acceso, productos **y servicios**, entradas con lote/vencimiento, inventario, alertas, punto de venta, caja y cierre, reporte diario. Todo sobre la base **local**, ya pensada para sincronizar (UUID, solo-añadir). | 4.5–5.5 semanas |
| **2. Servidor central y sincronización** | API y base central, envío/recepción de cambios, "última sincronización", 3 sucursales, vista de oficina, pruebas de desconexión y reconexión, copias de seguridad. | 4–6 semanas |
| **3. Reportes y consolidado** | Reportes mensuales, utilidad, comparativo entre sucursales, exportar PDF/Excel. | 1.5–2 semanas |
| **4. Piloto y arranque** | Una sucursal de prueba, carga del inventario inicial, ajustes, capacitación, luego las otras dos. | 2–3 semanas |
| **Total** | | **≈ 13–17 semanas** |
| **5. Mejoras opcionales** | Traslados, proveedores/compras, gastos, ficha de cliente y mascota, notificaciones, vista de celular. | Según lo que se elija |

Antes del requisito de trabajar sin internet estimé 9–12 semanas; la sincronización es lo que lo sube.

**Qué hace crecer esta estimación:** varias computadoras vendiendo a la vez en una misma sucursal sin internet, ventas a crédito, productos que se venden por fracción y por caja a la vez, requisitos de registro de medicamentos veterinarios, y una carga de inventario inicial que venga desordenada.

## 12. Riesgos y cómo se mitigan

| Riesgo | Efecto | Mitigación |
|---|---|---|
| Internet inestable en una tienda | Se puede vender (modo sin internet), pero la oficina ve datos viejos de esa sucursal. | Mostrar "última sincronización" por sucursal y avisar si pasan horas sin enviar. |
| Sincronización con errores (duplicados, ventas que no llegan) | Reportes de oficina que no coinciden con lo vendido. | Registros solo-añadir con UUID, envío repetible sin duplicar, pruebas de desconexión y reconexión, y una pantalla que compare lo local con lo del servidor. |
| PC de la sucursal se daña con ventas sin enviar | Se pierden esas ventas. | Sincronizar cada pocos minutos, copia local diaria, y aviso de pendientes. |
| Reloj de la PC mal puesto | Cierres y alertas de vencimiento errados. | Compararlo con el servidor al sincronizar y avisar. |
| Dos o más PCs vendiendo en la misma sucursal sin internet | Inventario local distinto en cada una. | Preguntarlo al cliente (pregunta 1); si ocurre, una PC hace de servidor local de la sucursal. |
| El personal sigue usando el libro | Inventario y cierre no cuadran. | Capacitación, piloto en una tienda, y no apagar el libro hasta que el cierre cuadre varios días. |
| Carga inicial de inventario incompleta | Alertas y reportes dan datos falsos. | Plantilla de Excel, validación al importar, conteo físico antes del arranque. |
| Alguien trata el sistema como si fuera un documento fiscal | Problema para el cliente. | El sistema no imprime ni numera documentos fiscales. Si algún día se agrega un comprobante, rotularlo "interno, no válido como factura". |
| Medicamentos veterinarios con requisitos de registro o control | Incumplimiento. | Preguntar al cliente (pregunta 11); no asumir. |
| Servicios sin costo distorsionan la utilidad | Ganancia inflada. | Mostrar "sin costo registrado" y no mezclarlos con el margen de productos. |
| Pérdida de datos | Pérdida de ventas del día. | Copias automáticas diarias y una prueba de restauración antes del arranque. |
| Alcance que crece ("ya que estamos…") | Retraso y sobrecosto. | Este documento fija el MVP; todo lo demás va después del MVP con su propia cotización. |
| Cliente no define quién decide | Cambios constantes. | Nombrar un contacto único que apruebe bocetos y entregas. |

## 13. Criterios de aceptación del MVP

El cliente acepta el MVP cuando, en la tienda piloto, durante **al menos 5 días seguidos**:

1. Todas las ventas se registraron en el sistema y el **cierre de caja cuadró** (efectivo esperado = contado, o la diferencia quedó explicada).
2. Un lote con vencimiento cercano **aparece en rojo** y el sistema vendió primero el lote que vence antes.
3. Una venta anulada **devolvió** las unidades al inventario.
4. El total vendido del día y del mes coincide con el libro manual de esos mismos días.
5. Un usuario de otra sucursal **no puede ver** esta tienda; el administrador sí.
6. Se restauró una copia de seguridad en una prueba.
7. Se **cortó el internet de la sucursal por una hora**, se vendió con normalidad, y al volver la conexión las ventas **aparecieron una sola vez** en la oficina y el cierre coincidió.
8. Una venta con un **servicio y un producto** descontó solo el producto, y los reportes mostraron cada uno por separado.

## 14. Siguientes pasos

1. Enviar al cliente `04_resumen_y_preguntas_cliente.md` y pedirle las respuestas marcadas con ★ primero.
2. Con las respuestas (sobre todo cuántas PCs por sucursal y cómo es el internet), confirmar el diseño híbrido y cerrar el alcance del MVP.
3. Hacer bocetos de las pantallas de vender, inventario y cierre, y validarlos.
4. Cotizar con la estimación de §11 y tu tarifa.
5. Empezar la Fase 1 en este repositorio.
