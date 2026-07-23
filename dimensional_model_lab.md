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

> ![Editor de consultas con el código y el botón Run](img_lab2/part1/1.png)

4. **Verificación de la tabla creada**  
Para confirmar que la tabla se había creado correctamente, pulsé el botón **Refresh** en la barra de herramientas para actualizar la vista del explorador.  
Luego, en el panel **Explorer**, expandí la ruta **Schemas > dbo > Tables** y efectivamente apareció la tabla `f_Sales`.

> ![Explorador mostrando la tabla f_Sales creada](img_lab2/part1/3.png)

---

**Nota:** La tabla de hechos utiliza el prefijo `f_` para identificarse como tal, y no incluye una clave primaria, siguiendo las buenas prácticas para tablas de hechos en modelos dimensionales.

## Agregar restricciones de tabla (claves primarias y foráneas)

1. **Creación de la consulta SQL para las restricciones**  
   Desde el warehouse `ContosoDW`, abrí una nueva consulta SQL (botón **New SQL query** en la barra de herramientas). En el editor, pegué el siguiente código que agrega las claves primarias a las tablas de dimensiones y las claves foráneas a la tabla de hechos `f_Sales`:

       -- Claves primarias en dimensiones
       ALTER TABLE d_Date
           ADD CONSTRAINT PK_d_Date PRIMARY KEY NONCLUSTERED (DateKey) NOT ENFORCED;

       ALTER TABLE d_Store
           ADD CONSTRAINT PK_d_Store PRIMARY KEY NONCLUSTERED (StoreKey) NOT ENFORCED;

       ALTER TABLE d_Product
           ADD CONSTRAINT PK_d_Product PRIMARY KEY NONCLUSTERED (ProductKey) NOT ENFORCED;

       ALTER TABLE d_Customer
           ADD CONSTRAINT PK_d_Customer PRIMARY KEY NONCLUSTERED (CustomerKey) NOT ENFORCED;

       -- Claves foráneas en la tabla de hechos
       ALTER TABLE f_Sales
           ADD CONSTRAINT FK_Sales_Date FOREIGN KEY (DateKey)
               REFERENCES d_Date(DateKey) NOT ENFORCED;

       ALTER TABLE f_Sales
           ADD CONSTRAINT FK_Sales_Store FOREIGN KEY (StoreKey)
               REFERENCES d_Store(StoreKey) NOT ENFORCED;

       ALTER TABLE f_Sales
           ADD CONSTRAINT FK_Sales_Product FOREIGN KEY (ProductKey)
               REFERENCES d_Product(ProductKey) NOT ENFORCED;

       ALTER TABLE f_Sales
           ADD CONSTRAINT FK_Sales_Customer FOREIGN KEY (CustomerKey)
               REFERENCES d_Customer(CustomerKey) NOT ENFORCED;

2. **Ejecución del script**  
   Hice clic en el botón **▷ Run** para ejecutar todas las instrucciones. La operación se completó sin errores; en el panel de mensajes aparecieron los identificadores de cada declaración confirmando que las restricciones se habían agregado correctamente.

   > ![Captura de la ejecución de las restricciones](img_lab2/part2/2.png)

3. **Verificación del esquema**  
   Una vez ejecutado el script, actualicé la vista del explorador (botón **Refresh**) y, aunque las restricciones no son validadas (NOT ENFORCED), quedaron registradas como metadatos que documentan las relaciones entre las tablas. Esto permitirá que, en un futuro, Power BI pueda detectar automáticamente las relaciones al crear un modelo semántico desde el warehouse.

---

**Nota:** Las restricciones `NOT ENFORCED` son típicas en los data warehouses de Fabric, ya que la integridad referencial se gestiona normalmente en los procesos de ETL/ELT y no es necesario que el motor las valide en tiempo de ejecución, pero sí sirven como documentación del modelo estrella.

## Carga de datos de muestra

1. **Creación de una nueva consulta SQL**  
   En el warehouse `ContosoDW`, desde la barra de herramientas, seleccioné **New SQL query** para abrir un editor de consultas.

