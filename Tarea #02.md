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

Primero, vamos a contar cuantos tweets tenemos de los hispanohablantes
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

```javascript

```

6. En intervalos de 7:00:00pm a 6:59:59am y de 7:00:00am a 6:59:59pm, de qué paises la mayoría de los tuits?

```javascript

```

7. De qué país son los tuiteros más famosos de nuestra colección?

Esta consulta nos va a arrojar a los 15 más seguidos en twitter, con su respectivo país y la cantidad de seguidores.
```javascript
db.tweets.aggregate([
  {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
  {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
  {$group: {_id:{"país":"$fulllocale.languages","seguidores":"$user.friends_count"}}},
  {$sort: {"_id.seguidores":-1}},
  {$limit : 15}
])
```
> Podemos ver, que los 15 tuiteros más famosos de nuestra colección provienen de Estados Unidos (US).
