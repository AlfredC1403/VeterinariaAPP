# Sistema de ventas e inventario para sus sucursales: resumen y preguntas

Este documento junta lo que entendimos de lo que necesitan, cómo se les puede resolver y lo que necesitamos que nos confirmen para empezar. Si algo de lo que dice aquí no es así, corríjanlo.

---

## 1. Lo que entendimos

- Son una **veterinaria con 3 sucursales**. Hoy todo se lleva **a mano** en un libro.
- **Venden productos** (medicinas y otros) y también **dan servicios** (por ejemplo, consultas).
- **No emiten factura fiscal ni trabajan con la SAR.** El sistema sería solo para **control interno y reportes**; no imprime facturas.
- Cada sucursal debe trabajar **por su cuenta**: su propio inventario, sus propias ventas y su propio cierre de caja. Los datos de una no se mezclan con los de otra.
- La **oficina principal** quiere poder ver las ventas, los cierres y el inventario de todas las sucursales.
- Cada sucursal debe poder **seguir trabajando aunque se caiga el internet**, y después enviar sus datos sola cuando vuelva.
- El inventario debe **actualizarse solo**: si se vende algo, se rebaja; si entra producto, se suma.
- Cada producto lleva **nombre, código, precio de costo, precio de venta y fecha de vencimiento**, y el sistema debe **avisar cuando algo esté por vencer**.
- Cada venta lleva su **método de pago**, y al final del día queda todo listo para **cerrar y cuadrar la caja**.
- Quieren **reportes**: ventas totales del día y del mes, y más.
- Debe ser **sencillo y verse bien**.

---

## 2. Lo que se les puede hacer

### 2.1 Cómo funcionaría, sucursal por sucursal

```
  Sucursal 1: computadora con el sistema ─┐
  Sucursal 2: computadora con el sistema ─┼─ cuando hay internet, ─►  SERVIDOR CENTRAL  ◄─ Oficina principal
  Sucursal 3: computadora con el sistema ─┘   envía sola lo nuevo      (base de datos y        (ve todo)
                                                                        copias de seguridad)
```

- En cada sucursal el sistema **guarda todo en su propia computadora**. Aunque no haya internet, se puede vender, registrar entradas de mercadería y cerrar caja.
- Cuando **vuelve el internet**, el sistema envía solo lo que se hizo al servidor. Nadie tiene que hacer nada.
- La **oficina** ve el resumen de todas las sucursales, y junto a cada una aparece **a qué hora fue su última actualización**. Si una sucursal estuvo horas sin internet, la oficina verá sus datos cuando se reconecte.
- Cada usuario ve **solo lo de su sucursal**. Solo el administrador ve todas.
- El servidor guarda **copias de seguridad**: si se daña una computadora, lo que ya se había enviado no se pierde.

### 2.2 Productos y alertas

- Cada producto tiene nombre, código, precio de costo, precio de venta y, cuando entra mercadería, su **lote con la fecha de vencimiento**.
- Cuando se vende, el sistema descuenta **primero lo que vence antes**.
- **Alerta de vencimiento:** el producto cambia de color según cuánto falte. *Propuesta:* amarillo a 90 días, **rojo a 30 días o menos**, y "vencido" si ya pasó la fecha. Los días se pueden cambiar.
- **Alerta de producto por acabarse:** cada producto tiene un mínimo, y el sistema lo marca **en rojo** cuando la existencia llega a ese mínimo.
- Las dos alertas aparecen en el panel de inicio, cada una con su etiqueta ("Por vencer" y "Por acabarse") para distinguirlas.

### 2.3 Servicios

- Un servicio (consulta, vacunación, baño, cirugía, etc.) se registra con **nombre, código y precio de venta**.
- **El costo es opcional:** si quieren saber cuánto les cuesta dar ese servicio, lo anotan; si no, se deja vacío y solo se usa el precio.
- Un servicio **no lleva inventario ni fecha de vencimiento**.
- Al vender, el servicio se agrega **igual que un producto**. En una misma cuenta puede haber servicios y productos (por ejemplo, una consulta y una vacuna).
- Los reportes **separan lo vendido en productos y lo vendido en servicios**.

### 2.4 Ventas y cierre de caja

- Se registra la venta y su **método de pago**. *Propuesta:* efectivo, transferencia y tarjeta. Pueden ajustarlo.
- Al **cerrar caja**, el sistema muestra cuánto se vendió por cada método, cuánto efectivo debería haber, y calcula la diferencia con lo que se cuenta.
- Una venta mal hecha **no se borra: se anula**, con el motivo y quién lo hizo. Si era un producto, vuelve al inventario.
- **No imprime factura.** Si alguna vez quieren un comprobante interno, se puede agregar.

### 2.5 Reportes

1. Cierre de caja del día de cada sucursal.
2. **Ventas del día, del mes o de las fechas que elijan**, por sucursal y de todas juntas.
3. Ventas separadas por **productos y servicios**.
4. **Ganancia** (lo vendido menos lo que costó). En los servicios se calcula solo si anotaron el costo.
5. Inventario actual y cuánto dinero hay en mercadería.
6. Productos **por vencer** y vencidos.
7. Productos **por acabarse**.
8. Productos y servicios **más vendidos**.
9. Comparación entre sucursales.

Se pueden ver en pantalla y sacar en **PDF o Excel**.

### 2.6 Lo que NO incluye

