# Pobrezae_equipo03
El objetivo de esta repositorio es entender la forma en que se deben realizar las consultas espaciales y no espaciales haciendo uso del lenguaje SQL (Structured Query Language) para los datos de pobreza entrema en las entidades federativas de la república mexicana .  
Vamos a explorar datos relacionados con porcentaje de población mayor al 30% con pobreza extrema municipal municipal .Con el objetivo de focalizar zonas prioritarias para la atención relacionándola con variables de importancia como son el rezago educativo, acceso a la seguridad social y carencia por acceso a la alimentación.
Para comenzar se creó una una base de datos que se llame proyectofinal y no se creó un schema ya que se utilizará el schema public:
```SQL
-- CREAR LA BASE DE DATOS
create database proyectofinal;
-- HABILITAR LA EXTENSIÓN POSTGIS
create extension postgis;
```

--Consultas con los datos para el análisis de Municipios con  30% o más de población con pobreza extrema --

--Para realizar la consulta ¿Cuáles son los municipios que tienen más pobreza extrema del 2015 y 2020?--
(importante seleccionar las columnas que quiero obtener y en from la tabla donde obtendré los datos puede ser del 2015 o del 2020).

##Consultas con los datos para el análisis de 2015 (Pobreza extrema)--

```SQL
--Para realizar la consulta ¿Cuáles son los municipios que tienen más pobreza en 2015?--
(importante seleccionar las columnas que quiero obtener y en from la tabla donde obtendré los datos puede ser del 2015 o del 2020).
select nomgeo, pobrezaext, entidad_fe
from public.pobreza_extrema_alta_2015 pa
order by pobrezaext desc
limit 10;
````
