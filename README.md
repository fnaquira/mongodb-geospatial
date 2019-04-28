# Uso de Índices Geo Espaciales en MongoDB

El siguiente tutorial es mi resumen de la documentación oficial de MongoDB, cuyo link dejo aquí:

[Find Restaurants with Geospatial Queries](https://docs.mongodb.com/manual/tutorial/geospatial-tutorial/)

La indexación Geo Espacial de MongoDB nos permite ejecutar eficientemente sentencias (querys) espaciales que contienen formas geo espaciales o puntos en un plano. Para mostrar dicho funcionamiento, vamos a mostrar dicha funcionalidad mediante un ejemplo práctico.
Digamos que está diseñando una aplicación mobil para ayudar a sus usuarios a encontrar restaurantes en Nueva York. La aplicación debe:

- Determinar el vecindario actual del usuario.
- Mostrar el número de restaurante dentro del vecindario actual.
- Encontrar restaurantes dentro de una distancia especificada a la ubicación del usuario.

# Pre requisitos

Descargue por favor los siguientes datasets:

- [neighborhoods.json](https://raw.githubusercontent.com/mongodb/docs-assets/geospatial/neighborhoods.json)
- [restaurants.json](https://raw.githubusercontent.com/mongodb/docs-assets/geospatial/restaurants.json)

Después de descargarlos, importelos a su base de datos como cualquier json:

```
mongoimport <path to restaurants.json> -c restaurants
mongoimport <path to neighborhoods.json> -c neighborhoods
```

Ahora crearemos los índices geo espaciales dentro del **shell de MongoDB**

```
db.restaurants.createIndex({ location: "2dsphere" })
db.neighborhoods.createIndex({ geometry: "2dsphere" })
```

# Explorando la data

Veamos como luce uno de los documentos creados dentro de restaurants

```
db.restaurants.findOne()
```

La sentencia ejecutada nos devolverá un documento como este:

```
{
   location: {
      type: "Point",
      coordinates: [-73.856077, 40.848447]
   },
   name: "Morris Park Bake Shop"
}
```
