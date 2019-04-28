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

![Restaurante](https://github.com/fnaquira/mongodb-geospatial/blob/master/geospatial-single-point.png?raw=true)

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

![Vecindario](https://github.com/fnaquira/mongodb-geospatial/blob/master/geospatial-polygon-hells-kitchen.png?raw=true)

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

Combinaremos la sentencia anterior y buscaremos todos los restaurantes que se encuentran en ese punto

```
var neighborhood = db.neighborhoods.findOne( { geometry: { $geoIntersects: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] } } } } )
db.restaurants.find( { location: { $geoWithin: { $geometry: neighborhood.geometry } } } ).count()
```

Esta sentencia nos retornará 127 restaurantes en el vecindario indicado, que pueden ser visualizados aquí:

![Vecindario](https://github.com/fnaquira/mongodb-geospatial/blob/master/geospatial-all-restaurants.png?raw=true)

# Encontrar restaurante dentro de cierta distancia

para realizar esto, podemos utilizar dos métodos: **\$geoWithin** con **\$centerSphere** para obtener los resultados de forma desordenada, o **\$nearSphere** con **\$masDistance**

## Desordenados con \$geoWithin

Utilizaremos **\$geoWithin** y **\$centerSphere**, que denota una región circular especificando el centro y el radio en radianes.

```
db.restaurants.find({ location:
   { $geoWithin:
      { $centerSphere: [ [ -73.93414657, 40.82302903 ], 5 / 6378.1 ] } } })
```

El segundo argumento de \$centerSphere acepta el radio en radianes, por lo que debemos dividirlo por el radio de la tierra en kilómetros (el radio de la tierra en millas es de 3963.2).

## Ordenados con \$nearSphere

Ahora utilizaremos **nearSphere** para especificar una distancia máxima en metros. Esto retornará todos los restaurantes dentro de cinco kilómetros ordenados desde el más cercano al más lejano.

```
db.restaurants.find({ location: { $nearSphere: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] }, $maxDistance: 5 * 1000 } } })
```

# Conclusiones

Como podemos ver, MongoDB nos permite hacer búsqueda muy potentes, sobre todo al momento de tener data geo espacial.
Esto brinda una gran oportunidad si estamos desarrollando una aplicación que de recomendaciones al usuario según su ubicación ggeográfica.
