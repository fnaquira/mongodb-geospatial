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

Veamos como luce uno de los documentos creados dentro de **restaurants**

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

Dicho punto corresponde a la siguiente figura:
![Restaurante](https://github.com/fnaquira/mongodb-geospatial/blob/master/geospatial-single-point.png)

Ahora veamos como luce un vecindario, es decir, un documento de **neighborhoods**

```
db.neighborhoods.findOne()
```

Dicho documento tiene un atributo coordenadas que definen los límites del vecindario:

```
{
   geometry: {
      type: "Polygon",
      coordinates: [[
         [ -73.99, 40.75 ],
         ...
         [ -73.98, 40.76 ],
         [ -73.99, 40.75 ]
      ]]
    },
    name: "Hell's Kitchen"
}
```

Lo cual luciría como una figura así:
![Vecindario](https://github.com/fnaquira/mongodb-geospatial/blob/master/geospatial-polygon-hells-kitchen.png)

# Encontrando el vecindario actual

Asumiento que el dispositivo mobil nos devuelve las siguientes coordenadas:

- Longitud: -73.93414657
- Latitud: 40.82302903
  Realizaremos el siguiente query, que utiliza la opción \$geoIntersects (que busca un polígono que contenga al punto indicado)

```
db.neighborhoods.findOne({ geometry: { $geoIntersects: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] } } } })
```

El cual nos devolverá el vecindario donde nos encontramos:

```
{
    "_id" : ObjectId("55cb9c666c522cafdb053a68"),
    "geometry" : {
        "type" : "Polygon",
        "coordinates" : [
            [
                [
                    -73.93383000695911,
                    40.81949109558767
                ],
                ...
            ]
        ]
    },
    "name" : "Central Harlem North-Polo Grounds"
}
```

# Encontrando todos los restaurantes del vecindario
