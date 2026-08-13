# Gestión del ciclo de vida del modelo semántico

## Crear los Workspace dev y Prod. Crear tablas mediante un notebook

### 1. Creación de los workspaces de desarrollo y producción
Accedí a Microsoft Fabric y, desde la sección **Workspaces**, creé dos espacios de trabajo con el mismo nombre base pero distintos sufijos:

- **SalesLifecycle-dev** (para desarrollo)
- **SalesLifecycle-prod** (para producción)

Ambos se crearon con capacidad de Fabric (usé la opción de prueba). El workspace `-dev` apareció vacío, listo para comenzar.

> ![Workspace de desarrollo vacío](lifecycle_lab_img/1.%20saleslife%20cycle%20development.png)  
> ![Workspace de producción vacío](lifecycle_lab_img/2.%20saleslife%20cycle%20production.png)

### 2. Importación del notebook del laboratorio
Descargué el archivo `21b-manage-semantic-model-lifecycle.ipynb` desde la URL proporcionada y lo guardé localmente. En el workspace `SalesLifecycle-dev`, utilicé la opción **Import** y seleccioné **Notebook** para cargar el archivo. Una vez importado, el notebook apareció en la lista de elementos del workspace.

> ![Importación del notebook](lifecycle_lab_img/3.%20imported%20python%20code%20as%20fabric%20notebook%20.png)

### 3. Creación del lakehouse dentro del workspace de desarrollo
Desde el workspace `SalesLifecycle-dev`, hice clic en **+ New item** y elegí **Lakehouse**. Asigné el nombre **SalesLakehouse** y esperé a que se creara. El lakehouse se abrió mostrando su estructura vacía (carpetas Tables y Files).

> ![Lakehouse creado en el workspace de desarrollo](lifecycle_lab_img/4.%20created%20lakehouse%20inside%20dev%20workspace.png)

### 4. Apertura del notebook existente
En el lakehouse, desde la barra de herramientas, seleccioné **Open notebook > Existing notebook** y elegí el notebook importado (`21b-manage-semantic-model-lifecycle`). El notebook se abrió en el entorno de Fabric.

> ![Opción para abrir un notebook existente](lifecycle_lab_img/5.%20open%20a%20notebook%20in%20dev%20workspace%20lakehouse.png)  
> ![Selección del notebook importado](lifecycle_lab_img/6.%20open%20existing%20notebook.png)

### 5. Ejecución de la primera celda para generar datos de muestra
Dentro del notebook, localicé la sección **Generate sample data** y ejecuté la primera celda de código. Este código crea tres tablas (`products`, `customers`, `sales`) con datos de ejemplo y las guarda en el lakehouse como tablas Delta. No ejecuté ninguna celda adicional, tal como se indicaba en las instrucciones.

> ![Ejecución de la primera celda y refresco de tablas](lifecycle_lab_img/7.%20once%20first%20code%20line%20is%20runned%20refresh%20tables.png)

### 6. Verificación de las tablas creadas
Después de ejecutar el código, en el explorador del lakehouse, hice clic derecho sobre la carpeta **Tables** y seleccioné **Refresh**. Confirmé que las tablas `products`, `customers` y `sales` aparecían correctamente. El mensaje de salida indicaba que se habían creado 10 filas en productos, 20 en clientes y 200 en ventas, con algunos valores nulos intencionales para pruebas posteriores.

> ![Tablas creadas y mensaje de confirmación](lifecycle_lab_img/8.%20tables%20created%20by%20the%20notebook%20code%20.png)

---

**Nota final:** Con estos pasos completé la configuración inicial del entorno, la importación del notebook, la creación del lakehouse y la generación de los datos de muestra. El siguiente paso sería crear el modelo semántico y aplicar la validación con SemPy.

## Crear el módelo semantico

### 1. **Navegación al lakehouse y cambio al endpoint SQL**  
   Desde el workspace `SalesLifecycle-dev`, seleccioné el lakehouse **SalesLakehouse**. En la esquina superior derecha, cambié la vista de **Lakehouse** a **SQL analytics endpoint** para poder crear el modelo semántico basado en las tablas.

   > ![Cambio al SQL analytics endpoint](lifecycle_lab_img/9.%20Navigate%20SalesLakehouse%20Switch%20to%20the%20SQL%20analytics%20endpoint.png)

### 2. **Inicio de la creación del modelo semántico**  
   En la barra de herramientas del endpoint SQL, hice clic en **New semantic model**. Se abrió el panel de configuración donde completé los siguientes campos:

   - **Direct Lake semantic model name**: `SalesData`
   - **Workspace**: `SalesLifecycle-dev` (mi workspace de desarrollo)
   - **Storage mode**: `Direct Lake on SQL` (seleccioné esta opción para consultar directamente los datos en el lakehouse)
   - **Tables**: marqué la opción **Select all** para incluir las tres tablas (`products`, `customers`, `sales`)

   > ![Inicio de la creación del modelo semántico](lifecycle_lab_img/10.%20start%20creating%20new%20semantic%20model.png)  
   > ![Configuración del modelo semántico](lifecycle_lab_img/11.%20create%20configuration%20of%20semantic%20model.png)

### 3. **Confirmación y espera de la creación**  
   Hice clic en **Confirm** y esperé unos minutos hasta que el modelo semántico se creara por completo. Una vez finalizado, el modelo `SalesData` apareció en la lista de elementos del workspace, confirmando que estaba disponible para su uso.

   > ![Validación de la creación del modelo semántico](lifecycle_lab_img/12.%20Semantic%20model%20creation%20validation.png)

---

