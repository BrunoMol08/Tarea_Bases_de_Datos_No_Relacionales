# Tarea 02
Autor: Bruno Molina Zacatenco
Fecha: 22 de septiembre del 2021

Ejercicio Integrador

1. Qué idiomas base son los que más tuitean con hashtags? Cuál con URLs? Y con @?

```javascript
# Con Hashtags
db.tweets.aggregate([
	{$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
	{$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
	{$match:{"entities.hashtags":{$not:{$size:0}}}},
	{$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])

# Con URLs
db.tweets.aggregate([
	{$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
	{$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
	{$match:{"entities.urls":{$not:{$size:0}}}},
	{$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])

# Con User Mentions
db.tweets.aggregate([
	{$match:{"entities.user_mentions":{$not:{$size:0}}}},
	{$group: {_id:"$user.lang", "conteo": {$count:{}}}},
	{$lookup: {from:"primarydialects","localField":"_id","foreignField":"lang","as":"language"}},
	{$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
])
```

2. Qué idioma base es el que más hashtags usa en sus tuits?

```javascript
db.tweets.aggregate([
	{$group: {_id:"$user.lang", "totalHashtags": {$sum:{$size:"$entities.hashtags"}}}},
	{$lookup: {from:"primarydialects","localField":"_id","foreignField":"lang","as":"language"}},
	{$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
	{$project:{"language":0}},
	{$sort:{"totalHashtags":-1}}
])
```

4. Cómo podemos saber si los tuiteros hispanohablantes interactúan más en las noches?

Por la definición de hispanohablante, sabemos que es una persona que habla bien el español o es su lengua materna. Por lo tanto, podemos considerar los tweets en idioma español como tweets de hispanohablantes. Seguido de esto, vamos a contar cuantos tweets tenemos de los hispanohablantes
```javascript
db.tweets.aggregate([
{$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
{$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
{$match:{"user.lang":'es'}},
{$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])
```
> Esto nos dice, que tenemos 2385 tweets de hispanohablantes.

Después, con está consulta vamos a contar el numero de tweets de los hispanohablantes que fueron publicados, desde las 19 horas hasta las 23:59:59, consideraron que ese lapso de tiempo es noche.
```javascript
db.tweets.aggregate([
  {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
  {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
  {$match:{"user.lang":'es',"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([1]+[9]|[2]+[0-3])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])
```
> Esto nos dice, que 1407 tweets de los hispanohablantes son en la noche, es decir el 59% de los tweets fueron publicados en la noche.

Para corroborar, que el restante de los tweets fue publicado en otro horario hacemos está consulta.
```javascript
db.tweets.aggregate([
  {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
  {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
  {$match:{"user.lang":'es',"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([0]+[1-9]|[1]+[0-8])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])
```
> Esto nos dice, que 978 tweets de los hispanohablantes fueron publicados en un horario distinto al de la noche.

Se puede concluir, que los tuiteros hispanohablantes interactúan más en la noche.

5. Cómo podemos saber de dónde son los tuiteros que más tiempo tienen en la plataforma?

``` javascript

```

Para poder contestar las demás preguntas, vamos a estar siguiendo una serie de pasos para manejar una nueva colección de zonas horarias junto con su respectivo país. Además, de que se le va a ser una limpieza a esos datos.

 Primero, vamos a copiar la tabla de zonas horarias con su respectivo país de la página https://www.zeitverschiebung.net/es/all-time-zones.html, dentro de un archivo .csv al cual nombraremos zona_horaria.csv.

> Vamos a tener un archivo así, con 429 líneas
```
Zona horaria,País
Pacific/Midway,United States
Pacific/Niue,Niue
Pacific/Pago_Pago,American Samoa
America/Adak,United States
Pacific/Honolulu,United States
Pacific/Rarotonga,Cook Islands
Pacific/Tahiti,French Polynesia
Pacific/Marquesas,French Polynesia
America/Anchorage,United States
America/Juneau,United States
America/Metlakatla,United States
America/Nome,United States
America/Sitka,United States
America/Yakutat,United States
Pacific/Gambier,French Polynesia
...
```

 Después, debemos limpiar esos datos para quitarles el continente al que pertenece y tener solamente, la zona horaria y su país de forma que lo podamos cargar a mongodb. Corremos los siguientes códigos sobre el archivo.

```
sed -e 's/^Pacific.//gI' zona_horaria.csv | sed -e 's/^America.//gI' | sed -e 's/^Atlantic.//gI' | sed -e 's/^Antarctica.//gI' | sed -e 's/^Africa.//gI' | sed -e 's/^Europe.//gI' | sed -e 's/^Asia.//gI' | sed -e 's/^Indian.//gI' | sed -e 's/^Australia.//gI' | sed -e 's/^America.Argentina.//gI' | sed -e 's/^America.Kentucky.//gI' | sed -e 's/^America.Indiana.//gI' | sed -e 's/^America.North_Dakota.//gI' > zona_horaria_1.csv
```

> Después, hacemos esta otra modificación.
```
sed -e 's/^Argentina.//gI' zona_horaria_1.csv | sed -e 's/^Arctic.//gI' | sed -e 's/^Kentucky.//gI' | sed -e 's/^North_Dakota//gI' | sed -e 's/\///gI' > zona_horaria_2.csv
```

> Vamos a hacer que los '_' sea espacios en nuestro archivo
```
sed -e 's/_/ /gI' zona_horaria_2.csv > zona_horaria_limpia.csv
```
> Ya tenemos limpio el archivo.

Después, vamos a preparar el archivo para que lo podamos subir a mongodb.

