# Laboratorio: Aplicar la seguridad del modelo semántico

## ROLES ESTÁTICOS: Implementación de seguridad a nivel de fila (RLS) en Power BI

### 1. Preparación del entorno
Descargué el archivo ZIP desde la URL proporcionada y lo extraje en mi equipo local. Luego abrí el archivo `17-Starter-Sales Analysis.pbix` en Power BI Desktop. Ignoré cualquier advertencia de seguridad que apareció, asegurándome de no descartar cambios ni actualizar los datos, tal como se indicaba en las instrucciones.
> [Apertura de archivo](enf_sem_mod_sec_img/17-Starter-Sales%20Analysis.pbix)


### 2. Revisión del modelo de datos
En Power BI Desktop, cambié a la **Vista de modelo** (Model view) para examinar la estructura del modelo semántico. El modelo consta de seis tablas de dimensiones y una tabla de hechos (`Sales`), formando un esquema estrella. Localicé la tabla **Sales Territory** y verifiqué que contenía la columna **Region**, que almacena las regiones de ventas de Adventure Works. Esta columna sería clave para aplicar los filtros de seguridad, ya que los vendedores solo pueden ver datos de su región asignada.

> ![Vista del modelo de datos mostrando las relaciones entre tablas](enf_sem_mod_sec_img/1.%20Model%20view.png)

### 3. Creación de roles estáticos
Para implementar la seguridad a nivel de fila (RLS), comencé con roles estáticos. Desde la vista **Informe** (Report view), en la cinta **Modelado** (Modeling), seleccioné **Administrar roles** (Manage roles) dentro del grupo **Seguridad** (Security). Allí creé dos roles:

- **Australia**: configuré un filtro en la tabla `Sales Territory` con la condición `Region equals Australia`.
- **Canada**: repetí el proceso, creando un segundo rol con el filtro `Region equals Canada`.

Guardé los cambios y los roles quedaron listos.

> ![Configuración de roles estáticos](enf_sem_mod_sec_img/2.%20Manage%20roles.png)  
> ![Creación del rol Australia con su regla de filtro](enf_sem_mod_sec_img/3.%20New%20role%20&%20new%20rule.png)  
> ![Ambos roles creados (Australia y Canada) con sus respectivas reglas](enf_sem_mod_sec_img/4.%20two%20roles%20with%20one%20rule%20each.png)

### 4. Validación de los roles estáticos
Para probar que los roles funcionaban correctamente, utilicé la opción **Ver como** (View as) en la cinta **Modelado**. Seleccioné el rol **Australia** y confirmé que el gráfico de columnas apiladas mostraba únicamente los datos correspondientes a esa región. Luego, detuve la vista y repetí la prueba con el rol **Canada**, verificando que también filtrara correctamente. Ambos roles funcionaron como se esperaba.

> ![Vista previa con el rol Australia activo](enf_sem_mod_sec_img/5.%20view%20as%20the%20role%20Australia.png)  
> ![Resultado del gráfico filtrado para la región Australia](enf_sem_mod_sec_img/6.%20result%20of%20the%20role%20view%20Australia.png)

### 5. Limpieza de roles estáticos
Dado que los roles estáticos no escalan bien (cada nueva región requeriría un nuevo rol), eliminé los roles **Australia** y **Canada** antes de continuar con el siguiente paso.

> ![Eliminación de los roles estáticos](enf_sem_mod_sec_img/7.%20delete%20static%20roles.png)

---

**Nota final:** Con estos pasos completé la configuración y validación de los roles estáticos de RLS en Power BI. Comprobé que el filtrado por región funciona correctamente, pero también entendí sus limitaciones para escenarios con muchas regiones, lo que motiva la implementación de RLS dinámica en el siguiente paso del laboratorio.

## ROLES DINÁMICOS: Implementación de seguridad a nivel de fila (RLS) en Power BI

Se documenta el proceso para implementar la Seguridad a Nivel de Fila Dinámica (Dynamic RLS). El objetivo es restringir el acceso a los datos de ventas para que cada vendedor solo pueda visualizar la información de su territorio asignado.

El enfoque utiliza una tabla de mapeo (`salesperson`) que vincula el correo electrónico del usuario (UPN) con su clave de territorio, configurando filtros de seguridad bidireccionales.

## 1. Importación de la tabla Salesperson (CSV)

Para comenzar, se importan los datos de los vendedores desde un archivo CSV utilizando Power Query.