**Nota: ** Con este paso, ya tengo un modelo semántico basado en el lakehouse, que podré gestionar y validar utilizando SemPy desde el notebook en los siguientes pasos del laboratorio.

## Validación y corrección de un modelo semántico con SemPy

### 1. Regreso al notebook
Una vez creado el modelo semántico `SalesData`, navegué de vuelta al workspace `SalesLifecycle-dev` y abrí el notebook `21b-manage-semantic-model-lifecycle` que había importado anteriormente. El notebook ya contenía las celdas de código para la validación y corrección del modelo.

> ![Lista de elementos en el workspace, incluyendo el notebook y el modelo semántico](lifecycle_lab_2_img/1.%20back%20to%20the%20notebook.png)

### 2. Validación del modelo con SemPy
Dentro del notebook, localicé la sección **Validate the semantic model with SemPy** y ejecuté cada celda en orden, revisando los resultados:

- **Listar tablas del modelo:**  
  Utilicé `fabric.list_tables("SalesData")` para confirmar que el modelo era accesible. El resultado mostró las tres tablas esperadas: `customers`, `sales` y `products`.

  > ![Código de importación y listado de tablas](lifecycle_lab_2_img/2.%20begin%20the%20process%20of%20validation%20of%20semantic%20model%20with%20Sempy.png)  
  > ![Salida con las tres tablas](lifecycle_lab_2_img/3.%20three%20tables%20output%20(first%20code%20chunck).png)

- **Listar columnas de todas las tablas:**  
  Ejecuté la celda que muestra el nombre, tipo de datos y tabla padre de cada columna. El resultado fue un DataFrame con 21 columnas y 15 filas (aunque el número exacto de columnas puede variar), lo que me permitió inspeccionar la estructura del modelo sin necesidad de abrir Power BI Desktop.

  > ![Análisis de columnas con estadísticas básicas](lifecycle_lab_2_img/4.%2021%20columns%20and%20their%20EDA%20automatic%20analysis.png)  
  > ![Más estadísticas de columnas](lifecycle_lab_2_img/5.%20More%20statstics.png)

- **Verificar valores nulos y duplicados:**  
  Ejecuté la celda que cuenta nulos en todas las columnas y duplicados en la clave primaria de ventas. Los resultados mostraron que la columna `CustomerKey` en la tabla `sales` tenía 3 valores nulos, y que no había duplicados en `SalesKey`. Esto indicaba que algunas filas de ventas no podrían vincularse con la tabla de clientes, causando posibles espacios en blanco en los informes.

  > ![Resultado de nulos y duplicados](lifecycle_lab_2_img/6.%20null%20values%20in%20customerkey%20.png)

- **Descubrir relaciones potenciales:**  
  Utilicé la función `find_relationships` de SemPy para detectar posibles relaciones entre las tablas. El análisis encontró una relación de muchos a uno (m:1) entre la tabla `sales` (columna `ProductKey`) y la tabla `products` (columna `ProductKey`), con una cobertura completa (1.0 en ambas direcciones). Esto sugería que esa relación debería existir en el modelo.

  > ![Relación descubierta entre sales y products](lifecycle_lab_2_img/7.%20one%20many-to-one%20relationship%20on%20ProductKey%20between%20the%20sales%20and%20products%20tables..png)

- **Evaluar una consulta DAX para verificar cálculos:**  
  Ejecuté una consulta DAX que agrupaba las ventas por categoría de producto. El resultado mostró el mismo total para las tres categorías (`Bikes`, `Clothing`, `Accessories`), lo cual era incorrecto. Esto confirmaba que el modelo no tenía relaciones definidas, por lo que el motor DAX no podía filtrar las ventas por categoría.

  > ![Resultado de la consulta DAX mostrando totales idénticos](lifecycle_lab_2_img/8.%20The%20identical%20values%20%20the%20semantic%20model%20has%20no%20relationships%20DAX%20engine%20can’t%20filter%20sales%20by%20category.png)

### 3. Corrección del modelo con SemPy
Con la validación completada y el problema identificado, pasé a la sección **Fix the semantic model with SemPy** para agregar las relaciones faltantes de forma programática.

- **Agregar relaciones mediante TOM:**  
  Ejecuté la celda que abre una conexión de lectura/escritura al modelo mediante la API de Tabular Object Model (TOM). El código creó dos relaciones de muchos a uno:

  - `sales[ProductKey]` → `products[ProductKey]`
  - `sales[CustomerKey]` → `customers[CustomerKey]`

  Los cambios se guardaron automáticamente al cerrar el contexto, y luego se refrescó el modelo para activar las nuevas relaciones.

  > ![Código para fijar el modelo con TOM](lifecycle_lab_2_img/9.%20now%20we%20atar%20fixing%20the%20model.png)  
  > ![Confirmación de relaciones agregadas y modelo refrescado](lifecycle_lab_2_img/10.%20%20creates%20two%20many-to-one%20relationships%20using%20the%20TOM%20API.png)

- **Re-ejecutar la consulta DAX:**  
  Una vez aplicadas las relaciones, volví a ejecutar la misma consulta DAX que antes daba resultados idénticos. Esta vez, los totales fueron diferentes para cada categoría, confirmando que las relaciones funcionaban correctamente y que el modelo ahora podía filtrar ventas por producto.

  > ![Resultados correctos de la consulta DAX con relaciones activas](lifecycle_lab_2_img/11.%20rerun%20the%20DAX%20query%20resukts%20are%20diferent.png)

---

**Nota:** Con estos pasos completé la validación y corrección del modelo semántico utilizando SemPy. El modelo ahora tiene las relaciones necesarias y devuelve resultados correctos en las consultas DAX.

