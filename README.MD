 ## SAKILA

Kevin Bueno

```sql
Create Database Sakila;

use Sakila;

Create Table sakilacampus_film_text (
    film_id smallint primary key,
    title varchar(255),
    description text
);

Create Table sakilacampus_categoria(
    id_categoria tinyint primary key,
    nombre varchar(25),
    ultima_actualizacion timestamp
);

Create Table sakilacampus_idioma(
    id_idioma tinyint Auto_increment primary key,
    nombre char(20),
    ultima_actualizacion timestamp
);

Create table sakilacampus_pais(
    id_pais smallint Auto_Increment primary key,
    nombre varchar(50),
    ultima_actualizacion timestamp
);

Create table sakilacampus_ciudad(
    id_ciudad smallint Auto_increment primary key,
    nombre varchar(50),
    id_pais smallint,
    ultima_actualizacion timestamp,
    Foreign key (id_pais) References sakilacampus_pais(id_pais)
);

Create table sakilacampus_direccion(
    id_direccion smallint Auto_increment primary key,
    direccion varchar(50),
    direccion2 varchar(50),
    distrito varchar(50),
    id_ciudad smallint ,
    codigo_postal varchar(10),
    telefono varchar(20),
    ultima_actualizacion timestamp,
    Foreign key (id_ciudad) References sakilacampus_ciudad(id_ciudad)
);

Create table sakilacampus_actor(
    id_actor smallint Auto_increment primary key,
    nombre varchar(45),
    apellidos varchar(45),
    ultima_actualizacion timestamp
);


create table sakilacampus_almacen(
    id_almacen tinyint auto_increment primary key,
    id_empleado_jefe tinyint,
    id_direccion smallint,
    ultima_actualizacion timestamp,
    Foreign key (id_direccion) References sakilacampus_direccion(id_direccion)
);

create table sakilacampus_empleado (
    id_empleado tinyint auto_increment primary key,
    nombre varchar(45),
    apellidos varchar(45),
    id_direccion smallint ,
    imagen blob,
    email varchar(45),
    id_almacen tinyint ,
    activo tinyint(1),
    username varchar(16),
    password varchar(40),
    ultima_actualizacion timestamp,
    Foreign key (id_direccion) References sakilacampus_direccion(id_direccion),
   Foreign key (id_almacen) References sakilacampus_almacen(id_almacen)
    
);

create table sakilacampus_cliente (
    id_cliente smallint auto_increment primary key,
    id_almacen tinyint ,
    nombre varchar(45),
    apellidos varchar(45),
    email varchar(50),
    id_direccion smallint,
    activo tinyint(1),
    fecha_creacion datetime,
    ultima_actualizacion timestamp,
    Foreign key (id_direccion) References sakilacampus_direccion(id_direccion),
   Foreign key (id_almacen) References sakilacampus_almacen(id_almacen)
    
);

create table sakilacampus_pelicula(
    id_pelicula smallint auto_increment primary key,
    titulo varchar(255),
    descripcion text,
    anyo_lanzamiento year,
    id_idioma tinyint ,
    id_idioma_original tinyint ,
    duracion_alquiler tinyint ,
    rental_rate decimal(4,2),
    duracion smallint ,
    replacement_cost decimal(5,2),
    clasificacion enum('G','PG','PG-13','R','NC-17'),
    caracteristicas_especiales set('Trailers','Commentaries','Deleted Scenes','Behind the Scenes'),
    ultima_actualizacion timestamp,
    Foreign key (id_idioma) References sakilacampus_idioma(id_idioma),
    Foreign key (id_idioma_original) References sakilacampus_idioma(id_idioma)
);

create table sakilacampus_inventario(
    id_inventario  mediumint auto_increment primary key,
    id_pelicula smallint ,
    id_almacen tinyint ,
    ultima_actualizacion timestamp,
    Foreign key (id_pelicula) References sakilacampus_pelicula(id_pelicula),
    Foreign key (id_almacen) References sakilacampus_almacen(id_almacen)
);

create table sakilacampus_alquiler(
    id_alquiler int primary key,
    fecha_alquiler datetime,
    id_inventario mediumint ,
    id_cliente smallint,
    fecha_devolucion datetime,
    id_empleado tinyint ,
    ultima_actualizacion timestamp,
    Foreign key (id_empleado) References sakilacampus_empleado(id_empleado),
    Foreign key (id_inventario) References sakilacampus_inventario(id_inventario)
);




create table sakilacampus_pagos (
    id_pago smallint auto_increment primary key,
    id_cliente smallint ,
    id_empleado tinyint ,
    id_alquiler int,
    total decimal(5,2),
    fecha_pago datetime,
    ultima_actualizacion timestamp,
    Foreign key (id_alquiler) References sakilacampus_alquiler(id_alquiler),
    Foreign key (id_empleado) References sakilacampus_empleado(id_empleado),
    Foreign key (id_cliente) References sakilacampus_cliente(id_cliente)
    
);

create table sakilacampus_pelicula_categoria(
    id_pelicula smallint,
    id_categoria tinyint,
    ultima_actualizacion timestamp,
    Primary key (id_pelicula,id_categoria),
    Foreign key (id_Pelicula) References sakilacampus_pelicula(id_pelicula),
    Foreign key (id_categoria) References sakilacampus_categoria(id_categoria)

);

create table sakilacampus_pelicula_actor(
    id_pelicula smallint,
    id_actor smallint,
    ultima_actualizacion timestamp,
    Primary key (id_pelicula,id_actor),
    Foreign key (id_Pelicula) References sakilacampus_pelicula(id_pelicula),
    Foreign key (id_actor) References sakilacampus_actor(id_actor)

);
```



