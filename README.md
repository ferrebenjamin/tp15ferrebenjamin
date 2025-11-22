# tp15-consultas-sql-huertas-abp

## Descripción del TP

Este trabajo práctico consiste en la aplicación de los contenidos vistos en clase sobre bases de datos, incluyendo el modelo entidad-relación, claves primarias y foráneas, relaciones entre tablas y consultas básicas en SQL. El objetivo es realizar 5 ejercicios de consultas SQL utilizando la base de datos "huertas_abp". No se deben crear tablas, insertar datos ni modificar el esquema existente.

## Nombre completo

Ferre Benjamin

## Fecha de entrega

Viernes 21 de noviembre de 2025

## Explicación de cómo probé mis consultas

Las consultas SQL fueron probadas utilizando MySQL y la librería `mysql-connector-python` en un entorno de Google Colab. Se estableción una conexión a la base de datos `huertas_abp` alojada en Clever Cloud utilizando las credenciales proporcionadas. Se ejecutaron las consultas y se verificaron los resultados para asegurar que coincidan con los requerimientos de cada ejercicio. Se revisó la sintaxis y la lógica de cada consulta para garantizar su correcto funcionamiento.

## Descripción de la Base de Datos `huertas_abp`

La base de datos `huertas_abp` está diseñada para gestionar la información de una huerta comunitaria, incluyendo miembros, parcelas, inventario, cultivos, plantaciones, tareas y un foro de discusión. A continuación, se describen las tablas y sus relaciones:

*   **`miembros`:** Almacena información sobre los miembros de la huerta.
    *   Campos: `miembro_id` (clave primaria, autoincremental), `nombre`, `email` (único), `telefono`, `rol` (miembro, coordinador, administrador), `fecha_alta`.
*   **`parcelas`:** Contiene información sobre las parcelas de la huerta.
    *   Campos: `parcela_id` (clave primaria, autoincremental), `nombre`, `ubicacion`, `area_m2`, `responsable_id` (clave foránea que referencia a `miembros`).
    *   Índice: `idx_parcelas_responsable` en el campo `responsable_id`.
*   **`inventario`:** Almacena información sobre los ítems del inventario de la huerta (semillas, herramientas, insumos).
    *   Campos: `item_id` (clave primaria, autoincremental), `tipo` (semilla, herramienta, insumo), `nombre`, `cantidad`, `unidad`, `descripcion`.
    *   Índice: `idx_inventario_tipo_nombre` en los campos `tipo` y `nombre`.
*   **`cultivos`:** Contiene información sobre los diferentes cultivos que se siembran en la huerta.
    *   Campos: `cultivo_id` (clave primaria, autoincremental), `nombre`, `tiempo_cosecha_dias`, `descripcion`.
    *   Índice: `idx_cultivos_nombre` en el campo `nombre`.
*   **`plantaciones`:** Registra las plantaciones realizadas en la huerta.
    *   Campos: `plantacion_id` (clave primaria, autoincremental), `parcela_id` (clave foránea que referencia a `parcelas`), `cultivo_id` (clave foránea que referencia a `cultivos`), `sembrado_por` (clave foránea que referencia a `miembros`), `fecha_siembra`, `cantidad_semillas`, `estado` (activa, finalizada, fallida).
*   **`tareas`:** Almacena información sobre las tareas asignadas a los miembros para el mantenimiento de la huerta.
    *   Campos: `tarea_id` (clave primaria, autoincremental), `titulo`, `descripcion`, `asignada_a` (clave foránea que referencia a `miembros`), `parcela_id` (clave foránea que referencia a `parcelas`), `fecha_inicio`, `fecha_vencimiento`, `estado` (pendiente, en_progreso, completada), `prioridad` (baja, media, alta).
*   **`foro_posts`:** Contiene los posts del foro de discusión de la huerta.
    *   Campos: `post_id` (clave primaria, autoincremental), `autor_id` (clave foránea que referencia a `miembros`), `titulo`, `contenido`, `creado_en` (timestamp por defecto).
    *   Índice: `idx_post_autor` en el campo `autor_id`.
*   **`foro_comentarios`:** Almacena los comentarios realizados en los posts del foro.
    *   Campos: `comentario_id` (clave primaria, autoincremental), `post_id` (clave foránea que referencia a `foro_posts`), `autor_id` (clave foránea que referencia a `miembros`), `contenido`, `creado_en` (timestamp por defecto).

**Relaciones entre Tablas:**

*   `parcelas` tiene una relación de uno a muchos con `miembros` (un miembro puede ser responsable de varias parcelas).
*   `plantaciones` tiene relaciones de uno a muchos con `parcelas`, `cultivos` y `miembros`.
*   `tareas` tiene relaciones de uno a muchos con `miembros` y `parcelas`.
*   `foro_posts` tiene una relación de uno a muchos con `miembros` (un miembro puede crear varios posts).
*   `foro_comentarios` tiene relaciones de uno a muchos con `foro_posts` y `miembros`.

## Consultas SQL

Las 5 consultas SQL se encuentran en el archivo [consultas.sql](consultas.sql). Cada consulta está comentada indicando el número de ejercicio correspondiente.

A continuación, se describen las consultas y cómo interactúan con la base de datos:

1.  **Listar todas las parcelas con el nombre del responsable**
    *   SQL:
        ```sql
        SELECT 
            p.nombre AS parcela,
            p.ubicacion,
            p.area_m2,
            m.nombre AS responsable
        FROM parcelas p
        JOIN miembros m ON p.responsable_id = m.miembro_id
        ORDER BY p.nombre;
        ```
    *   Descripción: Esta consulta utiliza un `JOIN` para combinar las tablas `parcelas` y `miembros` a través del campo `responsable_id`.  Muestra el nombre, ubicación y área de cada parcela, junto con el nombre del miembro responsable. Los resultados se ordenan alfabéticamente por el nombre de la parcela.