2. **Inserción de los datos de muestra**  
   En el editor, pegué el siguiente bloque de código que inserta registros en todas las tablas de dimensiones y en la tabla de hechos `f_Sales`:

   ```sql
   -- Carga de la dimensión fecha
   INSERT INTO d_Date VALUES
   (20260105, '2026-01-05', 2026, 1, 1, 'January', 5, 'Monday', 2026, 3, 0, 1),
   (20260112, '2026-01-12', 2026, 1, 1, 'January', 12, 'Monday', 2026, 3, 0, 1),
   (20260209, '2026-02-09', 2026, 1, 2, 'February', 9, 'Monday', 2026, 3, 0, 1),
   (20260302, '2026-03-02', 2026, 1, 3, 'March', 2, 'Monday', 2026, 3, 0, 1),
   (20260406, '2026-04-06', 2026, 2, 4, 'April', 6, 'Monday', 2026, 4, 0, 1),
   (20260504, '2026-05-04', 2026, 2, 5, 'May', 4, 'Monday', 2026, 4, 0, 1);

   -- Carga de la dimensión tienda
   INSERT INTO d_Store VALUES
   (1, 'ST-001', 'Contoso Downtown', 'Flagship', 'Seattle', 'Washington', 'United States', 'West', '2020-03-15', '2026-01-01', '9999-12-31', 1),
   (2, 'ST-002', 'Contoso Mall', 'Standard', 'Portland', 'Oregon', 'United States', 'West', '2021-07-01', '2026-01-01', '9999-12-31', 1),
   (3, 'ST-003', 'Contoso Central', 'Standard', 'Chicago', 'Illinois', 'United States', 'Central', '2019-11-20', '2026-01-01', '9999-12-31', 1),
   (4, 'ST-004', 'Contoso Plaza', 'Express', 'New York', 'New York', 'United States', 'East', '2022-01-10', '2026-01-01', '9999-12-31', 1);

   -- Carga de la dimensión producto
   INSERT INTO d_Product VALUES
   (1, 'MB-PRO', 'Mountain Bike Pro', 'AdventureWorks', 'Mountain Bikes', 'Bikes', 1200.00, '2026-01-01', '9999-12-31', 1),
   (2, 'RB-ELT', 'Road Bike Elite', 'AdventureWorks', 'Road Bikes', 'Bikes', 900.00, '2026-01-01', '9999-12-31', 1),
   (3, 'HL-STD', 'Cycling Helmet', 'SafeRide', 'Helmets', 'Accessories', 25.00, '2026-01-01', '9999-12-31', 1),
   (4, 'WB-STD', 'Water Bottle', 'HydroGear', 'Bottles', 'Accessories', 5.00, '2026-01-01', '9999-12-31', 1),
   (5, 'LK-STD', 'Bike Lock', 'SecureLock', 'Locks', 'Accessories', 15.00, '2026-01-01', '9999-12-31', 1);

   -- Carga de la dimensión cliente
   INSERT INTO d_Customer VALUES
   (1, 'Jordan Rivera', 'Premium', 'Seattle', 'Washington', 'United States', 'Gold', '2023-06-15'),
   (2, 'Alex Chen', 'Standard', 'Portland', 'Oregon', 'United States', 'Silver', '2024-01-20'),
   (3, 'Sam Patel', 'Premium', 'Chicago', 'Illinois', 'United States', 'Gold', '2022-11-05'),
   (4, 'Taylor Kim', 'Budget', 'New York', 'New York', 'United States', 'Bronze', '2025-03-12'),
   (5, 'Morgan Lee', 'Standard', 'Seattle', 'Washington', 'United States', 'Silver', '2024-08-30');

   -- Carga de la tabla de hechos (ventas)
   INSERT INTO f_Sales VALUES
   (20260105, 1, 1, 1, 1, 1500.00, 1500.00, 0.00),
   (20260105, 1, 3, 1, 2, 35.00, 70.00, 5.00),
   (20260112, 2, 2, 2, 1, 1100.00, 1100.00, 100.00),
   (20260112, 2, 4, 2, 3, 8.00, 24.00, 0.00),
   (20260209, 3, 1, 3, 2, 1500.00, 3000.00, 150.00),
   (20260209, 3, 5, 3, 1, 22.00, 22.00, 0.00),
   (20260302, 1, 2, 5, 1, 1100.00, 1100.00, 0.00),
   (20260302, 4, 3, 4, 4, 35.00, 140.00, 10.00),
   (20260406, 2, 1, 2, 1, 1500.00, 1500.00, 75.00),
   (20260504, 3, 4, 3, 5, 8.00, 40.00, 0.00);

3. **Ejecución de la carga**  
Hice clic en el botón ▷ Run para ejecutar el script. La operación se completó correctamente; en el panel de mensajes se confirmó que se habían insertado los registros en cada tabla, mostrando el número de filas afectadas (por ejemplo, 6 registros en la tabla de hechos y las respectivas inserciones en las dimensiones).

3. **Verificación de los datos**
Para confirmar que los datos se cargaron adecuadamente, actualicé la vista del explorador (botón Refresh) y verifiqué que las tablas ya contenían los registros insertados. También ejecuté una consulta rápida de selección sobre f_Sales para visualizar las primeras filas y confirmar que los valores coincidían con los insertados.

> ![Captura de la ejecución de carga](img_lab2/part3/3.png)