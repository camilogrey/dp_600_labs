# Laboratorio: real time intelligence

## Requisitos previos
- Disponer de acceso a una capacidad de Fabric de pago o de prueba (trial).  
- Contar con un inquilino (tenant) de Microsoft Fabric.

---

## Desarrollo del laboratorio

1. **Acceso a la plataforma**  

2. **Creación de un nuevo workspace**  
   Dentro de la sección de workspaces, elegí la opción de crear un nuevo workspace. Asigné el nombre **`real_time_intelligence`** (un nombre a elección, tal como se indica en el laboratorio).  
   Complete los campos adicionales:  
   - **Descripción**: escribí una breve descripción para identificar el propósito del workspace.  
   - **Dominio**: dejé este campo opcional sin asignar.  
   - **Imagen del workspace**: mantuve la imagen por defecto (no cargué ninguna).  

   Seleccioné un modo de licencia que incluyera capacidad de Fabric (en mi caso, elegí la opción **Trial** para la prueba).  
   Finalmente, hice clic en **Apply** para confirmar la creación.

   > **Evidencia del paso de creación**  

![Pantalla de creación del workspace](img_lab1/1.png)

3. **Verificación del workspace vacío**  
   Una vez creado, el workspace se abrió automáticamente. Tal como se esperaba, se mostró completamente vacío, sin elementos ni tareas predefinidas.  
   En la interfaz pude observar las opciones para agregar nuevos elementos, carpetas, importar o migrar, así como la posibilidad de elegir flujos de tareas prediseñados o añadir tareas personalizadas.

   > **Evidencia del workspace recién creado y vacío**  
   > ![Workspace vacío](img_lab1/2.png)

# Creación de un eventstream

1. **Acceso al Real‑Time Hub**  
   Desde el workspace `real_time_intelligence` que creé anteriormente, localicé en la barra lateral izquierda el icono de **Real‑Time hub** (si no aparecía, lo fijé usando el botón de puntos suspensivos).  
   > ![Vista del workspace y acceso al hub](img2_lab1/1.png)

2. **Inicio de la adición de datos**  
   Dentro del Real‑Time Hub, hice clic en el botón **Add data** para comenzar a incorporar una fuente de streaming.  
   > ![Real‑Time Hub con opcion Add data](img2_lab1/2.png)

3. **Selección de la fuente de muestra**  
   Entre las opciones disponibles, elegí el origen de datos de muestra **Stock market**, que proporciona datos bursátiles en tiempo real.

4. **Configuración de la conexión**  
   En el panel de configuración, completé los campos tal como se indicaba:  
   - **Source name**: `stock`  
   - **Workspace**: seleccioné `real_time_intelligence` (el que había creado)  
   - **Eventstream name**: `stock-data`  
   - El nombre del stream asociado se generó automáticamente como `stock-data-stream`.  
   > ![Configuración inicial](img2_lab1/4.png)  
   > ![Confirmación de parámetros](img2_lab1/5.png)

5. **Conexión y creación del eventstream**  
   Revisé los datos y pulsé **Next**, luego **Connect** para crear el eventstream.  
   Una vez finalizado, seleccioné **Open eventstream** para visualizar el flujo en el lienzo de diseño.

6. **Resultado final**  
   El eventstream se mostró correctamente en el canvas, con el origen `stock` y el stream `stock-data-stream` listos para ser utilizados en transformaciones o enrutamientos.  
   > ![Eventstream abierto en el canvas](img2_lab1/6.png)

---
**Nota:** Con este eventstream ya tengo una fuente de datos en tiempo real sobre la que podré aplicar procesamientos posteriores.

