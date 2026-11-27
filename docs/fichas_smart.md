# Fichas S.M.A.R.T. de los KPIs — Invacom

Los 8 KPIs del dashboard cumplen el criterio S.M.A.R.T.: cada uno es **E**specífico, **M**edible, **A**lcanzable, **R**elevante y tiene un **T**iempo definido. Las metas salen de los datos reales del DW, que van de julio 2013 a mayo 2015.

| KPI | Responsable | Página | Meta | Frecuencia |
| --- | --- | --- | --- | --- |
| 1 · Crecimiento anual de ventas | John | Ventas | ≥ 10 % | Mensual, contra el mismo mes del año anterior |
| 2 · Variación mensual de ventas | John | Ventas | ≥ 0 % | Mensual |
| 3 · Ticket promedio por pedido | Emily | Clientes | ≥ 1 500 | Mensual |
| 4 · Tasa de recompra | Emily | Clientes | ≥ 50 % | Trimestral |
| 5 · Tasa de descuento | Juanda | Productos y descuentos | ≤ 6 % | Mensual |
| 6 · Concentración en los 10 clientes principales | Juanda | Productos y descuentos | ≤ 45 % | Anual |
| 7 · Entregas a tiempo | Ryan | Logística | ≥ 95 % | Mensual |
| 8 · Flete sobre ventas | Ryan | Logística | ≤ 5 % | Mensual |

En los KPIs 5, 6 y 8 **menos es mejor**: el valor cumple cuando queda por debajo de la meta.

## KPI 1 · Crecimiento anual de ventas

**Responsable:** John · **Página:** Ventas · **Meta:** ≥ 10 %

Mide cuánto crecen las ventas netas de un periodo frente al mismo periodo del año anterior.

```
KPI1 Crecimiento anual % =
VAR actual = [Ventas netas]
VAR anterior = [Ventas mismo mes año anterior]   -- CALCULATE + SAMEPERIODLASTYEAR
RETURN IF ( NOT ISBLANK ( anterior ), DIVIDE ( actual - anterior, anterior ) )
```

| Criterio | Desglose |
| --- | --- |
| S · Específico | Crecimiento de las ventas netas de Invacom (monto bruto menos descuentos) contra el mismo periodo del año anterior. |
| M · Medible | (ventas del periodo − ventas del mismo periodo del año anterior) ÷ ventas del año anterior. Se calcula con SAMEPERIODLASTYEAR sobre la tabla de fechas. |
| A · Alcanzable | Desde julio 2014, las ventas de casi todos los meses superan las del mismo mes de 2013, así que un 10 % es una meta conservadora. |
| R · Relevante | Le dice a la gerencia si el negocio está creciendo, que es la base para planear compras, inventario y metas comerciales. |
| T · Temporal | Se mide cada mes contra el mismo mes del año anterior; en el dashboard se filtra por año y mes. |

En el dashboard: tarjeta verde o roja según la meta y gráfico de líneas de ventas netas contra el mismo mes del año anterior. Los meses de 2013 no tienen comparación porque los datos empiezan en julio 2013.

## KPI 2 · Variación mensual de ventas

**Responsable:** John · **Página:** Ventas · **Meta:** ≥ 0 %

Mide si las ventas netas de un mes subieron o bajaron frente al mes anterior.

```
KPI2 Variación mensual % =
VAR actual = [Ventas netas]
VAR anterior = [Ventas mes anterior]   -- CALCULATE + PREVIOUSMONTH
RETURN IF ( HASONEVALUE ( dim_tiempo[Periodo] ) && NOT ISBLANK ( anterior ),
            DIVIDE ( actual - anterior, anterior ) )
```

| Criterio | Desglose |
| --- | --- |
| S · Específico | Cambio porcentual de las ventas netas de un mes contra el mes inmediatamente anterior. |
| M · Medible | (ventas del mes − ventas del mes anterior) ÷ ventas del mes anterior. Usa PREVIOUSMONTH y solo se calcula cuando hay un único mes en el filtro (HASONEVALUE). |
| A · Alcanzable | En el histórico, la mayoría de los meses cierra con variación positiva; la meta pide no retroceder, no crecer un monto fijo. |
| R · Relevante | Detecta a tiempo una caída en ventas, antes de que se note en el resultado anual. |
| T · Temporal | Se mide cada mes, en cuanto cierra el mes. |

