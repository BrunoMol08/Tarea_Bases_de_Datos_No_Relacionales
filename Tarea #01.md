# Tarea #01
Autor: Bruno Molina Zacatenco
Fecha: 4 de septiembre del 2021

Ejercicio 12 al 32 de la BD Reviews Collection Restaurants

12. Escribe una función find() para encontrar los restaurantes que no preparan ninguna cocina del continente americano y lograron una puntuación superior a 70 y se ubicaron en la longitud inferior a -65.754168.

```javascript
db.restaurants.find({"cuisine":{$not:/American/},"grades.score":{$gt:70},"address.coord.0":{$lt:-65.754168}},{"cuisine":1,"grades":1,"address.coord":1})
```

13. Escribe una función find() para encontrar los restaurantes que no preparan ninguna cocina del continente americano y obtuvieron una calificación de 'A' que no pertenece al distrito de Brooklyn. El documento debe mostrarse según la cocina en orden descendente.

```javascript
db.restaurants.find({"cuisine":{$not:/(American|Hamburgers)/}, "grades.grade":'A', "borough":{$ne:'Brooklyn'}},{"cuisine":1, "grades.grade":1, "borough":1}).sort({"cuisine":-1})
```

14. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen 'Wil' como las primeras tres letras de su nombre.

```javascript
db.restaurants.find({"name":/^Wil/},{"restaurant_id":1, "name":1, "borough":1, "cuisine":1, "_id":0})
```

15. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen "ces" como las últimas tres letras de su nombre.

```javascript
db.restaurants.find({"name":/ces$/},{"restaurant_id":1, "name":1, "borough":1, "cuisine":1,"_id":0})
```

16. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen 'Reg' como tres letras en algún lugar de su nombre.
```javascript
db.restaurants.find({"name":/Reg/},{"restaurant_id":1, "name":1, "borough":1, "cuisine":1,"_id":0})
```

17. Escribe una función find() para encontrar los restaurantes que pertenecen al municipio del Bronx y que prepararon platos estadounidenses o chinos.

```javascript
db.restaurants.find({"borough":'Bronx', "cuisine":/^(American|Hamburgers|Chinese)/},{"borough":1, "cuisine":1})
```

18. Escribe una función find() para encontrar la identificación del restaurante, el nombre, el municipio y la cocina de los restaurantes que pertenecen al municipio de Staten Island o Queens o Bronx or Brooklyn.

```javascript
db.restaurants.find({"borough":/^(Staten Island$|Queens|Bronx|Brooklyn)/},{"restaurant_id":1, "name":1, "borough":1, "cuisine":1})
```

19. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que no pertenecen al municipio de Staten Island o Queens o Bronx or Brooklyn.

```javascript
db.restaurants.find({"borough":{$not:/^(Staten Island$|Queens|Bronx|Brooklyn)/}},{"restaurant_id":1, "name":1, "borough":1, "cuisine":1})
```

20. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que obtuvieron una puntuación que no sea superior a 10.

```javascript
db.restaurants.find({"grades.score":{$lt:10}},{"restaurant_id":1,"name":1,"borough":1,"cuisine":1,"grades.score":1})
```

21. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que prepararon platos excepto 'Americano' y 'Chinees' o el nombre del restaurante comienza con la letra 'Wil'.

```javascript
db.restaurants.find({"cuisine":{$not:/^(American|Chinese)/},"name":{$not:/^Wil/}},{"restaurant_id":1,"name":1,"borough":1,"cuisine":1})
```

22. Escribe una función find() para encontrar el ID del restaurante, el nombre y las calificaciones de los restaurantes que obtuvieron una calificación de "A" y obtuvieron una puntuación de 11 en un ISODate "2014-08-11T00: 00: 00Z" entre muchas de las fechas de la encuesta.

```javascript
db.restaurants.find({"grades":{$elemMatch:{"grade":'A',"date":ISODate("2014-08-11T00:00:00.000Z")}}},{"restaurant_id":1,"name":1,"grades":1})
```

23. Escribe una función find() para encontrar el ID del restaurante, el nombre y las calificaciones de aquellos restaurantes donde el segundo elemento de la matriz de calificaciones contiene una calificación de "A" y una puntuación de 9 en un ISODate "2014-08-11T00: 00: 00Z".

```javascript
db.restaurants.find({"grades.1.grade":'A',"grades.1.date":ISODate("2014-08-11T00:00:00.000Z"),"grades.1.score":9},{"restaurant_id":1,"name":1,"grades":1})
```

24. Escribe una función find() para encontrar el ID del restaurante, el nombre, la dirección y la ubicación geográfica para aquellos restaurantes donde el segundo elemento de la matriz de coordenadas contiene un valor que sea más de 42 y hasta 52 ..

```javascript
db.restaurants.find({"address.coord.1":{$gte:42,$lte:52}},{"restaurant_id":1,"name":1,"address":1})
```

25. Escribe una función find() para organizar el nombre de los restaurantes en orden ascendente junto con todas las columnas.

```javascript
db.restaurants.find().sort({"name":1})
```

26. Escribe una función find() para organizar el nombre de los restaurantes en orden descendente junto con todas las columnas.

```javascript
db.restaurants.find().sort({"name":-1})
```

27. Escribe una función find() para organizar el nombre de la cocina en orden ascendente y para ese mismo distrito de cocina debe estar en orden descendente.

```javascript
db.restaurants.find().sort({"cuisine":1,"borough":-1})
```

28. Escribe una función find() para saber si todas las direcciones contienen la calle o no.