*   **Paso 1:** En la cinta de opciones **Home**, haga clic en **Get data** (Obtener datos) y seleccione **Text/CSV**.
*   **Paso 2:** Navegue a la carpeta del laboratorio, seleccione el archivo `Salesperson.csv` y haga clic en **Open**.
![Archivo de descarga]()

![Imagen 1: Obtención de datos desde el menú Home](image1.png)

*   **Paso 3:** En la ventana de previsualización, verifique que las tres columnas sean: `EmployeeKey`, `SalesTerritoryKey` y `EmailAddress`.
*   **Paso 4:** Seleccione el botón **Transform Data** (Transformar datos) para abrir el Editor de Power Query.

![Imagen 2: Previsualización del archivo CSV y botón Transform Data](image2.png)

## 2. Preparación de datos en Power Query

En esta tarea, se renombra la columna de correo electrónico para que coincida con el identificador de usuario que devuelve la función DAX `USERPRINCIPALNAME()`.

*   **Paso 1:** En el Editor de Power Query, haga clic derecho en el encabezado de la columna `EmailAddress` y seleccione **Rename** (Cambiar nombre).
*   **Paso 2:** Cambie el nombre a `UPN` y presione Enter.
*   **Paso 3:** En la cinta de opciones **Home**, seleccione **Close & Apply** (Cerrar y aplicar) para cargar la tabla en el modelo.

![Imagen 3: Renombrado de columna y aplicación de cambios](image3.png)

## 3. Configuración de la relación y el filtro de seguridad

Para que los filtros de seguridad fluyan desde la tabla de vendedores hacia el territorio y finalmente hacia las ventas, es necesario configurar una relación bidireccional especial.

*   **Paso 1:** Cambie a la **Vista Modelo** (Model view).
*   **Paso 2:** Arrastre el campo `SalesTerritoryKey` de la tabla `Salesperson` al campo `SalesTerritoryKey` de la tabla `Sales Territory` para crear una relación. (Como se muestra en la línea de conexión entre ambas tablas).

![Imagen 4: Vista de modelo y creación de relación](image4.png)

*   **Paso 3:** Haga clic derecho sobre la línea de relación entre `Salesperson` y `Sales Territory` y seleccione **Properties** (Propiedades).
*   **Paso 4:** En la ventana **Edit relationship** (Editar relación):
    *   Configure **Cross-filter direction** en **Both** (Ambos).
    *   Marque la casilla **Apply security filter in both directions** (Aplicar filtro de seguridad en ambas direcciones).
*   **Paso 5:** Haga clic en **Save** para guardar la configuración.

![Imagen 5: Configuración del filtro cruzado bidireccional de seguridad](image5.png)

*   **Paso 6:** Verifique en el diagrama que el indicador de relación ahora muestra flechas en ambas direcciones.

![Imagen 6: Verificación de la relación bidireccional en el modelo](image6.png)

## 4. Ocultar la tabla Salesperson

Dado que la tabla `Salesperson` solo se utiliza para la seguridad y no debe visualizarse en los informes ni exponer datos de UPN, la ocultaremos de la vista de informes.

*   **Paso 1:** En la **Vista Modelo**, seleccione el encabezado de la tabla `Salesperson`.
*   **Paso 2:** Haga clic derecho y seleccione **Hide in report view** (Ocultar en la vista de informes).

![Imagen 7: Ocultar la tabla Salesperson de la vista de informes](image7.png)

## 5. Creación del rol de seguridad dinámico con DAX

Finalmente, se crea un único rol dinámico que filtrará los datos en función del usuario autenticado, evitando tener que crear un rol por cada vendedor.

*   **Paso 1:** En la cinta de opciones **Modeling** (Modelado), en el grupo **Security** (Seguridad), seleccione **Manage roles** (Administrar roles).
*   **Paso 2:** En la ventana emergente:
    *   Seleccione **+ New** para crear un nuevo rol.
    *   Asígnele el nombre `Salespeople`.
    *   Seleccione la tabla `Salesperson` en la lista de tablas.
    *   Seleccione el botón **Switch to DAX editor** (Cambiar al editor DAX).

![Imagen 8: Administración de roles y selección del editor DAX](image8.png)

*   **Paso 3:** En el editor, introduzca la siguiente expresión DAX y presione la marca de verificación:

    ```dax
    [UPN] = USERPRINCIPALNAME()