2.  **Mostrar las plantaciones activas con información del cultivo**
    *   SQL:
        ```sql
        SELECT 
            pa.plantacion_id,
            p.nombre AS parcela,
            c.nombre AS cultivo,
            pa.fecha_siembra,
            pa.cantidad_semillas
        FROM plantaciones pa
        JOIN parcelas p ON pa.parcela_id = p.parcela_id
        JOIN cultivos c ON pa.cultivo_id = c.cultivo_id
        WHERE pa.estado = 'activa';
        ```
    *   Descripción: Esta consulta utiliza múltiples `JOIN` para combinar las tablas `plantaciones`, `parcelas` y `cultivos`.  Muestra el ID de la plantación, el nombre de la parcela, el nombre del cultivo, la fecha de siembra y la cantidad de semillas utilizadas para las plantaciones que tienen el estado "activa".
3.  **Obtener todas las tareas pendientes o en progreso asignadas a cada miembro**
    *   SQL:
        ```sql
        SELECT 
            t.titulo,
            m.nombre AS asignado_a,
            t.estado,
            t.fecha_vencimiento
        FROM tareas t
        JOIN miembros m ON t.asignada_a = m.miembro_id
        WHERE t.estado IN ('pendiente','en_progreso')
        ORDER BY t.fecha_vencimiento;
        ```
    *   Descripción: Esta consulta utiliza un `JOIN` para combinar las tablas `tareas` y `miembros` a través del campo `asignada_a`. Muestra el título de la tarea, el nombre del miembro asignado, el estado de la tarea y la fecha de vencimiento para las tareas que están en estado "pendiente" o "en_progreso". Los resultados se ordenan por la fecha de vencimiento.
4.  **Listar los ítems del inventario de tipo “semilla” con el cultivo al que pertenecen (si aplica)**
    *   SQL:
        ```sql
        SELECT 
            i.nombre AS item,
            i.cantidad,
            i.unidad,
            c.nombre AS cultivo_asociado
        FROM inventario i
        LEFT JOIN cultivos c ON i.nombre = c.nombre
        WHERE i.tipo = 'semilla';
        ```
    *   Descripción: Esta consulta utiliza un `LEFT JOIN` para combinar las tablas `inventario` y `cultivos`. Muestra el nombre del ítem, la cantidad, la unidad y el nombre del cultivo asociado para los ítems de tipo "semilla".  El `LEFT JOIN` asegura que se muestren todos los ítems de tipo "semilla", incluso si no tienen un cultivo asociado.
5.  **Mostrar los posts del foro con la cantidad de comentarios que recibió cada uno**
    *   SQL:
        ```sql
        SELECT 
            p.titulo,
            m.nombre AS autor,
            p.creado_en,
            COUNT(c.comentario_id) AS total_comentarios
        FROM foro_posts p
        JOIN miembros m ON p.autor_id = m.miembro_id
        LEFT JOIN foro_comentarios c ON p.post_id = c.post_id
        GROUP BY p.post_id, p.titulo, m.nombre, p.creado_en
        ORDER BY total_comentarios DESC;
        ```
    *   Descripción: Esta consulta utiliza `JOIN` y `LEFT JOIN` para combinar las tablas `foro_posts`, `miembros` y `foro_comentarios`.  Muestra el título del post, el nombre del autor, la fecha de creación y la cantidad total de comentarios para cada post.  Utiliza `GROUP BY` para agrupar los resultados por post y `COUNT(c.comentario_id)` para contar los comentarios. Los resultados se ordenan por la cantidad de comentarios en orden descendente.

## Enunciados de los ejercicios

1.  **Listar todas las parcelas con el nombre del responsable**
    *   Mostrar:
        *   nombre de la parcela
        *   ubicación
        *   área
        *   nombre del miembro responsable
    *   Ordenar alfabéticamente por nombre de parcela.
2.  **Mostrar las plantaciones activas con información del cultivo**
    *   Mostrar:
        *   parcela
        *   cultivo
        *   fecha de siembra
        *   cantidad de semillas utilizadas
    *   Solo incluir plantaciones con estado "activa".
3.  **Obtener todas las tareas pendientes o en progreso asignadas a cada miembro**
    *   Mostrar:
        *   nombre del miembro
        *   título de la tarea
        *   estado
        *   fecha de vencimiento
    *   Ordenar por fecha de vencimiento (más próximas primero).
4.  **Listar los ítems del inventario de tipo “semilla” con el cultivo al que pertenecen (si aplica)**
    *   Incluir:
        *   nombre del ítem
        *   cantidad disponible
        *   unidad
        *   nombre del cultivo asociado (si coincide por nombre)
    *   Mostrar también los ítems que no tengan cultivo asociado.
5.  **Mostrar los posts del foro con la cantidad de comentarios que recibió cada uno**
    *   Incluir:
        *   título del post
        *   autor (nombre del miembro)
        *   fecha de creación
        *   cantidad total de comentarios
    *   Ordenar por cantidad de comentarios (mayor a menor).

## Criterios de evaluación

*   Correcto uso de SELECT, JOIN, WHERE, GROUP BY, ORDER BY.
*   Claridad y legibilidad del archivo consultas.sql.
*   Buena presentación del README.md.
*   Correcto uso de GitHub para la entrega.
*