Factura fiscal ni conexión con la SAR · contabilidad · planilla · ficha médica o historial de las mascotas (podría ser una etapa posterior). **Es un sistema de control interno y no sustituye ninguna obligación fiscal del negocio.**

---

## 3. Por qué hace falta un servidor y cómo se paga

Para que las 3 sucursales envíen sus datos y la oficina los vea todos juntos, la información tiene que llegar a **un lugar central en internet**: un servidor que guarda la base de datos y las copias de seguridad. Eso tiene un costo mensual.

- **Es un pago mensual pequeño** que cubre el servidor, las copias de seguridad y su mantenimiento. El monto exacto se les confirma al responder estas preguntas.
- **No necesitan tarjeta de crédito ni abrir cuentas.** Pagan por **transferencia** a quien desarrolla el sistema, y esa persona contrata y administra el servidor.
- **Si no quieren pagar el servidor**, es posible que cada sucursal trabaje sola, pero entonces: la oficina no vería nada en conjunto, no habría envío automático de datos, y los totales habría que juntarlos a mano con archivos de cada tienda. No lo recomendamos.

---

## 4. Preguntas para confirmar

**Marque con ★ las que necesitamos antes de cualquier otra cosa.**

### Sucursales y equipos

1. ★ **¿Cuántas computadoras se usan para vender en cada sucursal?** ¿Pueden estar dos personas vendiendo al mismo tiempo en una misma sucursal? *(Cambia cómo se arma el trabajo sin internet.)*
2. ★ **¿Qué computadoras tienen?** ¿Son Windows? ¿Cuántos años tienen más o menos? ¿Tienen lector de código de barras o impresora de tickets?
3. ★ **¿Cómo es el internet en cada sucursal?** ¿Se cae seguido? ¿Cuántas veces por semana y por cuánto tiempo?
4. ¿Cómo se llama cada sucursal?

### Productos

5. ★ **¿Qué tipo de productos manejan?** (medicinas, vacunas, alimento, accesorios, otros) ¿Cuántos productos distintos hay aproximadamente por sucursal?
6. **¿Algún producto se vende por peso o por fracción?** (por ejemplo, alimento por libra, o media dosis)
7. **¿Todos los productos llevan vencimiento?** ¿Les sirve la propuesta de amarillo a 90 días y rojo a 30? ¿Qué debe hacer el sistema con un producto **vencido**: no dejar venderlo, o solo avisar?
8. Para la **alerta de producto por acabarse**: ¿el mínimo es distinto para cada producto, o usamos uno general? ¿Nos pueden dar el mínimo de los productos principales?
9. **¿Los productos y los precios son los mismos en las 3 sucursales?** ¿Quién crea un producto nuevo: la oficina o cada sucursal?
10. ¿Tienen hoy una lista de productos en **Excel**, con costo, precio y vencimiento? Si no, ¿quién la armaría?
11. ¿Hay algún **requisito de registro o control** de medicamentos veterinarios que el sistema deba llevar?

### Servicios

12. ★ **¿Qué servicios dan y cuánto cobran por cada uno?** (consulta, vacunación, baño, cirugía, desparasitación, hospitalización, otros)
13. ¿El precio de un servicio es siempre el mismo, o a veces cambia según el caso? ¿Quién puede cambiarlo al cobrar?
14. ¿Quieren anotar **cuánto les cuesta** cada servicio, o solo el precio?
15. ¿Quieren saber **quién dio el servicio** (qué veterinario) para tener reportes por persona?
16. ¿Quieren llevar el **nombre del cliente y de su mascota** en cada venta, o solo la venta?

### Ventas, pagos y caja

17. ★ **¿Qué métodos de pago aceptan?** ¿Efectivo, transferencia, tarjeta (¿tienen datáfono?), otros? ¿Un cliente puede pagar una parte con uno y otra parte con otro?
18. ¿Hacen **descuentos**? ¿Quién los puede dar?
19. ¿Venden **a crédito o "fiado"**?
20. ★ **¿Cómo cierran caja hoy?** Cuéntennos paso a paso. ¿Empiezan con un fondo de efectivo? ¿Sacan efectivo de la caja para gastos durante el día? ¿Hay varios turnos con distintos cajeros?
21. ¿Quién puede **anular una venta**?

### Reportes

22. Además de las ventas del día y del mes, ¿cuáles de los reportes de la sección 2.5 les interesan más? ¿Quieren ver la **ganancia**? Para saber cuánto les queda **después de gastos** (alquiler, salarios, luz), habría que registrar también esos gastos. ¿Les interesa?
23. ¿Quién necesita ver los reportes de todas las sucursales y desde dónde? (oficina, casa, celular)

### Usuarios y proyecto

24. **¿Quiénes usarán el sistema y qué debe poder hacer cada uno?** *Propuesta:* **cajero** (vende), **encargado** (entradas de mercadería, anulaciones, cierre) y **administrador** (todo, todas las sucursales). ¿El cajero puede ver costos y ganancias?
25. ★ **¿Están de acuerdo con el pago mensual del servidor por transferencia?**
26. ★ **¿Quién es la persona que decide** y con quién hablamos del día a día?
27. ¿Para cuándo lo necesitan? ¿Les parece bien **empezar con una sola sucursal de prueba** y luego pasar a las otras dos?
28. ¿Quieren un **soporte mensual** después de la entrega, para dudas, correcciones y cambios pequeños?

---

Con sus respuestas preparamos el plan de trabajo, el calendario y el precio.
