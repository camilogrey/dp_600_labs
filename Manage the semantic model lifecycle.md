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

1. **Navegación al lakehouse y cambio al endpoint SQL**  
   Desde el workspace `SalesLifecycle-dev`, seleccioné el lakehouse **SalesLakehouse**. En la esquina superior derecha, cambié la vista de **Lakehouse** a **SQL analytics endpoint** para poder crear el modelo semántico basado en las tablas.

   > ![Cambio al SQL analytics endpoint](lifecycle_lab_img/9.%20Navigate%20SalesLakehouse%20Switch%20to%20the%20SQL%20analytics%20endpoint.png)

2. **Inicio de la creación del modelo semántico**  
   En la barra de herramientas del endpoint SQL, hice clic en **New semantic model**. Se abrió el panel de configuración donde completé los siguientes campos:

   - **Direct Lake semantic model name**: `SalesData`
   - **Workspace**: `SalesLifecycle-dev` (mi workspace de desarrollo)
   - **Storage mode**: `Direct Lake on SQL` (seleccioné esta opción para consultar directamente los datos en el lakehouse)
   - **Tables**: marqué la opción **Select all** para incluir las tres tablas (`products`, `customers`, `sales`)

   > ![Inicio de la creación del modelo semántico](lifecycle_lab_img/10.%20start%20creating%20new%20semantic%20model.png)  
   > ![Configuración del modelo semántico](lifecycle_lab_img/11.%20create%20configuration%20of%20semantic%20model.png)

3. **Confirmación y espera de la creación**  
   Hice clic en **Confirm** y esperé unos minutos hasta que el modelo semántico se creara por completo. Una vez finalizado, el modelo `SalesData` apareció en la lista de elementos del workspace, confirmando que estaba disponible para su uso.

   > ![Validación de la creación del modelo semántico](lifecycle_lab_img/12.%20Semantic%20model%20creation%20validation.png)

---

**Nota: ** Con este paso, ya tengo un modelo semántico basado en el lakehouse, que podré gestionar y validar utilizando SemPy desde el notebook en los siguientes pasos del laboratorio.