# # Consulta





-- 1. Encuentra el cliente que ha realizado la mayor cantidad de alquileres en los últimos 6 meses.
```sql
SELECT id_cliente, COUNT(*) AS total_alquileres
FROM sakilacampus_alquiler
WHERE fecha_alquiler > CURDATE() - INTERVAL 6 MONTH
GROUP BY id_cliente
ORDER BY total_alquileres DESC
LIMIT 1;

-- 2. Lista las cinco películas más alquiladas durante el último año.
SELECT id_pelicula, COUNT(*) AS total_alquileres
FROM sakilacampus_alquiler
WHERE fecha_alquiler > CURDATE() - INTERVAL 1 YEAR
GROUP BY id_pelicula
ORDER BY total_alquileres DESC
LIMIT 5;

-- 3. Obtén el total de ingresos y la cantidad de alquileres realizados por cada categoría de película.
SELECT c.nombre AS categoria, 
       SUM(p.total) AS total_ingresos,
       COUNT(a.id_alquiler) AS cantidad_alquileres
FROM sakilacampus_pelicula_categoria pc
JOIN sakilacampus_categoria c ON c.id_categoria = pc.id_categoria
JOIN sakilacampus_alquiler a ON a.id_inventario = pc.id_pelicula
JOIN sakilacampus_pagos p ON p.id_alquiler = a.id_alquiler
GROUP BY c.id_categoria;

-- 4. Calcula el número total de clientes que han realizado alquileres por cada idioma disponible en un mes específico.
SELECT i.nombre AS idioma, 
       COUNT(DISTINCT a.id_cliente) AS total_clientes
FROM sakilacampus_alquiler a
JOIN sakilacampus_inventario i ON i.id_pelicula = a.id_inventario
WHERE a.fecha_alquiler BETWEEN '2025-03-01' AND '2025-03-31'
GROUP BY i.id_idioma;

-- 5. Encuentra a los clientes que han alquilado todas las películas de una misma categoría.
SELECT a.id_cliente
FROM sakilacampus_alquiler a
JOIN sakilacampus_pelicula_categoria pc ON a.id_inventario = pc.id_pelicula
WHERE pc.id_categoria = 1 -- Categoría ejemplo: 1 es Acción
GROUP BY a.id_cliente
HAVING COUNT(DISTINCT pc.id_pelicula) = (SELECT COUNT(*) FROM sakilacampus_pelicula_categoria WHERE id_categoria = 1);

-- 6. Lista las tres ciudades con más clientes activos en el último trimestre.
SELECT c.nombre AS ciudad, COUNT(DISTINCT cl.id_cliente) AS total_clientes
FROM sakilacampus_cliente cl
JOIN sakilacampus_direccion d ON cl.id_direccion = d.id_direccion
JOIN sakilacampus_ciudad c ON d.id_ciudad = c.id_ciudad
WHERE cl.activo = 1 AND cl.fecha_creacion > CURDATE() - INTERVAL 3 MONTH
GROUP BY c.id_ciudad
ORDER BY total_clientes DESC
LIMIT 3;

-- 7. Muestra las cinco categorías con menos alquileres registrados en el último año.
SELECT cat.nombre AS categoria, COUNT(*) AS total_alquileres
FROM sakilacampus_pelicula_categoria pc
JOIN sakilacampus_categoria cat ON cat.id_categoria = pc.id_categoria
JOIN sakilacampus_alquiler a ON a.id_inventario = pc.id_pelicula
WHERE a.fecha_alquiler > CURDATE() - INTERVAL 1 YEAR
GROUP BY cat.id_categoria
ORDER BY total_alquileres ASC
LIMIT 5;

-- 8. Calcula el promedio de días que un cliente tarda en devolver las películas alquiladas.
SELECT a.id_cliente, AVG(DATEDIFF(a.fecha_devolucion, a.fecha_alquiler)) AS promedio_dias
FROM sakilacampus_alquiler a
WHERE a.fecha_devolucion IS NOT NULL
GROUP BY a.id_cliente;

-- 9. Encuentra los cinco empleados que gestionaron más alquileres en la categoría de Acción.
SELECT e.id_empleado, COUNT(*) AS total_alquileres
FROM sakilacampus_alquiler a
JOIN sakilacampus_empleado e ON e.id_empleado = a.id_empleado
JOIN sakilacampus_inventario i ON i.id_inventario = a.id_inventario
JOIN sakilacampus_pelicula_categoria pc ON pc.id_pelicula = i.id_pelicula
WHERE pc.id_categoria = 1 -- Acción
GROUP BY e.id_empleado
ORDER BY total_alquileres DESC
LIMIT 5;

-- 10. Genera un informe de los clientes con alquileres más recurrentes.
SELECT id_cliente, COUNT(*) AS total_alquileres
FROM sakilacampus_alquiler
GROUP BY id_cliente
ORDER BY total_alquileres DESC;

-- 11. Calcula el costo promedio de alquiler por idioma de las películas.
SELECT i.nombre AS idioma, AVG(p.total) AS costo_promedio
FROM sakilacampus_pelicula p
JOIN sakilacampus_inventario i ON i.id_pelicula = p.id_pelicula
JOIN sakilacampus_pagos p ON p.id_alquiler = a.id_alquiler
GROUP BY i.id_idioma;

-- 12. Lista las cinco películas con mayor duración alquiladas en el último año.
SELECT p.titulo, p.duracion, COUNT(*) AS total_alquileres
FROM sakilacampus_alquiler a
JOIN sakilacampus_pelicula p ON a.id_inventario = p.id_pelicula
WHERE a.fecha_alquiler > CURDATE() - INTERVAL 1 YEAR
GROUP BY p.id_pelicula
ORDER BY p.duracion DESC
LIMIT 5;

-- 13. Muestra los clientes que más alquilaron películas de Comedia.
SELECT cl.id_cliente, COUNT(*) AS total_comedias
FROM sakilacampus_alquiler a
JOIN sakilacampus_pelicula_categoria pc ON a.id_inventario = pc.id_pelicula
JOIN sakilacampus_categoria c ON c.id_categoria = pc.id_categoria
WHERE c.nombre = 'Comedia'
GROUP BY cl.id_cliente
ORDER BY total_comedias DESC;

-- 14. Encuentra la cantidad total de días alquilados por cada cliente en el último mes.
SELECT id_cliente, SUM(DATEDIFF(fecha_devolucion, fecha_alquiler)) AS total_dias_alquilados
FROM sakilacampus_alquiler
WHERE fecha_alquiler BETWEEN CURDATE() - INTERVAL 1 MONTH AND CURDATE()
GROUP BY id_cliente;

-- 15. Muestra el número de alquileres diarios en cada almacén durante el último trimestre.
SELECT a.id_almacen, DATE(a.fecha_alquiler) AS fecha, COUNT(*) AS total_alquileres
FROM sakilacampus_alquiler a
WHERE a.fecha_alquiler > CURDATE() - INTERVAL 3 MONTH
GROUP BY a.id_almacen, fecha
ORDER BY fecha DESC;

-- 16. Calcula los ingresos totales generados por cada almacén en el último semestre.
SELECT a.id_almacen, SUM(p.total) AS total_ingresos
FROM sakilacampus_pagos p
JOIN sakilacampus_alquiler al ON al.id_alquiler = p.id_alquiler
WHERE al.fecha_alquiler > CURDATE() - INTERVAL 6 MONTH
GROUP BY a.id_almacen;

-- 17. Encuentra el cliente que ha realizado el alquiler más caro en el último año.
SELECT id_cliente, MAX(total) AS alquiler_mas_caro
FROM sakilacampus_pagos
WHERE fecha_pago > CURDATE() - INTERVAL 1 YEAR
GROUP BY id_cliente
ORDER BY alquiler_mas_caro DESC
LIMIT 1;

-- 18. Lista las cinco categorías con más ingresos generados durante los últimos tres meses.
SELECT c.nombre AS categoria, SUM(p.total) AS total_ingresos
FROM sakilacampus_pelicula_categoria pc
JOIN sakilacampus_categoria c ON c.id_categoria = pc.id_categoria
JOIN sakilacampus_alquiler a ON a.id_inventario = pc.id_pelicula
JOIN sakilacampus_pagos p ON p.id_alquiler = a.id_alquiler
WHERE a.fecha_alquiler > CURDATE() - INTERVAL 3 MONTH
GROUP BY c.id_categoria
ORDER BY total_ingresos DESC
LIMIT 5;

-- 19. Obtén la cantidad de películas alquiladas por cada idioma en el último mes.
SELECT i.nombre AS idioma, COUNT(*) AS total_peliculas
FROM sakilacampus_alquiler a
JOIN sakilacampus_inventario i ON i.id_pelicula = a.id_inventario
WHERE a.fecha_alquiler BETWEEN CURDATE() - INTERVAL 1 MONTH AND CURDATE()
GROUP BY i.id_idioma;

--20. Lista los clientes que no han realizado ningún alquiler en el último año.

SELECT c.id_cliente, c.nombre
FROM sakilacampus_cliente c
LEFT JOIN sakilacampus_alquiler a ON a.id_cliente = c.id_cliente AND a.fecha_alquiler > CURDATE() - INTERVAL 1 YEAR
WHERE a.id_alquiler IS NULL;

```

