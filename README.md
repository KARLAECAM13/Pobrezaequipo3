# Pobreza_equipo03
El objetivo de esta repositorio es entender la forma en que se deben realizar las consultas espaciales y no espaciales haciendo uso del lenguaje SQL (Structured Query Language) para los datos de pobreza entrema en las entidades federativas de la república mexicana .  
Vamos a explorar datos relacionados con porcentaje de población mayor al 30% con pobreza extrema municipal municipal .Con el objetivo de focalizar zonas prioritarias para la atención relacionándola con variables de importancia como son el rezago educativo, acceso a la seguridad social y carencia por acceso a la alimentación.
Para comenzar se creó una una base de datos que se llame proyectofinal y no se creó un schema ya que se utilizará el schema public:
```SQL
-- CREAR LA BASE DE DATOS
create database proyectofinal;
-- HABILITAR LA EXTENSIÓN POSTGIS
create extension postgis;
```
## Preparando los datos 
Cuando se trabaja con datos tanto espaciales como no espaciales es necesario que los datos cumplan con tres condiciones importantes: contar como llaves primarias y secundarias, tener índices espaciales y definir la proyección espacial idonea para trabajar con ellos.

Los datos de población y los polígonos de las manzanas se encuentran separados, es por eso que vamos a realizar una unión.

## Consultas con los datos para el análisis de Pobreza extrema
Consultas con los datos para el análisis de Municipios con  30% o más de población con pobreza extrema 

Para realizar la consulta ¿Cuáles son los municipios que tienen más pobreza extrema del 2015 y 2020?
**importante seleccionar las columnas que quiero obtener en *select* y
 en *from* la tabla donde obtendré los datos puede ser del 2015 o del 2020).**

```SQL
--Para realizar la consulta ¿Cuáles son los municipios que tienen más pobreza en 2015?--
select nomgeo, pobrezaext, entidad_fe
from public.pobreza_extrema_alta_2015 pa
order by pobrezaext desc
limit 10;
```
Para realizar la consulta ¿Cuáles son los estados con más municipios en pobreza extrema en 2015?--
```SQL
select entidad_fe, count(*) as
veces_apar
from public.pobreza_extrema_alta_2015 pa
group by entidad_fe
order by veces_apar desc;
```

Para Realizar la consulta ¿Qué municipios son los más pobres de cada estado y agregar otras variables de interés?--

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

Podemos realizar otras consultas como ¿De cada estado quiero obtener los 10 municipios más pobres?
```SQL
select entidad_fe, municipio, pobrezaext, rezagoedu_
from public.pobreza_extrema_alta_2015 pa
where pa.entidad_fe = 'Guerrero'
group by pa.municipio, pa.entidad_fe, pobrezaext, rezagoedu_
order by pobrezaext desc
select entidad_fe, nomgeo, SUM(pobrezaext) as
total_pobrezaext
from public.pobreza_extrema_alta_2015 pa
group by entidad_fe, nomgeo
order by total_pobrezaext desc;
```


