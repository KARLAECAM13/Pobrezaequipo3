# Pobreza_equipo03
El objetivo de esta repositorio es entender la forma en que se deben realizar las consultas espaciales y no espaciales haciendo uso del lenguaje SQL (Structured Query Language) para los datos de pobreza entrema en las entidades federativas de la república mexicana .  

Vamos a explorar datos relacionados con porcentaje de población mayor al 30% con pobreza extrema municipal municipal .Con el objetivo de focalizar zonas prioritarias para la atención relacionándola con variables de importancia como son el rezago educativo, acceso a la seguridad social y carencia por acceso a la alimentación. Los mapas interactivos puedes visitarlos en [MapasEquipo03](https://drive.google.com/drive/folders/1OB1ZVg60HA2GyrWp8SYETSWhnYRjEZWs).

Para comenzar se creó una una base de datos que se llame proyectofinal y  un schema, permiten organizar los objetos de la base de datos, por ejemplo, las tablas en grupos lógicos para hacerlos más manejables, se ctiva la extensión PostGIS en la base de datos para habilitar soporte espacial como las geometrías , funciones espaciales, etc.:
```SQL
-- CREAR LA BASE DE DATOS
create database proyectofinal;
-- HABILITAR LA EXTENSIÓN POSTGIS
create extension postgis;
--Create schema
create schema proyectofinal;
```

## Preparando los datos 
Cuando se trabaja con datos tanto espaciales como no espaciales es necesario que los datos cumplan con tres condiciones importantes: contar como llaves primarias y secundarias, tener índices espaciales y definir la proyección espacial idonea para trabajar con ellos.

Los datos de población y los polígonos de las manzanas se encuentran separados, es por eso que vamos a realizar una unión.
Se cambia el nombre de la columna"clave de municipio" a "cvegeo" para estandarizarla y facilitar los JOINs
```SQL
ALTER TABLE "Concentrado_indicadores_de_pobreza_2015-2020_Filtrado"
RENAME COLUMN "clave de municipio" TO cvegeo;
```
Se realiza una consulta para unir la tabla de geometría de municipios con la tabla de indicadores de pobreza usando como clave común la columna "cvegeo"
```SQL
SELECT
   m.*,  -- todo de la tabla con geometría
   p.*   -- todo de la tabla de pobreza
FROM
   "00mun" AS m
JOIN
   "Concentrado_indicadores_de_pobreza_2015-2020_Filtrado" AS p
ON
   m.cvegeo = p.cvegeo;
```
Consulta para ver los nombres de las columnas de la tabla "00mun"
```SQL
SELECT column_name
FROM information_schema.columns
WHERE table_name = '00mun';
```
 Consulta para ver los nombres de las columnas de la tabla "Concentrado_indicadores_de_pobreza_2015-2020_Filtrado"
```SQL
SELECT column_name
FROM information_schema.columns
WHERE table_name = 'Concentrado_indicadores_de_pobreza_2015-2020_Filtrado'
Se realiza la consulta para ver los nombres de las columnas de la tabla "Concentrado_indicadores_de_pobreza_2015-2020_Filtrado"
SELECT column_name
FROM information_schema.columns
WHERE table_name = 'Concentrado_indicadores_de_pobreza_2015-2020_Filtrado';
```
Se crea una nueva tabla con el resultado del JOIN entre geometrías y una versión diferente del concentrado de pobreza, Se renombran algunas columnas con alias más claros
```SQL
CREATE TABLE Concentrado_IndPob2015_2020_mun2 AS
SELECT
   m.id AS id_mun,
   m.geom,
   m.cvegeo,
   m.cve_ent,
   m.cve_mun,
   m.nomgeo,
  
   p.id AS id_pobreza,
   p.clave_entidad,
   p.entidad_federativa,
   p.municipio,
   p."pobtot_2015",
   p."pobtot_2020",
   p."pobrezaext_%2015",
   p."pobrezaext_%2020",
   p."rezagoedu_%2015",
   p."rezagoedu_%2020",
   p."segsocial_%2015",
   p."segsocial_%2020",
   p."acsalimen_%2015",
   p."acsalimen_%2020"
FROM
   "00mun" AS m
JOIN
   "concentrado_indicadores_de_pobreza_2015_2020_filtrado2" AS p
ON
   m.cvegeo = p.cvegeo;
```
Se crea otra versión de la tabla combinada incluyendo geometría y los mismos indicadores de pobreza, con diferente nombre de tabla de origen   
```SQL
CREATE TABLE concentrado_indpob2015_2020_mun2 AS
SELECT
   m.id AS id_mun,
   m.geom,
   m.cvegeo,
   m.cve_ent,
   m.cve_mun,
   m.nomgeo,
   p.id AS id_indicador,
   p.clave_entidad,
   p.entidad_federativa,
   p.municipio,
   p.pobtot_2015,
   p.pobtot_2020,
   p."pobrezaext_%2015",
   p."pobrezaext_%2020",
   p."rezagoedu_%2015",
   p."rezagoedu_%2020",
   p."segsocial_%2015",
   p."segsocial_%2020",
   p."acsalimen_%2015",
   p."acsalimen_%2020"
FROM
   "00mun" AS m
JOIN
   "concentrado_indicadores_de_pobreza_2015-2020_Filtrado2" AS p
ON
   m.cvegeo = p.cvegeo;
```
Se realiza la consulta para identificar si hay claves de municipio con longitudes distintas (por ejemplo, claves incompletas)
-- útil para detectar problemas en los JOINs
```SQL
SELECT DISTINCT cvegeo, LENGTH(cvegeo)
FROM "00mun"
ORDER BY LENGTH(cvegeo);
```
se realiza la limpieza de datos: reemplaza los valores 'n.d.' por '0' en los indicadores.
Esto es necesario para poder convertir los valores a tipo numérico más adelante,
valores n.d a cero y convertir a floats

```SQL
UPDATE "concentrado_indpob2015_2020_mun2"
SET
 "pobrezaext_%2015" = CASE WHEN "pobrezaext_%2015" = 'n.d.' THEN '0' ELSE "pobrezaext_%2015" END,
 "pobrezaext_%2020" = CASE WHEN "pobrezaext_%2020" = 'n.d.' THEN '0' ELSE "pobrezaext_%2020" END,
 "rezagoedu_%2015" = CASE WHEN "rezagoedu_%2015" = 'n.d.' THEN '0' ELSE "rezagoedu_%2015" END,
 "rezagoedu_%2020" = CASE WHEN "rezagoedu_%2020" = 'n.d.' THEN '0' ELSE "rezagoedu_%2020" END,
 "segsocial_%2015" = CASE WHEN "segsocial_%2015" = 'n.d.' THEN '0' ELSE "segsocial_%2015" END,
 "segsocial_%2020" = CASE WHEN "segsocial_%2020" = 'n.d.' THEN '0' ELSE "segsocial_%2020" END,
 "acsalimen_%2015" = CASE WHEN "acsalimen_%2015" = 'n.d.' THEN '0' ELSE "acsalimen_%2015" END,
 "acsalimen_%2020" = CASE WHEN "acsalimen_%2020" = 'n.d.' THEN '0' ELSE "acsalimen_%2020" END;

```
  
Conversión de tipo de dato: convierte los indicadores que están en texto (varchar) a tipo numérico (double precision)
Esto permite realizar cálculos estadísticos o mapas temáticos cuantitativos
```SQL
ALTER TABLE "concentrado_indpob2015_2020_mun2"
ALTER COLUMN "pobrezaext_%2015" TYPE double precision USING "pobrezaext_%2015"::double precision,
ALTER COLUMN "pobrezaext_%2020" TYPE double precision USING "pobrezaext_%2020"::double precision,
ALTER COLUMN "rezagoedu_%2015" TYPE double precision USING "rezagoedu_%2015"::double precision,
ALTER COLUMN "rezagoedu_%2020" TYPE double precision USING "rezagoedu_%2020"::double precision,
ALTER COLUMN "segsocial_%2015" TYPE double precision USING "segsocial_%2015"::double precision,
ALTER COLUMN "segsocial_%2020" TYPE double precision USING "segsocial_%2020"::double precision,
ALTER COLUMN "acsalimen_%2015" TYPE double precision USING "acsalimen_%2015"::double precision,
ALTER COLUMN "acsalimen_%2020" TYPE double precision USING "acsalimen_%2020"::double precision;

```
Se crea el Filtro: se crean nuevas tablas con municipios que tienen pobreza extrema mayor o igual al 30% en 2015 y 2020.
Esto sirve para identificar los municipios con mayor vulnerabilidad y tener umbrales pobreza extrema >=30
```SQL
CREATE TABLE "pobrezaext_alta_2015" AS
SELECT *
FROM "concentrado_indpob2015_2020_mun2"
WHERE "pobrezaext_%2015" >= 30;
CREATE TABLE "pobrezaext_alta_2020" AS
SELECT *
FROM "concentrado_indpob2015_2020_mun2"
WHERE "pobrezaext_%2020" >= 30;

```
Se agrega dos nuevas columnas tipo float8 (número decimal de doble precisión).
A la tabla 'concentrado_indpob2015_2020_mun2', para almacenar los valores de pobreza total (%) de los años 2015 y 2020.
```SQL
ALTER TABLE concentrado_indpob2015_2020_mun2
ADD COLUMN "pobreza%2015" float8,
ADD COLUMN "pobreza%2020" float8;
```
Actualiza la tabla 'para_pobrezatot_columns' para reemplazar los valores "n.d." (no disponibles) en la columna 'pobreza%2015' por '0'.

```SQL
UPDATE para_pobrezatot_columns
SET "pobreza%2015" = '0'
WHERE "pobreza%2015" = 'n.d.';
```
Se hace lo mismo que la línea anterior pero para el año 2020.
```SQL
UPDATE para_pobrezatot_columns
SET "pobreza%2020" = '0'
WHERE "pobreza%2020" = 'n.d.';
```
Se Reemplazan valores vacíos (cadenas vacías, incluso si tienen espacios)en la columna 'pobreza%2015' por '0'.
```SQL
UPDATE para_pobrezatot_columns
SET "pobreza%2015" = '0'
WHERE trim("pobreza%2015") = '';
```
Igual que la instrucción anterior, pero aplicada a 'pobreza%2020'.
```SQL
UPDATE para_pobrezatot_columns
SET "pobreza%2020" = '0'
WHERE trim("pobreza%2020") = '';
```
Se cambia el tipo de datos de las columnas 'pobreza%2015' y 'pobreza%2020'de tipo texto a tipo float8, reemplazando las comas por puntos decimales.
Esto es necesario porque en algunos sistemas los porcentajes vienen con coma como separador decimal (ej. "25,4" en lugar de "25.4").
```SQL
ALTER TABLE para_pobrezatot_columns
ALTER COLUMN "pobreza%2015" TYPE FLOAT8 USING replace("pobreza%2015", ',', '.')::FLOAT8,
ALTER COLUMN "pobreza%2020" TYPE FLOAT8 USING replace("pobreza%2020", ',', '.')::FLOAT8;
```
Finalmente, copia los valores numéricos ya procesados de pobreza desde 'para_pobrezatot_columns' hacia la tabla principal'concentrado_indpob2015_2020_mun2', igualando por la clave geográfica (cvegeo).
```SQL
UPDATE concentrado_indpob2015_2020_mun2 AS c
SET
 "pobreza%2015" = p."pobreza%2015",
 "pobreza%2020" = p."pobreza%2020"
FROM para_pobrezatot_columns AS p
WHERE c.cvegeo = p.cvegeo;
```

## Consultas con los datos para el análisis de Pobreza extrema
Consultas con los datos para el análisis de Municipios con  30% o más de población con pobreza extrema 

Para realizar la consulta ¿Cuáles son los municipios que tienen más pobreza extrema del 2015 y 2020?.
**importante seleccionar las columnas que quiero obtener en *select* y
 en *from* la tabla donde obtendré los datos puede ser del 2015 o del 2020).**