# # funciones 
``` sql

-- 1.
DELIMITER $$

CREATE FUNCTION TotalIngresosCliente(ClienteID INT, Año INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE Total DECIMAL(10,2);
    
    SELECT SUM(p.total) INTO Total
    FROM sakilacampus_pagos p
    JOIN sakilacampus_alquiler a ON p.id_alquiler = a.id_alquiler
    WHERE a.id_cliente = ClienteID
      AND YEAR(a.fecha_alquiler) = Año;
    
    RETURN IFNULL(Total, 0);
END $$

DELIMITER ;

--2 
DELIMITER $$

CREATE FUNCTION PromedioDuracionAlquiler(PeliculaID INT)
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
    DECLARE Promedio DECIMAL(5,2);
    
    SELECT AVG(TIMESTAMPDIFF(DAY, a.fecha_alquiler, a.fecha_devolucion)) INTO Promedio
    FROM sakilacampus_alquiler a
    JOIN sakilacampus_inventario i ON a.id_inventario = i.id_inventario
    WHERE i.id_pelicula = PeliculaID
      AND a.fecha_devolucion IS NOT NULL;
    
    RETURN IFNULL(Promedio, 0);
END $$

DELIMITER ;

--3 
DELIMITER $$

CREATE FUNCTION IngresosPorCategoria(CategoriaID INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE TotalIngresos DECIMAL(10,2);
    
    SELECT SUM(p.total) INTO TotalIngresos
    FROM sakilacampus_pelicula_categoria pc
    JOIN sakilacampus_alquiler a ON pc.id_pelicula = a.id_inventario
    JOIN sakilacampus_pagos p ON a.id_alquiler = p.id_alquiler
    WHERE pc.id_categoria = CategoriaID;
    
    RETURN IFNULL(TotalIngresos, 0);
END $$

DELIMITER ;

--4 
DELIMITER $$

CREATE FUNCTION DescuentoFrecuenciaCliente(ClienteID INT)
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
    DECLARE TotalAlquileres INT;
    DECLARE Descuento DECIMAL(5,2);
    
    SELECT COUNT(*) INTO TotalAlquileres
    FROM sakilacampus_alquiler
    WHERE id_cliente = ClienteID;
    
    IF TotalAlquileres >= 50 THEN
        SET Descuento = 0.15;  -- 15% de descuento si ha alquilado más de 50 veces
    ELSEIF TotalAlquileres >= 30 THEN
        SET Descuento = 0.10;  -- 10% de descuento si ha alquilado más de 30 veces
    ELSE
        SET Descuento = 0.05;  -- 5% de descuento si ha alquilado más de 10 veces
    END IF;
    
    RETURN Descuento;
END $$

DELIMITER ;


```



# #  RECLAMOS

Los triggers son imposibles de hacer debido a que el examen no permite la adicion dse tablas y estas son ncesarias para el trigger 