En el dashboard: tarjeta verde o roja según la meta y gráfico de columnas por mes, verde si sube y rojo si baja. Mayo 2015 sale muy negativo porque ese mes solo tiene datos de los primeros días.

## KPI 3 · Ticket promedio por pedido

**Responsable:** Emily · **Página:** Clientes · **Meta:** ≥ 1 500

Mide cuánto vende Invacom, en promedio, en cada pedido.

```
KPI3 Ticket promedio = DIVIDE ( [Ventas netas], [Pedidos] )   -- Pedidos = DISTINCTCOUNT ( pedido_id )
Ticket mes anterior = CALCULATE ( [KPI3 Ticket promedio], DATEADD ( dim_tiempo[fecha], -1, MONTH ) )
```

| Criterio | Desglose |
| --- | --- |
| S · Específico | Ventas netas divididas entre la cantidad de pedidos distintos del periodo. |
| M · Medible | Ventas netas ÷ pedidos. En el total del DW da 1 525,05 (1 265 792,76 en ventas y 830 pedidos). |
| A · Alcanzable | El ticket anual pasó de 1 369 (2013) a 1 512 (2014) y 1 632 (2015), así que 1 500 ya se alcanza y se puede sostener. |
| R · Relevante | Un ticket más alto aumenta los ingresos sin necesidad de conseguir más clientes ni más pedidos. |
| T · Temporal | Se mide cada mes y se compara contra el mes anterior con DATEADD. |

## KPI 4 · Tasa de recompra

**Responsable:** Emily · **Página:** Clientes · **Meta:** ≥ 50 % por trimestre

Mide qué parte de los clientes que compraron en el periodo hizo dos pedidos o más.

```
KPI4 Tasa de recompra % =
VAR clientes = VALUES ( fact_ventas[cliente_key] )
VAR recurrentes = COUNTROWS ( FILTER ( clientes, CALCULATE ( DISTINCTCOUNT ( fact_ventas[pedido_id] ) ) >= 2 ) )
RETURN DIVIDE ( recurrentes, COUNTROWS ( clientes ) )
```

| Criterio | Desglose |
| --- | --- |
| S · Específico | Porcentaje de clientes activos del trimestre que hicieron 2 pedidos o más en ese mismo trimestre. |
| M · Medible | Clientes con 2+ pedidos ÷ clientes con al menos 1 pedido. Se calcula con VALUES, FILTER y COUNTROWS. |
| A · Alcanzable | En los trimestres del histórico la recompra ronda el 50 %, así que la meta pide mantener o mejorar ese nivel. |
| R · Relevante | Mide la fidelización: retener a un cliente cuesta menos que conseguir uno nuevo. |
| T · Temporal | Se mide por trimestre, porque en un solo mes pocos clientes alcanzan a repetir. |

## KPI 5 · Tasa de descuento

**Responsable:** Juanda · **Página:** Productos y descuentos · **Meta:** ≤ 6 % (menos es mejor)

Mide qué parte del valor de lista se pierde en descuentos.

```
KPI5 Tasa de descuento % = DIVIDE ( [Monto descuento], [Monto bruto] )   -- ambos con SUM
```

| Criterio | Desglose |
| --- | --- |
| S · Específico | Monto total descontado sobre el monto bruto vendido (precio × cantidad, antes del descuento). |
| M · Medible | Monto descuento ÷ monto bruto. En el total del DW da 6,5 %. |
| A · Alcanzable | La tasa bajó de 8,0 % (2013) a 6,3 % (2014) y 6,2 % (2015); bajar a 6 % sigue esa tendencia. |
| R · Relevante | Cada punto de descuento sale directo del margen; controlarlo protege la rentabilidad. |
| T · Temporal | Se mide cada mes, con formato rojo cuando pasa la meta. |