Para realizar la consulta ¿Cuáles son los municipios que tienen más pobreza en 2015?
```SQL
select nomgeo, pobrezaext, entidad_fe
from public.pobreza_extrema_alta_2015 pa
order by pobrezaext desc
limit 10;
```
Para realizar la consulta ¿Cuáles son los estados con más municipios en pobreza extrema en 2015?
```SQL
select entidad_fe, count(*) as
veces_apar
from public.pobreza_extrema_alta_2015 pa
group by entidad_fe
order by veces_apar desc;
```

Para Realizar la consulta ¿De esos estados cual es el municipio con mayor  pobreza extrema ?

```SQL
with municipiopobrezaext as (
select entidad_fe,
		nomgeo,
		pobrezaext,
		rezagoedu_,
		row_number() over (partition by entidad_fe
		order by pobrezaext desc) as rn
from public.pobreza_extrema_alta_2015)
select entidad_fe,
		nomgeo,
		pobrezaext,
		rezagoedu_
from municipiopobrezaext
where rn = 1
order by pobrezaext desc;
```

Podemos realizar otras consultas de una entidad federativa epecífic, ¿Cuales son los municipios con mayor pobreza extrema en Guerrero?
```SQL
select entidad_fe, municipio, pobrezaext, rezagoedu_
from public.pobreza_extrema_alta_2015 pa
where pa.entidad_fe = 'Guerrero'
group by pa.municipio, pa.entidad_fe, pobrezaext, rezagoedu_
order by pobrezaext desc
```

o escribiendo el número del estado en where para obtener datos de Yucatán.
```SQL
select entidad_fe,  municipio,  rezagoedu_ ,acsalimen_ ,pobrezaext 
from public.pobreza_extrema_alta_2020
where entidad_fe = '31'
group by entidad_fe, municipio,  rezagoedu_,acsalimen_ ,pobrezaext 
order by acsalimen_ desc;
```
