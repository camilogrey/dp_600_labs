# Parte I. Design scalable semantic models

En este ejercicio, exploré cómo utilizar funciones DAX y características avanzadas de Power BI para mejorar la flexibilidad y eficiencia de los modelos de datos, centrándome en los Grupos de Cálculo y los Parámetros de Campo. El objetivo era construir un modelo semántico escalable y altamente interactivo. 

## Preparación inicial
Antes de comenzar, descargué el archivo `15-Starter-Sales Analysis.pbix` desde el repositorio de Microsoft Learning, lo extraje en mi carpeta de Descargas y lo abrí en Power BI Desktop, ignorando cualquier advertencia emergente para aplicar cambios.

---

## 1. Trabajando con relaciones en el modelo

Mi primer paso fue cambiar a la **Vista Modelo** para revisar el diseño del esquema de datos.

![Imagen 1: Vista Modelo del diagrama de datos](sem_mod_imgs/1%20view%20model.png)
*(Nota: En esta imagen se observa el diagrama completo con las tablas Reseller, Region, Sales, Product, Date, Salesperson y Targets).*

![Imagen 2: Detalle de relaciones y columna Sales](sem_mod_imgs/2%20relations%20between%20sales%20&%20Date.png)
Al explorar el diagrama, noté una particularidad: existían **tres relaciones** entre la tabla `Date` y la tabla `Sales`. 

![Imagen 3: Relación activa con OrderDate](sem_mod_imgs/3.%20active%20relation%20date%20&%20Orderdate.png)
Al pasar el mouse por encima de cada línea de relación, identifiqué que la columna `Date` de la tabla `Date` es la clave única (lado "uno"). La conexión activa que se mostraba en línea sólida era con la columna `OrderDate` de la tabla `Sales`. Las otras dos relaciones, correspondientes a `DueDate` y `ShipDate`, estaban representadas con líneas punteadas (relaciones inactivas). Comprendí que la tabla `Date` actúa como una **dimensión de rol-playing**, capaz de filtrar los datos de ventas según la fecha de pedido, fecha de vencimiento o fecha de envío.

---

## 2. Visualizando ventas por fecha (usando la relación activa)

Para poner esto a prueba, me dirigí a la **Vista Informe**.

![Imagen 4: Selección del objeto visual Tabla](sem_mod_imgs/4%20add%20visualization.png)
Desde el panel de **Visualizaciones**, seleccioné el objeto visual de **Tabla**.

![Imagen 5: Añadiendo Year y Total Sales a la tabla](sem_mod_imgs/5%20visualization%20year%20vs%20total%20sales.png)
En el panel de **Datos**, expandí la tabla `Date` y arrastré el campo `Year` a la tabla visual. Luego, expandí la tabla `Sales` y arrastré `Total Sales`. La tabla generada me mostró el total de ventas agrupado por año fiscal. En ese momento, corroboré que el dato representaba el año en que se *realizó el pedido*.

---

## 3. Utilizando relaciones inactivas con USERELATIONSHIP

Sabía que quería analizar las ventas basándome en la fecha de envío, pero sin tener que crear tablas duplicadas ni cambiar la relación activa permanentemente.

![Imagen 6: Creando una nueva medida en la tabla Sales](sem_mod_imgs/6%20%20create%20new%20measure%20from%20sale%20stable%20.png)
Hice clic derecho sobre la tabla `Sales` en el panel de Datos y seleccioné **New measure**.

