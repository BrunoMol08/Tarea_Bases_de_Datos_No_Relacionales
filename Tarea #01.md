# Tarea #01
Autor: Bruno Molina Zacatenco
Fecha: 4 de septiembre del 2021

Ejercicio 12 al 32 de la BD Reviews Collection Restaurants

12. Escribe una función find() para encontrar los restaurantes que no preparan ninguna cocina del continente americano y lograron una puntuación superior a 70 y se ubicaron en la longitud inferior a -65.754168.

```
    db.restaurants.find({"cuisine":{$not:/American/},"grades.score":{$gt:70},"address.coord.0":{$lt:-65.754168}},{"cuisine":1,"grades":1,"address.coord":1})
```

13. Escribe una función find() para encontrar los restaurantes que no preparan ninguna cocina del continente americano y obtuvieron una calificación de 'A' que no pertenece al distrito de Brooklyn. El documento debe mostrarse según la cocina en orden descendente.

```
    db.restaurants.find({"cuisine":{$not:/(American|Hamburgers)/}, "grades.grade":'A', "borough":{$ne:'Brooklyn'}},{"cuisine":1, "grades.grade":1, "borough":1}).sort({"cuisine":-1})
```