## KPI 6 · Concentración en los 10 clientes principales

**Responsable:** Juanda · **Página:** Productos y descuentos · **Meta:** ≤ 45 % (menos es mejor)

Mide qué parte de las ventas depende de los 10 clientes que más compran.

```
Ventas top 10 clientes = CALCULATE ( [Ventas netas], TOPN ( 10, VALUES ( dim_cliente[cliente_key] ), [Ventas netas] ) )
KPI6 Concentración top 10 % = DIVIDE ( [Ventas top 10 clientes], [Ventas netas] )
```

| Criterio | Desglose |
| --- | --- |
| S · Específico | Participación de los 10 clientes con más ventas netas dentro del total de ventas netas del periodo. |
| M · Medible | Ventas de los 10 primeros clientes ÷ ventas netas totales. Usa TOPN para elegir a los 10. En el total del DW da 45 %. |
| A · Alcanzable | Por año quedó en 50 % (2013), 46,5 % (2014) y 54,5 % (2015); el acumulado ya está en 45 %, así que la meta es exigente pero posible si crecen los clientes medianos. |
| R · Relevante | Mide el riesgo de depender de pocos clientes: si uno de los grandes se va, las ventas caen mucho. |
| T · Temporal | Se revisa por año y en el acumulado, porque en periodos cortos hay pocos clientes y el valor sube solo. |

## KPI 7 · Entregas a tiempo

**Responsable:** Ryan · **Página:** Logística · **Meta:** ≥ 95 %

Mide qué parte de los pedidos enviados salió a más tardar en la fecha que pidió el cliente.

```
Pedidos enviados = CALCULATE ( [Pedidos], fact_ventas[a_tiempo] <> "Pendiente" )
Pedidos a tiempo = CALCULATE ( [Pedidos], fact_ventas[a_tiempo] = "Sí" )
KPI7 Entregas a tiempo % = DIVIDE ( [Pedidos a tiempo], [Pedidos enviados] )
```

| Criterio | Desglose |
| --- | --- |
| S · Específico | Pedidos con fecha de envío igual o anterior a la fecha requerida, sobre los pedidos ya enviados. Los pendientes no cuentan. |
| M · Medible | Pedidos a tiempo ÷ pedidos enviados. En el DW: 772 a tiempo, 37 tarde y 21 pendientes, o sea 95,4 %. Usa la columna `a_tiempo` que se agregó al DW para este proyecto. |
| A · Alcanzable | Por año quedó en 95,4 % (2013), 94,6 % (2014) y 96,8 % (2015); la meta pide sostener el nivel que ya se logra. |
| R · Relevante | Entregar tarde afecta la satisfacción del cliente y la recompra (KPI 4). |
| T · Temporal | Se mide cada mes y por transportista. |

Dato de contexto para la página: el promedio de días de entrega es 8,49 (medida con AVERAGEX).

## KPI 8 · Flete sobre ventas

**Responsable:** Ryan · **Página:** Logística · **Meta:** ≤ 5 % (menos es mejor)

Mide cuánto cuesta el envío en proporción a lo vendido.

```
Flete total = SUM ( fact_ventas[flete_prorrateado] )
KPI8 Flete sobre ventas % = DIVIDE ( [Flete total], [Ventas netas] )
```

| Criterio | Desglose |
| --- | --- |
| S · Específico | Costo total de flete sobre las ventas netas del periodo. El flete de cada pedido se reparte entre sus líneas. |
| M · Medible | Flete total ÷ ventas netas. En el total del DW da 5,1 % (64 942,70 de flete sobre 1 265 792,76 en ventas). |
| A · Alcanzable | El valor real está apenas 0,1 puntos arriba de la meta; bajar a 5 % se logra eligiendo mejor el transportista en los pedidos pequeños. |
| R · Relevante | El flete es el principal costo logístico; si crece más rápido que las ventas, se come el margen. |
| T · Temporal | Se mide cada mes y por transportista. |