```
sed -e 's/^Zona\sHoraria,País/time_zone,country/gI' zona_horaria_limpia.csv > zona_horaria_corregida.csv
```

Ahora vamos a hacer uso de un mongoimport que nos permite subir un archivo .csv

```
mongoimport -d trainingsessions -c countries --type CSV --file /home/bruno/Descargas/zona_horaria_corregida.csv --headerline
```
> En este momento, ya tenemos la nueva base de datos en mongodb.

Si se quiere consultar la base de datos, aquí está el link en

6. En intervalos de 7:00:00pm a 6:59:59am y de 7:00:00am a 6:59:59pm, de qué paises la mayoría de los tuits?

En esta primera consulta vamos a analizar los tweets, en el intervalo de 7:00:00pm a 6:59:59am, para conocer de qué país es la mayoría.
```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$match:{"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([1]+[9]|[2]+[0-3]|[0]+[0-6])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$countryy.country", "conteo": {$count:{}}}},
  {$sort: {"conteo":-1}}
])
```

Esto nos arroja la siguiente lista de resultados:
```javascript
[
  { _id: [], conteo: 4761 },
  { _id: [ 'United States or Canada' ], conteo: 3593 },
  { _id: [ 'Brazil' ], conteo: 1312 },
  { _id: [ 'Chile' ], conteo: 1117 },
  { _id: [ 'United Kingdom' ], conteo: 607 },
  { _id: [ 'Japan' ], conteo: 552 },
  { _id: [ 'Netherlands' ], conteo: 323 },
  { _id: [ 'Venezuela' ], conteo: 189 },
  { _id: [ 'Mexico' ], conteo: 183 },
  { _id: [ 'Germany' ], conteo: 145 },
  { _id: [ 'Indonesia' ], conteo: 139 },
  { _id: [ 'Argentina' ], conteo: 127 },
  { _id: [ 'France' ], conteo: 126 },
  { _id: [ 'Spain' ], conteo: 109 },
  { _id: [ 'Italy' ], conteo: 56 },
  { _id: [ 'Russia' ], conteo: 48 },
  { _id: [ 'Turkey' ], conteo: 48 },
  { _id: [ 'Sweden' ], conteo: 38 },
  { _id: [ 'Ireland' ], conteo: 33 },
  { _id: [ 'Colombia' ], conteo: 31 },
  ...
]
```
> El espacio [] quiere decir, o que no encontró el país o estaba en null el dato.
> Se puede corroborar, igual que en el caso anterior, que la mayoría de los tweets, en el intervalo, provienen de Estados Unidos o Canadá, seguido de Brasil.

En esta segunda consulta vamos a analizar los tweets, en el intervalo de 7:00:00am a 6:59:59pm, para conocer de que país es la mayoría.
```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$match:{"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([0]+[7-9]|[1]+[0-8])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$countryy.country", "conteo": {$count:{}}}},
  {$sort: {"conteo":-1}}
])
```

Esto nos arroja los siguientes resultados:
```javascript
[
  { _id: [], conteo: 3705 },
  { _id: [ 'United States or Canada' ], conteo: 2806 },
  { _id: [ 'Brazil' ], conteo: 1009 },
  { _id: [ 'Chile' ], conteo: 775 },
  { _id: [ 'Japan' ], conteo: 564 },
  { _id: [ 'United Kingdom' ], conteo: 449 },
  { _id: [ 'Netherlands' ], conteo: 179 },
  { _id: [ 'Indonesia' ], conteo: 170 },
  { _id: [ 'Mexico' ], conteo: 142 },
  { _id: [ 'Venezuela' ], conteo: 125 },
  { _id: [ 'Germany' ], conteo: 119 },
  { _id: [ 'France' ], conteo: 90 },
  { _id: [ 'Argentina' ], conteo: 85 },
  { _id: [ 'Spain' ], conteo: 63 },
  { _id: [ 'South Korea' ], conteo: 40 },
  { _id: [ 'Italy' ], conteo: 39 },
  { _id: [ 'Russia' ], conteo: 37 },
  { _id: [ 'Singapore' ], conteo: 32 },
  { _id: [ 'Belgium' ], conteo: 30 },
  { _id: [ 'Turkey' ], conteo: 28 },
  ...
]

```
> El espacio [] quiere decir, o que no encontró el país o estaba en null el dato.
>Se puede corroborar, igual que en el caso anterior, que la mayoría de los tweets, en el intervalo, provienen de Estados Unidos o Canadá, seguido de Brasil.

7. De qué país son los tuiteros más famosos de nuestra colección?

Esta consulta nos va a arrojar a los 15 más seguidos en twitter, con su respectivo país y la cantidad de seguidores.
```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$group: {_id:{"país":"$countryy.country","seguidores":"$user.friends_count"}}},
  {$sort: {"_id.seguidores":-1}},
  {$limit : 15}
])
```

Esto nos arroja los siguientes resultados:
```javascript
[
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 295035 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 52031 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 51869 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 50513 } },
  { _id: { 'país': [ 'Brazil' ], seguidores: 39464 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 37195 } },
  { _id: { 'país': [ 'United Kingdom' ], seguidores: 37076 } },
  { _id: { 'país': [ 'Venezuela' ], seguidores: 35951 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 31532 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 30857 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 30756 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 30558 } },
  { _id: { 'país': [ 'Brazil' ], seguidores: 28142 } },
  { _id: { 'país': [ 'Brazil' ], seguidores: 28045 } },
  { _id: { 'país': [ 'United States or Canada' ], seguidores: 27231 } }
]
```
> Podemos ver, que la mayoría de los tuiteros más famosos de nuestra colección provienen de Estados Unidos o de Canadá.
