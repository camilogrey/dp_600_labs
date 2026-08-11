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



