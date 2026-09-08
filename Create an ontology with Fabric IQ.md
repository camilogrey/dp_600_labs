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

## Creación de un modelo semántico para ontología

### 1. Creación del modelo semántico desde el lakehouse
En el workspace, abrí el lakehouse **LamnaHealthcareLH** que había creado anteriormente. Desde la cinta de opciones del lakehouse, seleccioné **New semantic model** para comenzar a crear el modelo que servirá como base para la ontología.

En el cuadro de diálogo **New semantic model**, configuré los siguientes parámetros:
- **Direct Lake semantic model name**: `LamnaHealthcareModel`
- Seleccioné las cinco tablas disponibles: `hospitals`, `departments`, `rooms`, `patients`, `vitalsignequipment`
- Hice clic en **Confirm** para crear el modelo.

El modelo se abrió directamente, mostrando las cinco tablas en el área de trabajo del modelo.

> ![Creación del modelo semántico desde el lakehouse](ontology_img2/1.%20creation%20of%20the%20lakehouse%20LamnaHealthcareLH%20and%20start%20creating%20a%20new%20semantic%20model.png)

### 2. Definición de las relaciones entre tablas
Para que el modelo refleje las relaciones del mundo real (un departamento pertenece a un hospital, una habitación es parte de un departamento, un paciente está asignado a una habitación y un equipo está asignado a un paciente), procedí a definir las relaciones.

En la cinta de opciones del modelo, seleccioné **Manage relationships** y luego **+ New relationship**. Creé las siguientes cuatro relaciones, todas con cardinalidad **Many to one (*:1)** y filtro cruzado **Both**:

| Desde tabla | Columna desde | Hacia tabla | Columna hacia |
|-------------|---------------|-------------|---------------|
| departments | HospitalId    | hospitals   | HospitalId    |
| rooms       | DepartmentId  | departments | DepartmentId  |
| patients    | CurrentRoomId | rooms       | RoomId        |
| vitalsignequipment | PatientId | patients | PatientId |

Cada relación la guardé antes de crear la siguiente. Al finalizar, verifiqué que el panel **Manage relationships** mostrara exactamente 4 relaciones activas, tal como se esperaba. Luego hice clic en **Close**.

> ![Ventana Manage relationships con las 4 relaciones activas](ontology_img2/2.%20manage%20relationships.png)

### 3. Visualización del modelo y diagrama de relaciones
Una vez definidas las relaciones, el modelo semántico mostró el diagrama completo con las cinco tablas conectadas mediante las relaciones establecidas. El diagrama refleja claramente la estructura jerárquica: hospitales → departamentos → habitaciones → pacientes → equipos de signos vitales.

> ![Diagrama de relaciones del modelo semántico](ontology_img2/3.%20MER.png)

### 4. Cierre y reapertura del modelo
En la barra de navegación superior, seleccioné la **×** junto a `LamnaHealthcareModel` para cerrar el modelo y volver a la lista de elementos del workspace. Luego, desde la lista de elementos, hice clic en el botón de puntos suspensivos (**…**) junto al modelo y seleccioné **Open semantic model** para volver a abrirlo y confirmar que las relaciones se habían guardado correctamente.

---

**Nota** El modelo semántico `LamnaHealthcareModel` quedó configurado con las cinco tablas y las relaciones necesarias para representar la estructura de datos de la organización sanitaria. Estas relaciones serán la base para la creación de la ontología en Fabric IQ, donde cada relación se convertirá en un tipo de relación entre entidades.

##  Generación de una ontología

### 1. Generación de la ontología desde el modelo semántico
Con el modelo semántico `LamnaHealthcareModel` abierto, en la cinta de opciones superior seleccioné **Generate Ontology** para iniciar el proceso de creación de la ontología. En el cuadro de diálogo, elegí mi workspace y asigné el nombre **`LamnaHealthcareOntology`** (sin espacios ni guiones, solo letras, números y guiones bajos). Hice clic en **Create** y esperé unos momentos mientras el sistema generaba la ontología.

> ![Botón Generate Ontology en la cinta del modelo semántico](ontology_img2/4.%20Generate%20Ontology.png)  
> ![Asignación de nombre a la ontología](ontology_img2/5.%20Ontology%20name.png)

El sistema generó 5 tipos de entidad (`hospitals`, `departments`, `rooms`, `patients`, `vitalsignequipment`) con sus propiedades, y 4 tipos de relación basados en las relaciones del modelo semántico.

### 2. Revisión y adición de claves de entidad
Cada tipo de entidad necesita una propiedad clave que identifique de forma única cada instancia. Revisé cada entidad para verificar que la clave estuviera definida.

- **Entidad `hospitals`**: Verifiqué que tuviera la clave `HospitalId` definida. Aparecía correctamente.

> ![Revisión de la clave en la entidad hospitals](ontology_img2/6.%20review%20if%20ontolgy%20have%20gereate%20its%20restive%20relation%20key%20in%20thi%20case%20we%20review%20the%20entity%20hospitals.png)  
> ![Detalle de la clave HospitalId en hospitals](ontology_img2/7.%20relationship%20key%20reviewed.png)

