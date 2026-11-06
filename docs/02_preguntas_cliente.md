# Preguntas iniciales para el cliente (lista larga, de consulta)

> **Para enviar al cliente usa `04_resumen_y_preguntas_cliente.md`**, que ya incorpora lo que hablaste por llamada: es una veterinaria, dan servicios, no imprimen, y cada sucursal debe trabajar sin internet.
> Esta lista es la primera versión, con el **porqué** de cada pregunta, y la dejo como respaldo. Algunas preguntas ya no aplican (por ejemplo, si imprimen factura o si el servidor es opcional).

Cada pregunta dice **por qué la preguntamos**, para que tú sepas qué decisión desbloquea.

Marca de prioridad: 🔴 bloquea el diseño o el precio · 🟡 importante, se puede cerrar durante el diseño · 🟢 detalle, se puede decidir luego.

---

## A. Las 10 que más importan (si solo alcanzan diez)

| # | Pregunta | Por qué la preguntamos |
|---|---|---|
| 1 🔴 | **¿Están dispuestos a pagar una mensualidad por un servidor en línea?** (Es un gasto pequeño y fijo, del orden de unos US$10 a US$40 al mes según la opción.) | Sin servidor, cada tienda solo vería sus propios datos y la oficina no podría ver el consolidado. Es la decisión que más cambia el sistema. |
| 2 🔴 | **¿Las 3 tiendas tienen internet estable durante todo el horario de venta?** ¿Qué hacen hoy cuando se cae? | Si el sistema vive en línea y se cae el internet, no se podría vender. Si hay cortes frecuentes, hay que diseñar un modo de respaldo. |
| 3 🔴 | **¿Qué productos manejan?** (medicinas, vacunas, alimento, accesorios) Aproximadamente, **¿cuántos productos distintos por tienda?** | Cambia las reglas: por ejemplo, el alimento se vende por libra o por saco, y los medicamentos por dosis o por frasco. *(Ya sabemos que es veterinaria.)* |
| 4 | ~~¿Entregan algún papel al cliente al vender?~~ | **Resuelta por llamada:** no emiten factura fiscal ni imprimen nada. |
| 5 🔴 | **¿Quién necesita ver la información de todas las tiendas, y desde dónde?** (oficina, celular, casa) | Define los usuarios de "oficina" y si hace falta que se abra desde el celular. |
| 6 🔴 | **Cuando dicen "costos" y "valor neto", ¿qué quieren ver exactamente?** (a) cuánto ganaron = lo vendido menos lo que costó la mercadería vendida; (b) cuánto compraron en el mes; (c) además restar gastos como alquiler y salarios. | Son tres cálculos distintos. El (c) requiere registrar gastos, que es más trabajo. |
| 7 🔴 | **¿Cuántas personas venden por tienda y cuántas cajas hay por tienda?** (¿una caja por tienda o varias a la vez?) ¿Hay turnos con distintos cajeros en el mismo día? | Define cuántos usuarios simultáneos y si el cierre es por día o por turno. |
| 8 🔴 | **¿Cuándo lo necesitan funcionando?** ¿Aceptan empezar con una sola tienda de prueba y luego pasar a las otras dos? | Define el plan. Probar en una tienda primero reduce mucho el riesgo. |
| 9 🟡 | **¿Qué computadoras tienen en cada tienda?** ¿Hay impresora de tickets, lector de código de barras, cajón de dinero? ¿Sus productos ya traen código de barras? | Si no hay lector, se busca por nombre o código escrito. Afecta la velocidad de venta. |
| 10 🟡 | **¿Tienen hoy un listado de productos en Excel (con costo, precio y vencimiento)?** ¿Quién lo cargaría al inicio? | Cargar el inventario inicial es el trabajo más grande del arranque. |

---

## B. Servidor, internet y las sucursales

