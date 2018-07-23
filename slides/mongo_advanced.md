# Mongo - продвинутые техники

Отсутствуют джойны. Чтобы как-то моделировать джойны предлагается особенным образом формировать документы - чтобы сохранять связи между различными коллекциями

<pre>
> db.posts.insert({day: 'Wed', author: ObjectId("5b571bf081d67789509607f1")})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({day: 'Sat', author: ObjectId("5b571bf081d67789509607f1")})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({day: 'Sat', author: ObjectId("5b57207bbff183a0a45dcfb3")})
WriteResult({ "nInserted" : 1 })
> db.posts.find()
{ "_id" : ObjectId("5b57392b81d67789509607f2"), "day" : "Wed", "author" : ObjectId("5b571bf081d67789509607f1") }
{ "_id" : ObjectId("5b57393481d67789509607f3"), "day" : "Sat", "author" : ObjectId("5b571bf081d67789509607f1") }
{ "_id" : ObjectId("5b57394381d67789509607f4"), "day" : "Sat", "author" : ObjectId("5b57207bbff183a0a45dcfb3") }
> db.posts.find({author: ObjectId("5b571bf081d67789509607f1")})
{ "_id" : ObjectId("5b57392b81d67789509607f2"), "day" : "Wed", "author" : ObjectId("5b571bf081d67789509607f1") }
{ "_id" : ObjectId("5b57393481d67789509607f3"), "day" : "Sat", "author" : ObjectId("5b571bf081d67789509607f1") }
</pre>

ПО умолчанию возвращаются все поля документа. Это поведение можно изменить:
<pre>
> db.posts.find({author: ObjectId("5b571bf081d67789509607f1")}, {day: 1})
{ "_id" : ObjectId("5b57392b81d67789509607f2"), "day" : "Wed" }
{ "_id" : ObjectId("5b57393481d67789509607f3"), "day" : "Sat" }
</pre>

Служебное поле _id возвращается всегда - его можно выключить в явном виде

<pre>
> db.posts.find({author: ObjectId("5b571bf081d67789509607f1")}, {day: 1, _id: 0})
{ "day" : "Wed" }
{ "day" : "Sat" }
</pre>

В остальных случаях включение и исключение полей нельзя смешивать.

Порядок выдачи изменяется командой .sort():

<pre>
> db.posts.find({author: ObjectId("5b571bf081d67789509607f1")}, {day: 1, _id: 0}).sort({day: 1})
{ "day" : "Sat" }
{ "day" : "Wed" }

или в обратном порядке

> db.posts.find({author: ObjectId("5b571bf081d67789509607f1")}, {day: 1, _id: 0}).sort({day: -1})
{ "day" : "Wed" }
{ "day" : "Sat" }
</pre>

Так же можно управлять количеством записей в выдаче и смещением от начала списка

<pre>
полная выдача
> db.posts.find({author: ObjectId("5b571bf081d67789509607f1")}, {day: 1, _id: 0}).sort({day: -1})
{ "day" : "Wed" }
{ "day" : "Sat" }

Задаём смещение
> db.posts.find({author: ObjectId("5b571bf081d67789509607f1")}, {day: 1, _id: 0}).sort({day: -1}).limit(2).skip(1)
{ "day" : "Sat" }
</pre>

Гибкость курсора позволяет управлять нагрузкой на Mongo, так как курсор умеет делать смещения без непосредственного чтения записей

## Загрузка JSON

Для загрузки JSON выполним скрипт формирования JSON - это будут случайные картинки собачек, получаемых по API

<pre>
# rm -f /data/test.json; for i in $(seq 100 $END); do curl "https://dog.ceo/api/breeds/image/random">>/data/test.json>>/data/test.json; done;
</pre>

На выходе получаем JSON-файл
<pre>
/ # head /data/test.json;
{"status":"success","message":"https:\/\/images.dog.ceo\/breeds\/weimaraner\/n02092339_1013.jpg"},
{"status":"success","message":"https:\/\/images.dog.ceo\/breeds\/basenji\/n02110806_5971.jpg"},
{"status":"success","message":"https:\/\/images.dog.ceo\/breeds\/dingo\/n02115641_5815.jpg"},
{"status":"success","message":"https:\/\/images.dog.ceo\/breeds\/corgi-cardigan\/n02113186_10475.jpg"},
{"status":"success","message":"https:\/\/images.dog.ceo\/breeds\/boxer\/n02108089_1357.jpg"},
/ #
</pre>

Этот файл можно достать из контейнера для дальнейших экспериментов
<pre>
$ sudo docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                      NAMES
78a72285499e        mongoclient_alpine-client   "/bin/sh"                About an hour ago   Up About an hour                               mongoclient_alpine-client_run_1

$ sudo docker cp 78a72285499e:/data/test.json /home/adzhumurat/test.json
$ ls /home/adzhumurat/test.json
/home/adzhumurat/test.json

</pre>

Загрузим файл в Mongo o с  помощью утилиты mongoimport:

<pre>
/ # /usr/bin/mongoimport --host $APP_MONGO_HOST --port $APP_MONGO_PORT --db pets --collection dogs --file /data/test.json
</pre>

Проверим, что документы успешно загружены
<pre>
> use pets
switched to db pets
> db.stats()
{
	"db" : "pets",
	"collections" : 1,
	"views" : 0,
	"objects" : 100,
	"avgObjSize" : 115.46,
	"dataSize" : 11546,
	"storageSize" : 16384,
	"numExtents" : 0,
	"indexes" : 1,
	"indexSize" : 16384,
	"fsUsedSize" : 43341520896,
	"fsTotalSize" : 244529655808,
	"ok" : 1
}

Посмотрим на записи, которые появились в таблице
<pre>
> use pets
switched to db pets
> db.dogs.find().limit(3)
{ "_id" : ObjectId("5b5aad7b54c17bb03ac25e64"), "status" : "success", "message" : "https://images.dog.ceo/breeds/greyhound-italian/n02091032_9131.jpg" }
{ "_id" : ObjectId("5b5aad7b54c17bb03ac25e65"), "status" : "success", "message" : "https://images.dog.ceo/breeds/hound-afghan/n02088094_1534.jpg" }
{ "_id" : ObjectId("5b5aad7b54c17bb03ac25e66"), "status" : "success", "message" : "https://images.dog.ceo/breeds/chihuahua/n02085620_5093.jpg" }
>
</pre>