- **Entidad `vitalsignequipment`**: Al revisar, observé que no tenía definida una clave. En el panel de configuración, el campo **Entity type key** mostraba "None". 

> ![Falta de clave en vitalsignequipment](ontology_img2/8.%20lack%20of%20relationship%20key%20defintion.png)

Procedí a definirla. Seleccioné **Define entity type key**, y en el cuadro de diálogo elegí la propiedad **EquipmentId** como clave, guardando el cambio.

> ![Definición de la clave EquipmentId](ontology_img2/9,%20lets%20define%20the%20key.png)  
> ![Selección de EquipmentId como clave](ontology_img2/10.%20defined%20key.png)  
> ![Clave EquipmentId confirmada en la entidad](ontology_img2/11.%20key%20confirmed.png)

Repetí este proceso para las entidades `departments`, `rooms` y `patients`, verificando que tuvieran sus respectivas claves (`DepartmentId`, `RoomId`, `PatientId`). Todas estaban correctamente definidas.

### 3. Verificación y configuración de relaciones
A continuación, revisé las relaciones generadas y configuré aquellas que no tenían enlace a datos.

En el lienzo de la ontología, seleccioné la relación entre `departments` y `hospitals` (llamada `departments_has_hospitals`). Al abrir el panel de configuración, verifiqué que la relación tuviera configurada la fuente de datos:

- **Workspace**: mi workspace
- **Lakehouse**: `LamnaHealthcareLH`
- **Schema**: `dbo`
- **Table**: `Departments` (esta tabla contiene tanto el identificador del departamento como la referencia al hospital)

Luego, en las asignaciones de entidad:
- **Source entity type**: `Departments` → columna `DepartmentId`
- **Target entity type**: `Hospitals` → columna `HospitalId`

Guardé la configuración.

> ![Verificación de la relación departments_has_hospitals](ontology_img2/13.%20verify%20relationships%20between%20tables%20clcick%20in%20relationship.png)  
> ![Detalle de la relación configurada](ontology_img2/14.%20relationship%20checked.png)

Repetí el proceso para las tres relaciones restantes, utilizando los siguientes valores:

| Relación | Tabla origen | Columna origen (entidad origen) | Columna destino (entidad destino) |
|----------|--------------|----------------------------------|-----------------------------------|
| rooms_has_departments | Rooms | RoomId | DepartmentId |
| patients_has_rooms | Patients | PatientId | CurrentRoomId |
| vitalsignequipment_has_patients | VitalSignEquipment | EquipmentId | PatientId |

Para cada una, seleccioné la tabla correspondiente y asigné las columnas según la tabla.

> ![Lista de relaciones en el explorador](ontology_img2/12.%20add%20relationships.png)

### 4. Resultado final
Con todas las claves y relaciones configuradas, la ontología `LamnaHealthcareOntology` quedó completa. Ahora comprende la estructura completa del modelo de datos de la organización sanitaria: los hospitales contienen departamentos, los departamentos contienen habitaciones, los pacientes están asignados a habitaciones y los equipos de signos vitales están asignados a pacientes. La ontología está lista para ser utilizada en aplicaciones de análisis y búsqueda semántica.

---

**Nota** completé la generación automática de la ontología, revisé y agregué las claves de entidad faltantes, y configuré manualmente las relaciones con sus respectivos enlaces a datos. El sistema ahora puede conectar todos los elementos del modelo semántico y del eventhouse (aunque los datos de signos vitales en tiempo real se agregarán en un paso posterior).

## Adición de un enlace de series temporales a VitalSignEquipment

### 1. Acceso a la entidad VitalSignEquipment

En el explorador de la ontología, dentro de la lista de tipos de entidad, seleccioné **vitalsignequipment**. Esta entidad actualmente solo tiene propiedades estáticas provenientes del lakehouse (`EquipmentId`, `PatientId`, `EquipmentType`, `MonitoringStartDate`). El objetivo era agregar un segundo enlace para conectar los datos de series temporales de signos vitales desde el eventhouse.

> ![Selección de la entidad vitalsignequipment en el explorador](ontology_img2/15.%20bidding%20with%20this%20entity.png)

### 2. Inicio de la adición de un enlace de datos
En el panel de configuración de la entidad, fui a la pestaña **Bindings** (o bien, utilicé la opción **Add data to entity type**). Allí se mostraba el enlace estático existente desde el lakehouse. Seleccioné **Add data binding** para agregar un segundo enlace.

> ![Botón Add data binding en la entidad](ontology_img2/16.%20add%20data%20bidding%20in%20thi%20entity.png)

### 3. Selección del origen de datos (Eventhouse)
En el cuadro de diálogo **OneLake catalog**, navegué hasta mi workspace y localicé el eventhouse **LamnaHealthcareEH**. Lo seleccioné y luego elegí la tabla **VitalSignsReadings** que contenía los datos de series temporales. Hice clic en **Next** para continuar.

> ![Selección del eventhouse LamnaHealthcareEH en el catálogo](ontology_img2/17.%20connect%20with%20event%20house.png)  
> ![Selección de la tabla VitalSignsReadings](ontology_img2/18.%20selecting%20the%20table%20to%20connect.png)