- 🔴 **Sobre la alternativa de entrar desde la PC de la sucursal 1 a las otras:** para que funcione, esas computadoras tienen que estar **encendidas y con internet** todo el tiempo que alguien quiera consultar, y se configura un acceso especial. ¿Les sirve así, o prefieren la mensualidad del servidor? *(Les conviene saber que es la opción más frágil: si una PC se apaga, esa tienda desaparece del reporte.)*
- 🟡 Si **no** quieren pagar servidor: ¿aceptan que cada tienda trabaje sola y que el consolidado se arme **exportando un archivo** de cada tienda al final del día o del mes?
- 🟡 ¿Quieren que **una tienda pueda ver el inventario de otra** (por ejemplo, para saber dónde hay un producto), o solo la oficina?
- 🟡 ¿Se **pasan productos de una tienda a otra**? Si sí, ¿cómo lo anotan hoy? (Se llama traslado entre sucursales.)
- 🟢 ¿Cómo se llama cada sucursal? (para los usuarios: "Sucursal Centro", etc.)

## C. Productos e inventario

- 🔴 **¿Un mismo producto entra en lotes con distinta fecha de vencimiento?** (Casi seguro que sí.) ¿Todos los productos vencen o solo algunos?
- 🔴 **¿Con cuántos días de anticipación quieren la alerta de vencimiento?** Propuesta para discutir: **amarillo a 90 días, rojo a 30 días**, y "vencido" en rojo oscuro. ¿Igual para todos los productos o distinto por categoría?
- 🔴 **¿Qué debe pasar con un producto vencido?** (a) que el sistema **no deje venderlo**; (b) que deje pero avise; (c) que se pueda dar de baja registrándolo como pérdida.
- 🟡 **¿Quieren alerta cuando un producto se esté acabando?** Si sí: ¿cuál es el mínimo de cada producto, o un mínimo general?
- 🟡 **¿Los productos y los precios son los mismos en las 3 tiendas, o cada tienda tiene precios distintos?** (Si son iguales, se crea el producto una vez y las tres lo usan; el inventario sigue siendo propio de cada tienda.)
- 🟡 **¿Cómo se vende cada producto?** Por unidad, por caja, por blíster, por fracción (pastilla suelta). ¿Un producto puede venderse de dos formas?
- 🟡 **¿Quién registra la mercadería que entra?** ¿Con factura del proveedor? ¿Quieren llevar el **nombre del proveedor**?
- 🟡 **¿El costo de un mismo producto cambia entre compras?** (Si cambia, la ganancia se calcula lote por lote para que sea exacta.)
- 🟡 **¿Hacen conteos físicos?** ¿Quién ajusta cuando falta o sobra mercadería, y con qué motivo (daño, robo, error)?
- 🟡 **Medicamentos veterinarios:** ¿hay productos de control especial o requisitos de registro que el sistema deba respetar? ¿Hay que anotar el número de lote al vender?
- 🟢 ¿Los productos se agrupan en categorías (analgésicos, vitaminas, etc.)?

## D. Ventas y pagos

- 🟡 ¿Pueden pagar **una parte en efectivo y otra con tarjeta** en la misma venta?
- 🟡 Para **transferencia y tarjeta**, ¿hay que anotar el banco o el número de referencia/voucher?
- 🟡 ¿Dan **descuentos**? ¿Quién los autoriza y hay un máximo?
- 🟡 ¿Hay **ventas a crédito o "fiado"**? Si sí, hay que llevar cuentas por cobrar y eso es un módulo aparte.
- 🟡 ¿Registramos **datos del cliente** (nombre, RTN, teléfono) o las ventas son anónimas?
- 🟡 **¿Los precios ya incluyen impuesto (ISV)?** ¿Hay productos exentos? ¿Debe aparecer el ISV en el comprobante?
- 🟡 **Devoluciones y ventas mal hechas:** ¿quién puede anularlas y con qué motivo? (Propuesta: solo el encargado, y queda anotado quién y por qué.)

## E. Caja y cierre del día

