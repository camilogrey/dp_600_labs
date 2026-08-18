# Creación de una ontología con Fabric IQ

## Set up del workspace, files and tables

### 1. Creación del workspace
Accedí a Microsoft Fabric mediante el navegador e inicié sesión con mis credenciales. En la barra lateral izquierda, seleccioné el icono de **Workspaces** y creé un nuevo workspace con un nombre de mi elección, asegurándome de seleccionar un modo de licencia que incluyera capacidad de Fabric (usé la opción de prueba). El workspace quedó vacío, listo para comenzar.

### 2. Creación del lakehouse con datos de muestra
Desde el workspace, seleccioné **+ New item > Lakehouse** y asigné el nombre **`LamnaHealthcareLH`**. Una vez creado, procedí a cargar los datos de muestra.

### 3. Carga de los archivos CSV al lakehouse
Descargué el archivo `sample-data.zip` y extraje los cinco archivos CSV en mi equipo local:
- `Hospitals.csv`
- `Departments.csv`
- `Rooms.csv`
- `Patients.csv`
- `VitalSignEquipment.csv`

> [Archivos CSV](ontology_img/sample-data/)

En el lakehouse, desde la vista principal, seleccioné **Upload files** y subí los cinco archivos simultáneamente. Una vez completada la carga, verifiqué que los archivos aparecían en la carpeta **Files**.

> ![Archivos CSV cargados en el lakehouse](ontology_img/1.Upload%205%20csv%20files%20to%20Lakehouse%20and%20load%20then%20into%20tables%20each%20one.png)

### 4. Conversión de los archivos a tablas Delta
Para cada archivo CSV, desde el menú contextual (puntos suspensivos) seleccioné **Load to Tables > New table**. Configuré cada tabla con el nombre del archivo en minúsculas (sin extensión) y verifiqué que la opción **Use header for column names** estuviera activada. El proceso se repitió para los cinco archivos, creando las tablas:
- `hospitals`
- `departments`
- `rooms`
- `patients`
- `vitalsignequipment`

Luego, en el explorador, confirmé que las cinco tablas aparecían en la sección **Tables** del lakehouse.

### 5. Creación del eventhouse para datos de streaming
Para almacenar los datos de signos vitales en tiempo real, creé un nuevo eventhouse. En el workspace, seleccioné **+ New item > Eventhouse** y asigné el nombre **`LamnaHealthcareEH`**. El eventhouse se creó con una base de datos KQL predeterminada del mismo nombre.

> ![Creación del eventhouse](ontology_img/2.%20Create%20an%20eventhouse.png)

### 6. Ingesta de los datos de signos vitales
Dentro del eventhouse, seleccioné la base de datos KQL `LamnaHealthcareEH` y, desde la vista principal, elegí la opción **Get data** para iniciar el proceso de ingesta.

> ![Opción Get data en el eventhouse](ontology_img/3.%20get%20data%20not%20the%20eventhouse%20.png)

En el asistente de ingesta, configuré los siguientes parámetros:
- **Source**: Local file
- **Destination table**: Creé una nueva tabla con el nombre `VitalSignsReadings`
- **File**: Seleccioné el archivo `VitalSignsReadings.csv` que había descargado previamente

> ![Configuración de la tabla y selección del archivo](ontology_img/4.%20give%20a%20name%20to%20the%20table%20and%20%20then%20upload%20the%20respective%20file.png)

Continué con el asistente, manteniendo las opciones predeterminadas, y finalicé la ingesta. El proceso se completó con éxito.

> ![Resumen de la ingesta completada](ontology_img/5.%20uploaded%20completed.png)

### 7. Verificación de la tabla en la base de datos KQL
En la base de datos KQL, confirmé que la tabla `VitalSignsReadings` aparecía correctamente, con los datos cargados (20 filas iniciales) y disponible para consultas.

> ![Tabla VitalSignsReadings en la base de datos KQL](ontology_img/6.%20KQL%20database%20show%20the%20VitalSignsReadings%20table.png)

---

**Nota** Con estos pasos completé la preparación del entorno para la creación de la ontología. El lakehouse contiene las tablas de entidades (hospitales, departamentos, habitaciones, pacientes, equipos) y el eventhouse contiene los datos de streaming de signos vitales, listos para ser vinculados en la ontología de Fabric IQ.