![Imagen 7: Escribiendo el DAX de Sales Shipped](sem_mod_imgs/7%20measure%20creation.png)
En la barra de fórmulas, reemplacé el texto con la siguiente medida DAX, utilizando la función `CALCULATE` y, dentro de ella, `USERELATIONSHIP` para activar de forma transitoria el enlace con `Sales[ShipDate]`:
"""
```dax
Sales Shipped = 
CALCULATE (
    SUM ('Sales'[Sales]),
    USERELATIONSHIP('Date'[Date], 'Sales'[ShipDate])
)
```
![Imagen 8: ](sem_mod_imgs/8%20measured%20just%20created%20in%20visualization.png)
Agregué esta nueva medida a mi objeto visual de tabla. Al ampliar las columnas, pude comparar los resultados. La fila de totales era idéntica en ambas columnas, pero los montos por año fiscal diferían. Esto era correcto, ya que un pedido hecho en el año fiscal 2017 (Total Sales) podía perfectamente haberse enviado en el 2018 (Sales Shipped).
Sin embargo, me di cuenta de una limitación: si tuviera 10 medidas de ventas y 3 fechas distintas, tendría que crear 30 medidas. Esto sería tedioso y difícil de mantener.

## 4. Creando grupos de cálculo para inteligencia de tiempo

Para solucionar esa ineficiencia, se implemento un Grupo de Cálculo.

![Imagen 9: ](sem_mod_imgs/9%20create%20a%20calculation%20group.png)
Regresé a la Vista Modelo y, desde la pestaña superior (Inicio o Modelado), seleccioné la opción Calculation group.

![Imagen 10: ](sem_mod_imgs/10%20change%20name%20to%20calculation%20group%20&%20item%20and%20update%20formula.png)
Renombré el grupo a Time Calculations y la columna de cálculo a Yearly Calculations. Posteriormente, empecé a crear los ítems de cálculo. El primero lo nombré Year-to-Date (YTD) y le asigné la siguiente fórmula:

```
Year-to-Date (YTD) = CALCULATE(SELECTEDMEASURE(), DATESYTD('Date'[Date]))
```

![Imagen 11: ](sem_mod_imgs/11%20New%20calculation%20item%20on%20calculation%20item.png)
Haciendo clic derecho sobre la sección Calculation items, seleccioné New calculation item para añadir el segundo ítem.

![Imagen 12: ](sem_mod_imgs/12%20dax%20code%20on%20new%20calculaton%20item.png)
El segundo ítem fue Previous Year (PY), con la función PREVIOUSYEAR:

```
dax
Previous Year (PY) = CALCULATE(SELECTEDMEASURE(), PREVIOUSYEAR('Date'[Date]))
```
![Imagen 13: ](sem_mod_imgs/13%20new%20calculation%20item.png)
Para el tercer ítem, Year-over-Year (YoY) Growth, utilicé variables DAX (VAR) para calcular el crecimiento interanual con SAMEPERIODLASTYEAR:
```
dax
Year-over-Year (YoY) Growth = 
VAR MeasurePriorYear =
CALCULATE(
    SELECTEDMEASURE(),
    SAMEPERIODLASTYEAR('Date'[Date])
)
RETURN
DIVIDE(
    (SELECTEDMEASURE() - MeasurePriorYear),
    MeasurePriorYear
)
```
![Imagen 14: ](sem_mod_imgs/14%20enable%20dynamic%20format%20string%20on%20YoY%20calculation%20item.png)
El ítem YoY Growth devolvía un porcentaje. Para que el formato se ajustara automáticamente en el visual, fui al panel de Propiedades de ese ítem, activé el interruptor Dynamic format string y escribí la cadena de formato "0.##%". Esto aseguró que el resultado se mostrara correctamente sin necesidad de formatear manualmente cada medida base.

## 5. Aplicando el grupo de cálculo a las medidas
Una vez configurado el grupo, fui a probarlo en mi informe.

![Imagen 15: ](sem_mod_imgs/15%20report%20preview%20.png)
Me cambié a la pestaña Overview en el informe y seleccioné la matriz que ya se encontraba en el lienzo. Desde el panel de Datos, arrastré la columna Yearly Calculations (perteneciente a mi grupo Time Calculations) a la sección Columns del panel de visualizaciones.

![Imagen 16: ](sem_mod_imgs/16%20replacing%20matriz%20values%20with%20yearly%20calculations%20.png)

El resultado fue espectacular. La matriz pasó a mostrar todas mis medidas (Total Sales, Profit, etc.) cruzadas por todos mis cálculos temporales (YTD, PY, YoY Growth) de manera simultánea y sin tener que crear medidas redundantes.

Cierre de la primera parte:
El uso de los grupos de cálculo ha demostrado ser una herramienta de modelado extremadamente poderosa para la inteligencia de tiempo. Sin embargo, al observar la matriz final, me di cuenta de que tener tantas columnas de información a la vez puede resultar abrumador y difícil de leer para el usuario. Para solucionar esto y permitir que el usuario seleccione qué métrica desea visualizar en un momento dado, la segunda parte del laboratorio (que adjuntaré en la siguiente consulta) utiliza Parámetros de Campo, permitiendo una experiencia de usuario mucho más limpia y dinámica.

# Parte II: Implementando Parámetros de Campo (Field Parameters)

Al finalizar la primera parte, me encontré con un dilema: tenía una matriz muy poderosa con los Grupos de Cálculo aplicados, pero mostraba **todas** las métricas (Total Sales, Profit, Profit Margin, etc.) cruzadas por todos los cálculos de inteligencia de tiempo al mismo tiempo. Esto hacía que el informe fuera difícil de leer y poco amigable para el usuario final. 

La solución a este problema, según el siguiente paso del taller, era implementar un **Parámetro de Campo (Field Parameter)**.

## 1. Creando el Parámetro de Campo

Para darle al usuario la capacidad de elegir qué métrica ver, me dirigí a la cinta de opciones principal.

![Imagen 17: Botón New Parameter en la pestaña Modeling](sem_mod_imgs/17%20new%20parameter%20in%20Modelling.png)
Fui a la pestaña **Modelado (Modeling)** y desplegué el menú **Nuevo parámetro (New parameter)**. En lugar de un rango numérico, seleccioné la opción **Campos (Fields)**.

![Imagen 18: Configuración del diálogo de parámetros](sem_mod_imgs/18%20configure%20parameter.png)
Se abrió un cuadro de diálogo de configuración. Aquí realicé los siguientes ajustes:
*   En el desplegable *What will your variable adjust?*, dejé seleccionado **Fields**.
*   En el campo *Name*, nombré al parámetro como **Sales Figures**.
*   En la sección central, añadí las medidas existentes en mi modelo: **Total Sales**, **Profit**, **Profit Margin** y **Orders**.
*   **El paso clave:** Marqué la casilla **Add slicer to this page** (Agregar segmentador de datos a esta página). Esto me generaría automáticamente un menú desplegable o lista de selección en el informe. Finalmente, hice clic en **Create**.

## 2. Aplicando el Parámetro de Campo al visual y al slicer

Una vez creado, el nuevo parámetro apareció en el panel de Datos como una tabla calculada (bajo el nombre `Sales Figures`), y en el lienzo del informe apareció el segmentador de datos ya configurado.

![Imagen 19: El segmentador de datos recién creado en el informe](sem_mod_imgs/19%20use%20parameters%20to%20see%20changes%20in%20matriz.png)
El nuevo slicer `Sales Figures` se generó con las opciones "Total Sales", "Profit", "Profit Margin" y "Orders". 

![Imagen 20: Arrastrando el parámetro a la matriz](sem_mod_imgs/20%20remove%20value%20from%20matriz%20and%20include%20only%20the%20parameter%20sales%20figures%20.png)
Acto seguido, seleccioné la matriz donde había aplicado mis Grupos de Cálculo. Desde el panel de Datos, arrastré el campo `Sales Figures` a la sección **Values** (o Columnas) del panel de visualizaciones, reemplazando las medidas individuales.

![Imagen 21: Código DAX subyacente del Parámetro de Campo](sem_mod_imgs/21%20how%20metrics%20change%20by%20filtering%20parameter.png)
Por curiosidad, revisé la barra de fórmulas. Power BI genera automáticamente una tabla DAX para manejar este parámetro. La sintaxis es bastante clara, utilizando la función `NAMEOF` para referirse a las medidas:
```dax
Sales Figures = {
    ("Total Sales", NAMEOF('Sales'[Total Sales]), 0),
    ("Profit", NAMEOF('Sales'[Profit]), 1),
    ("Profit Margin", NAMEOF('Sales'[Profit Margin]), 2),
    ("Orders", NAMEOF('Sales'[Orders]), 3)
}
```

## 3.  Probando la interactividad y el poder de la combinación

Con todo configurado, llegó el momento de probar la interactividad del modelo.

![Imagen 22:](sem_mod_imgs/22%20replace%20total%20sales%20fro%20parameter%20sales%20figures%20&%20create%20a%20slicer%20and%20inseet%20the%20parameter.png)
En el segmentador de datos, hice clic en la opción Orders. Instantáneamente, la matriz se actualizó para mostrar únicamente la columna de datos referente al número de pedidos. Lo más impresionante fue que los Grupos de Cálculo de inteligencia de tiempo siguieron funcionando perfectamente sobre esta métrica. Ahora podía ver el "Previous Year (PY)" de los pedidos, el "Year-over-Year (YoY)" de los pedidos y el "Year-to-Date (YTD)" de los pedidos sin tener que crear medidas específicas para cada uno.

![Imagen 23: ](sem_mod_imgs/23%20before%20editing%20parameter%20sales%20figures.png)
Para ir un paso más allá, tomé el mismo parámetro Sales Figures y lo arrastré al campo "Y-axis" (Eje Y) de un gráfico de barras en otra pestaña del informe. Al igual que con la matriz, el gráfico se volvió dinámico y respondía en tiempo real a la selección del slicer, permitiéndome analizar ventas, beneficios o pedidos mes a mes con un solo clic.

![Imagen 24:](sem_mod_imgs/24%20after%20editing%20the%20parameter.png)

## Conclusión final de mi laboratorio

Completar este ejercicio me permitió entender el verdadero potencial de los modelos semánticos escalables en Power BI.

Combinando dos técnicas avanzadas (Grupos de Cálculo y Parámetros de Campo), logré un resultado increíblemente eficiente:

Evité decenas de medidas redundantes (no necesité crear 30 variaciones de "Total Sales", "Profit", etc., para cada fecha).

Mantuve la inteligencia de tiempo aplicada de forma automática a cualquier métrica que el usuario eligiera.

Creé una interfaz de usuario limpia y dinámica, donde el usuario tiene el control total de qué datos quiere visualizar, sin saturar la pantalla con información innecesaria.

Este enfoque no solo ahorra tiempo en el desarrollo, sino que hace que el modelo sea mucho más fácil de mantener y actualizar en el futuro.

