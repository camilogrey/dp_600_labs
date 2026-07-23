## Creación de un modelo dimensional en Fabric

1. **Creación del warehouse**  
   Dentro de mi workspace `real-time-intelligence`, hice clic en **+ New item** y seleccioné **Warehouse** en la sección **Store Data**. Asigné el nombre **ContosoDW**.  
   Tras unos segundos, el warehouse se creó y se abrió en el navegador mostrando una vista vacía, lista para empezar a trabajar.

   > ![Warehouse vacío recién creado](img_lab2/part1/01.png)

2. **Creación de la tabla de hechos (fact table)**  
   En la barra de herramientas del warehouse, seleccioné **New SQL query** para abrir un editor de consultas.  
   Escribí el siguiente código T‑SQL para definir la tabla `f_Sales`, que almacenará las transacciones de ventas:

"""
CREATE TABLE f_Sales
(
DateKey INT NOT NULL,
StoreKey INT NOT NULL,
ProductKey INT NOT NULL,
CustomerKey INT NOT NULL,
Quantity INT NOT NULL,
UnitPrice DECIMAL(10,2) NOT NULL,
SalesAmount DECIMAL(10,2) NOT NULL,
DiscountAmount DECIMAL(10,2) NOT NULL
);
"""

3. **Ejecución de la consulta**  
Hice clic en el botón **▷ Run** para ejecutar el script. La operación se completó sin errores.

> ![Editor de consultas con el código y el botón Run](1.png)

4. **Verificación de la tabla creada**  
Para confirmar que la tabla se había creado correctamente, pulsé el botón **Refresh** en la barra de herramientas para actualizar la vista del explorador.  
Luego, en el panel **Explorer**, expandí la ruta **Schemas > dbo > Tables** y efectivamente apareció la tabla `f_Sales`.

> ![Explorador mostrando la tabla f_Sales creada](3.png)

---

**Nota:** La tabla de hechos utiliza el prefijo `f_` para identificarse como tal, y no incluye una clave primaria, siguiendo las buenas prácticas para tablas de hechos en modelos dimensionales.