- 🔴 **Explíquennos cómo cierran caja hoy**, paso a paso, y qué papel usan. (Queremos copiar lo que ya funciona.)
- 🟡 ¿Empiezan el día con un **fondo de caja** (efectivo inicial)?
- 🟡 ¿Se sacan **gastos o retiros de efectivo** de la caja durante el día (pagos a proveedores, almuerzos, depósitos)? ¿Se anotan?
- 🟡 ¿Quién cierra la caja y **quién puede corregir un cierre ya hecho**? (Propuesta: una vez cerrado, solo la oficina puede reabrirlo.)
- 🟢 ¿Cuándo termina el "día" de venta? (¿medianoche o al cerrar la tienda?)

## F. Reportes

- 🔴 Además de **venta total del día y del mes**, ¿qué otros reportes quieren? Ideas para elegir: ganancia por día/mes, ventas por método de pago, producto más vendido, productos que no se mueven, productos por vencer, inventario valorizado (cuánto dinero hay en mercadería), comparativo entre tiendas.
- 🟡 ¿Quieren ver los reportes **por tienda, por todas juntas, o por vendedor**?
- 🟡 ¿En qué formato: **en pantalla, PDF, Excel, impreso**? ¿Les gustaría recibirlos por correo o WhatsApp automáticamente?
- 🟢 ¿Algún reporte que hoy hagan a mano y les quite mucho tiempo?

## G. Usuarios y seguridad

- 🟡 ¿Qué tipos de usuario hay? Propuesta: **cajero** (vende), **encargado de tienda** (entradas, ajustes, anulaciones, cierre) y **administrador/oficina** (todo, todas las tiendas). ¿Les cuadra?
- 🟡 ¿El cajero **puede ver el costo y la ganancia** de los productos, o solo el precio de venta?
- 🟡 Cada tienda tenía "su propio usuario": ¿prefieren un usuario compartido por tienda, o **un usuario por persona**? (Por persona es lo recomendable, para saber quién vendió, anuló o ajustó.)

## H. Imagen y proyecto

- 🟢 Dijeron que quieren que "se mire bien". ¿Tienen **logo y colores** del negocio? ¿Hay algún sistema que hayan visto y les guste?
- 🟡 ¿Quién es la **persona que decide** y a quién le preguntamos dudas del día a día? (Mencionaron a David y a alguien llamado Julio/Yule; no sé qué rol tiene cada uno.)
- 🟡 ¿Quién va a **capacitar** al personal y cuánta gente necesita capacitación?
- 🟡 ¿Quieren un **servicio de soporte mensual** después de entregarlo (corrección de errores, cambios pequeños)?
- 🟡 ¿Tienen un **presupuesto de referencia**?
- 🟡 ¿Qué tan importante es no perder datos? (Se hacen **copias de seguridad automáticas**; ¿necesitan guardar historial por años?)

---

## Preguntas sobre servicios (se agregaron después de la llamada)

Están en `04_resumen_y_preguntas_cliente.md`, preguntas 12 a 16: lista de servicios y precios, si el precio varía, si registran su costo, quién presta el servicio y si quieren nombre de cliente y mascota.

## Preguntas para ti, antes de cotizar (no son para el cliente)

1. **¿Con qué tecnología quieres hacerlo?** Mi sugerencia está en `03_propuesta.md` (§7), pero si dominas otra pila, esa es mejor.
2. **¿Quién aloja y administra el servidor?** ¿Lo pagas tú y lo cobras dentro de la mensualidad, o la cuenta queda a nombre del cliente?
3. **¿De quién es el código?** ¿Se lo entregas al cliente o lo conservas y le cobras licencia/mensualidad? Si piensas venderlo a otros negocios más adelante, conviene diseñarlo desde ahora para varios clientes.
4. **¿Qué disponibilidad tienes?** La estimación de `03_propuesta.md` asume una persona a tiempo completo; a medio tiempo, se duplica el calendario.
5. **¿Cuánto cobras?** No tengo tu tarifa; la estimación de esfuerzo está en semanas para que le pongas precio.
6. **Si algún día el cliente necesita factura fiscal**, sería otro proyecto de alcance distinto. Por ahora no aplica.