### 4. Configuración del enlace como series temporales
En la siguiente pantalla, para **Binding type** seleccioné **Time series**. Luego, en **Source data timestamp column**, elegí la columna **Timestamp** que contiene la marca de tiempo de cada lectura. Esto le indica al sistema que los datos son temporales.

> ![Configuración del timestamp y tipo de enlace](ontology_img2/19.%20add%20timestamp%20and%20add%20enetity%20type%20property.png)

### 5. Mapeo de columnas estáticas y de series temporales
En la sección **Static**, configuré la clave para vincular los datos de streaming con las entidades existentes: seleccioné **EquipmentId** como la columna que conecta las lecturas con los equipos. Esta columna coincide con la clave primaria de la entidad `vitalsignequipment`.

En la sección **Time series**, mapeé las propiedades de las lecturas con las columnas de la tabla:

- `ReadingId` → ReadingId
- `Timestamp` → Timestamp (ya configurado)
- `HeartRate` → HeartRate
- `OxygenSaturation` → OxygenSaturation
- `RespiratoryRate` → RespiratoryRate

Todas las columnas se auto‑mapearon correctamente, como se muestra en la imagen.

> ![Mapeo de columnas de series temporales](ontology_img2/20.%20map%20the%20souce%20collumn%20and%20it%20property%20name%20.png)

### 6. Guardado del enlace
Una vez verificadas todas las asignaciones, hice clic en **Save** para crear el enlace de series temporales. El sistema confirmó la operación y la entidad `vitalsignequipment` quedó con dos enlaces: uno estático (desde el lakehouse) y uno de series temporales (desde el eventhouse).

---

**Nota** Con este paso, la ontología `LamnaHealthcareOntology` quedó completamente configurada. La entidad `VitalSignEquipment` ahora combina datos de referencia estática (qué equipo monitorea a qué paciente) con datos de streaming en tiempo real (lecturas de signos vitales a lo largo del tiempo). La ontología cuenta con 5 tipos de entidad y 4 relaciones, con todos los enlaces de datos y relaciones completamente operativos.

## Previsualización de la ontología

### 1. Acceso a la vista de la entidad Rooms
En el explorador de la ontología, dentro de la lista de tipos de entidad, seleccioné **Rooms**. Luego, en la cinta de opciones de la ontología, hice clic en **Entity type overview** para acceder a la vista general de esta entidad. Mientras el sistema procesaba los datos en segundo plano, apareció un mensaje de "Updating your ontology". Esperé unos 1-2 minutos y luego refresqué el navegador para que la vista se mostrara correctamente.

La vista general mostraba tres elementos principales:
- **Relationship graph**: Representación visual de cómo esta entidad se conecta con otras entidades.
- **Property charts**: Gráficos de barras que muestran la distribución de los valores de las propiedades (como `RoomType`, `RoomNumber` o `DepartmentId`).
- **Entity instances table**: Lista de todas las instancias individuales de habitaciones con sus propiedades.

> ![Vista general de la entidad Rooms con gráficos y tabla de instancias](ontology_img2/21.%20Ontology%20preview%20of%20rooms.png)

### 2. Exploración de instancias individuales de Rooms
En la tabla de instancias, seleccioné la habitación **ICU-302** para abrir su vista detallada. La vista de instancia mostró las propiedades específicas de esta habitación (`RoomId`, `RoomNumber`, `DepartmentId`, `RoomType`) y sus conexiones con otras entidades a través del gráfico de relaciones.

> ![Lista detallada de instancias de Rooms](ontology_img2/22.%20check%20instance%20ICU%20-302.png)  
> ![Gráfico de relaciones para la instancia ICU-302](ontology_img2/23.%20grapgh%20about%20instance%20ICU%20-302.png)

### 3. Visualización de datos de signos vitales en tiempo real
Para verificar que el enlace de series temporales funcionaba correctamente, seleccioné la entidad `vitalsignequipment` y desde el panel de configuración, accedí a la vista de las lecturas de signos vitales. Allí pude observar:
- **HeartRate**: Gráfico de la frecuencia cardíaca a lo largo del tiempo.
- **OxygenSaturation**: Gráfico de la saturación de oxígeno.
- **RespiratoryRate**: Gráfico de la frecuencia respiratoria.

También verifiqué que los datos se mostraran con la granularidad configurada (1 minuto) y que la agregación (Average) se aplicara correctamente.

> ![Monitoreo de signos vitales en tiempo real](ontology_img2/24.%20patient%20vitalsigns%20monitoring.png)

---

**Nota final:** Con esta previsualización completé el laboratorio. La ontología `LamnaHealthcareOntology` cuenta con:
- **5 tipos de entidad**: Hospitals, Departments, Rooms, Patients, VitalSignEquipment
- **4 relaciones**: todas vinculadas a los datos de origen y completamente consultables
- **Datos estáticos + series temporales**: VitalSignEquipment combina datos de referencia del lakehouse con mediciones en tiempo real del eventhouse

La ontología ahora está lista para ser utilizada en aplicaciones de análisis, búsqueda semántica y visualización de datos en tiempo real.

