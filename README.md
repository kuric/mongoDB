# mongoDB
MongoDB
1. **Накатить бэкап базы** 
	-  mongoimport --db sampleDB ./users.json

2. **Найти средний возраст людей в системе**
``var avg = db.users.aggregate({ "$group": { _id: null, avg: { "$avg": "$age" }}})``` -- результат { "_id" : null, "avg" : 30.38862559241706 }  
avg = avg.map(function(ret) {return ret.avg})``

3.  **Найти средний возраст в штате Аляска**
  ``var avg_alaska = db.users.aggregate([{$match : { "address" : {"$regex": ".*Alaska.*"}} },{$group : { _id:null, avg_alaska: {"$avg":"$age" }}}])`` -- результат { "_id" : null, "avg_alaska" : 31.5 }
   `` avg_alaska = avg_alaska.map(function(ret) { return ret.avg_alaska} )``
   `` var avg_ceil = Math.ceil(+avg+ +avg_alaska)`` - результат 62
   
4. **Начиная от Math.ceil(avg + avg_alaska) найти первого человека с другом по имени Деннис**
``var dennis_friend = db.users.findOne({$and: [{"index" : { $gt : avg_ceil}}, {"friends.name": /Dennis/}]})`` -- 
``var state = dennis_friend.address.split(',')[2]`` - штат Utah

5. **Найти людей из того же штата, что и предыдущий человек и посмотреть какой фрукт любят больше всего в этом штате (аггрегация)**
``var favoriteFruit = db.users.aggregate({$match: {"address": {$regex: state}}},{$group: {_id:"$favoriteFruit", count : {$sum : 1}}},{$sort : {"count":-1}})``
``var favoriteFruit = favoriteFruit.map(function(fruit) { return fruit._id})``
``favouriteFruit = favoriteFruit[0]`` - результат apple

 6. **Найти саммого раннего зарегистрировавшегося пользователя с таким любимым фруктом**
``var registered = db.users.find({"favoriteFruit": favouriteFruit});``
или ``var registered  = db.users.aggregate({$match: {"favoriteFruit": {$eq: favouriteFruit}}},{$project: {"registered": 1,"favoriteFruit":1}})``
``var regObj = registered.map(function(user) { return {"id":user._id,"reg_date":Date.parse(user.registered.replace(" ",""))}})`` - массив с датами и id
``var regSort = regObj .sort(function(a,b) { return a.reg_date - b.reg_date})`` - сортируем по возрастанию
``var firstObj = regSort [0].id;``- id первого зарегистрировашегося

7. **Добавить этому пользовелю свойтво: { features: 'first apple eater' }**
``db.users.update({_id:firstObj},{$set: {features: 'first apple eater'}})``

8. **Удалить всех любителей клубники (написать количество удаленных пользователей)**
``db.users.remove({favoriteFruit:"strawberry"})`` - результат WriteResult({ "nRemoved" : 253 })
