# Pobrezaequipo3
El objetivo de esta repositorio es entender la forma en que se deben realizar las consultas espaciales y no espaciales haciendo uso del lenguaje SQL (Structured Query Language) para los datos de pobreza entrema en las entidades federativas de la república mexicana .  
Vamos a explorar datos relacionados con porcentaje de población mayor al 30% con pobreza extrema municipal municipal .Con el objetivo de focalizar zonas prioritarias para la atención relacionándola con variables de importancia como son el rezago educativo, acceso a la seguridad social y carencia por acceso a la alimentación
Para comenzar se creó una una base de datos que se llame proyectofinal y no se creó un schema ya que se utilizará el schema public:

create database proyectofinal;
create extension postgis;

