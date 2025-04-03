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



1. Encuentra el cliente que ha realizado la mayor cantidad de alquileres en los últimos 6 meses.

```sql

Select c.id_cliente, c.nombre, count(a.id_alquiler) as cantidad alquiler
From sakilacampus_clite c
inner join sakilacampus_alquiler a on c.id_cliente = a.id_cliente
group by c.id_cliente
order by cantidad_alquileres Desc 
```



2. Lista las cinco películas más alquiladas durante el último año.

   ```sql
   Select p.id_pelicula, p.nombre, count (sa.id_alquiler ) as alquileres
   from sakilacampus_pelicula p
   inner join sakilacampus_inventario si on p.id_pelicula = si.id_pelicula
   inner join sakilacampus_alquiler sa on si.id_inventario = sa.id_inventario
   Group by P.id_pelicula
   order by alquileres Desc
   
   ```

   

# # Funciones



1. TotalIngresosCliente(ClienteID, Año): Calcula los ingresos generados por un cliente en un
año específico.

```sql
SELECT id_cliente, COUNT(*) AS cantidad_alquileres
FROM sakilacampus_alquiler
WHERE fecha_alquiler >= CURDATE() - INTERVAL 6 MONTH
GROUP BY id_cliente
ORDER BY cantidad_alquileres DESC
LIMIT 1;

```

2.

```sql
SELECT p.titulo, COUNT(*) AS cantidad_alquileres
FROM sakilacampus_alquiler a
JOIN sakilacampus_inventario i ON a.id_inventario = i.id_inventario
JOIN sakilacampus_pelicula p ON i.id_pelicula = p.id_pelicula
WHERE a.fecha_alquiler >= CURDATE() - INTERVAL 1 YEAR
GROUP BY p.id_pelicula
ORDER BY cantidad_alquileres DESC
LIMIT 5;

```



3. ```sql
   SELECT c.nombre AS categoria, 
          SUM(p.total) AS total_ingresos,
          COUNT(a.id_alquiler) AS cantidad_alquileres
   FROM sakilacampus_pagos p
   JOIN sakilacampus_alquiler a ON p.id_alquiler = a.id_alquiler
   JOIN sakilacampus_inventario i ON a.id_inventario = i.id_inventario
   JOIN sakilacampus_pelicula_categoria pc ON i.id_pelicula = pc.id_pelicula
   JOIN sakilacampus_categoria c ON pc.id_categoria = c.id_categoria
   GROUP BY c.id_categoria;
   
   ```

   4. ```sql
      SELECT i.nombre AS idioma, COUNT(DISTINCT a.id_cliente) AS cantidad_clientes
      FROM sakilacampus_alquiler a
      JOIN sakilacampus_inventario i ON a.id_inventario = i.id_inventario
      JOIN sakilacampus_pelicula p ON i.id_pelicula = p.id_pelicula
      JOIN sakilacampus_idioma i ON p.id_idioma = i.id_idioma
      WHERE MONTH(a.fecha_alquiler) = 4 AND YEAR(a.fecha_alquiler) = 2025  
      GROUP BY i.id_idioma;
      
      ```

      5. ```sql
         SELECT c.id_cliente, c.nombre, c.apellidos
         FROM sakilacampus_cliente c
         WHERE NOT EXISTS (
             SELECT 1
             FROM sakilacampus_pelicula_categoria pc
             LEFT JOIN sakilacampus_inventario i ON pc.id_pelicula = i.id_pelicula
             LEFT JOIN sakilacampus_alquiler a ON i.id_inventario = a.id_inventario
             WHERE pc.id_categoria = 1  -- ejemplo de categoría de acción
             AND a.id_cliente != c.id_cliente
         )
         GROUP BY c.id_cliente;
         
         ```

         6.

         ```sql
         SELECT ci.nombre AS ciudad, COUNT(c.id_cliente) AS cantidad_clientes
         FROM sakilacampus_cliente c
         JOIN sakilacampus_direccion d ON c.id_direccion = d.id_direccion
         JOIN sakilacampus_ciudad ci ON d.id_ciudad = ci.id_ciudad
         WHERE c.activo = 1 AND c.fecha_creacion >= CURDATE() - INTERVAL 3 MONTH
         GROUP BY ci.id_ciudad
         ORDER BY cantidad_clientes DESC
         LIMIT 3;
         
         ```

         7. ```sql
            SELECT c.nombre AS categoria, COUNT(a.id_alquiler) AS cantidad_alquileres
            FROM sakilacampus_categoria c
            LEFT JOIN sakilacampus_pelicula_categoria pc ON c.id_categoria = pc.id_categoria
            LEFT JOIN sakilacampus_inventario i ON pc.id_pelicula = i.id_pelicula
            LEFT JOIN sakilacampus_alquiler a ON i.id_inventario = a.id_inventario
            WHERE a.fecha_alquiler >= CURDATE() - INTERVAL 1 YEAR
            GROUP BY c.id_categoria
            ORDER BY cantidad_alquileres ASC
            LIMIT 5;
            
            ```


            

# # trigger

2. AuditarActualizacionCliente: Cada vez que se modifica un cliente, registra el cambio en una
   tabla de auditoría.

```sql
CREATE TRIGGER trg_after_insert
AFTER Update ON cliente
FOR EACH ROW
BEGIN
    INSERT INTO auditoria (id_cliente, modificacion, fecha)
    VALUES ('INSERT', 'modifiacion', NOW());
END;
```



# #  RECLAMOS

Los triggers son imposibles de hacer debido a que el examen no permite la adicion dse tablas y estas son ncesarias para el trigger 



