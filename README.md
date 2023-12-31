## Starting mongo server
open terminal and type `mongosh` to start mongo

## DB operations

`show dbs` will list all the database

`use shop` will select db shop. It will create a new one if not already present.

`db.dropDatabase()` will drop the current database

`db.books.drop()` will drop the collection books

`db.dropDatabase()` will drop the database.

## To insert a document in a collection

`db.products.insertOne({ name:"book",price:299})`
this will create `products` collection if not already created and insert a document in it.

to insert many documents at once
`db.products.insertMany([{ name:"book",price:299},{ name:"book",price:399}])`

## To list all documents

`db.products.find()`
will list all the documents present in `products`

```
[
  {
    _id: ObjectId("652ff30c0122a885f7d751e8"),
    name: 'book',
    price: 299
  }
]
```

## To delete one record

`db.flightData.deleteOne({departureAirport:"TXL"})`
this will delete the first document where departureAirport is TXL.

`db.flightData.deleteMany({marker:"toDelete"})`
this will delete all records where marker value is toDelete.

## To update one record

`dv.flightData.updateOne({distance:12000},{$set:{marker:"delete"}})`
mongdb will update the value of `marker` to `"delete"`. If `marker` field is not present, it will create a new one.

`db.flightData.updateMany({},{$set:{marker:"toDelete"}})`
To update many records. Empty `{}` means no filter was passes, and every document will be updated.

## Using condition in filter

`db.books.find({ price : { $gt : 200 }})`
will fetch books where price is greater than 200

`db.books.find({ tags : :"fiction" })`
will give boooks where `tags` have `fiction` as element in it. `tags` is an array of string.

Lets say we have a nested document

```
{
  book:"Harry Potter",
  price:200,
  tags:["Fantasy","Fiction"],
  condition:{
    status: "good"
  }
}
```

to filter by nested field condition status
`db.books.find({"condition.status:"good"})`

## Projecting data

` db.books.find({},{_id:0, book:1, price:1})`
this will return documents with only book and price key, \_id and all other keys will be excluded.

## Updating nested data

`db.books.updateOne({book:"Java"},{$set:{tags:["Computer Science", "Technology"]}})`
this will add a key `tags` which is an array of string

## Accesing the structured data

`db.books.findOne({book:"Harry Potter"}).price`
will give price of Harry potter book price.

## lookup - way of joining document

Let's say there are 2 collections

#### Products collection

```{
  "_id": {
    "$oid": "6534c048163f6837a491dc35"
  },
  "itemName": "Harry Potter",
  "price": 299,
  "category": "books"
}
```

#### Orders collection

```{
  "_id": {
    "$oid": "6534c1d390b8ceeff319b31b"
  },
  "userName": "John",
  "orderTotal": 299,
  "products": [
    {
      "$oid": "6534c048163f6837a491dc35"
    },
    {
      "$oid": "6534c053163f6837a491dc37"
    }
  ]
}
```

to get orders data with products, orders data at the same time, use $lookup

```
db.orders.aggregate([
    {$lookup:{
        from:"products",
        localField:"products",
        foreignField:"_id",
        as:"purchased_products"
    }}
])
```

will return

```
[
  {
    _id: ObjectId("6534c1d390b8ceeff319b31b"),
    userName: 'John',
    orderTotal: 299,
    products: [
      ObjectId("6534c048163f6837a491dc35"),
      ObjectId("6534c053163f6837a491dc37")
    ],
    purchased_products: [
      {
        _id: ObjectId("6534c048163f6837a491dc35"),
        itemName: 'Harry Potter',
        price: 299,
        category: 'books'
      },
      {
        _id: ObjectId("6534c053163f6837a491dc37"),
        itemName: 'RTC 4090',
        price: 1299,
        category: 'GPU'
      }
    ]
  }
]
```

## Schema validation in mongodb

#### validationLevel

which documents get validated?
`strict`
All inserts and updates are validated
`moderate`
All inserts are validated. Updates are only checked for documents which were valid before.

### validationAction

what happens if validation fails?
`error`
Throws an error and deny insert/update.
`warn`
Log warning but allows insert/update

## Adding validation

```
db.createCollection('posts', {
  validator: {
    $jsonSchema: {
      bsonType: 'object', // document is an object
      required: ['title', 'text', 'creator', 'comments'], // mandatory props
      // describing the mandatory props
      properties: {
        title: {
          bsonType: 'string',
          description: 'must be a string and is required',
        },
        text: {
          bsonType: 'string',
          description: 'must be a string and is required',
        },
        creator: {
          bsonType: 'objectId',
          description: 'must be an objectId and is required',
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['text', 'author'],
            properties: {
              text: {
                bsonType: 'string',
                description: 'must be a string and is required',
              },
              author: {
                bsonType: 'objectId',
                description: 'must be an objectId and is required',
              },
            },
          },
        },
      },
    },
  },
});
```

## Adding validation actions

If the collection is already created, db admin will run the following cmd

```
db.runCommand({
  collMod: 'posts', // collection modifier
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['title', 'text', 'creator', 'comments'],
      properties: {
        title: {
          bsonType: 'string',
          description: 'must be a string and is required',
        },
        text: {
          bsonType: 'string',
          description: 'must be a string and is required',
        },
        creator: {
          bsonType: 'objectId',
          description: 'must be an objectId and is required',
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['text', 'author'],
            properties: {
              text: {
                bsonType: 'string',
                description: 'must be a string and is required',
              },
              author: {
                bsonType: 'objectId',
                description: 'must be an objectId and is required',
              },
            },
          },
        },
      },
    },
  },
  validationAction: 'warn',
});
```

## bulk insert

when inserting in bulk with insertMany, the documents are inserted one by one. If one of the insert operation fails, the remaining documents are not inserted.

```
db.books.insertMany([{book:"Harry Potter",{book:"Jack Reacher",{book:"Goosebumps"}}}])
```

so if insert of `Jack Reacher` fails, `Goosebumps` will not be inserted, but `Harry Potter` insert will persist.

This is the default behaviour called `ordered insert`. To turn it off, pass it in config

```
db.books.insertMany(
  [{book:"Harry Potter",{book:"Jack Reacher",{book:"Goosebumps"}}}],
  {ordered: false}
  )
```

This will make sure all the documents without any error will be inserted.

## writeConcern

This deals with how the storage engine stores the data
`db.books.insertOne({name:"Harry Potter"},{writeConcern:{w:1}})`

`w:1` means storage engine waits for acknowledgement from 1 server. if `w:0`, the storage engine returns response immediately without waiting for acknowledgement from server. default is 1.

`db.books.insertOne({name:"Jack Reacher"},{writeConcern:{w:1, j: false}})`
j is journal. Journal is basically like a todo task list of server which has list of write operation it needs to perform. default is true, which means document will be written in memory, then journal and then to the disk.

`db.books.insertOne({name:"Jack Reacher"},{writeConcern:{w:1, j: true, wtimeout: 1}})`
wtimeout is write timout

## importing data

`mongoimport tv-shows.json -d moviesDatabase -c moviesCollection --jsonArray --drop`

will import tv-shows.json data into `moviesDatabase` database and `moviesCollection` collection. `--jsonArray` specifies that it's bulk insert of documents. `--drop` will drop an existing `moviesCollection` collection, create a new `moviesCollection` and insert in it, else data will be appended in `moviesCollection`.

## Read operations

### filter

`find()` and `findOne()`
`findOne()` returns 1 document.`find()` returns cursor with multiple documents.

`db.series.find({name:"Reacher"})`
will return all documents where name is exactly `Reacher`.

### operators

`db.series.find({runtime:{$lt:90}})`
`$lt` will return where runtime is less than 90.

1. `eq`/`ne` - equals to/ not equals to
2. `gt` `gte` - greater than/ greater than or equals to
3. `lt` `lte`- lesser than / lesser than or equals to
4. `in` `nin`- matches value that is in an array / matches value that is not in an array
5. `exists` - check if key exists in a document
6. `type` - specifies the type of the key
7. `regex` - search by regex
8. `expr` - compare key with another key in a document

`db.series.find({"rating.average":{$gte:7}})`
will fetch all shows where rating is >= 7. Rating is a nested documented as

```
{
  ...
  rating:{
    avarage: 8.0
  }
}
```

`db.series.find({"genres":"Thriller"})`
will fetch all shows where `genres` is an array and `Thriller` is an item in it.

`db.series.find({"genres":["Thriller"]})`

`db.series.find({"genres": {$in: ["Thriller","Action"]}})`
will fetch all shows where `genres` is an array and has `Thriller` or `Action` in it.

`db.series.find({$or:[ {"rating.average" : {$gt: 9}} , {"genres":{$in:["Thriller","Action"]}} ]})`
fetch all shows where rating is above 9 or genre is in Thriller,Action

`db.series.find({$and:[ {"rating.average" : {$gt: 9}} , {"genres":{$in:["Thriller","Action"]}} ]})`

fetch all shows where rating is above 9 AND genre is in Thriller,Action

`db.users.find({email:{$exists: true}})`
fetch users where key email exists in the document.

`db.users.find({email:{$exists: true, $ne:null}})`
fetch users where key email exists in the document and is not null.

`db.users.find({email:{$exists: true, $type: "string"}})`
fetch users where key email exists in the document and is a string.

`db.series.find({summary: {$regex: /musical/ }})`

Lets say sales collection is as follows

```
[
  {
    volume: 200,
    target: 150
  },
  {
    volume: 200,
    target: 250
  }
]
```

To get all documents where volume > target, use `expr`
`db.sales.find({ $expr : { $gt: ["$volume","$target"] } })`

### Querying Arrays

```
[{
  ...
  genres:[
    {
      title: "Thriller"
    },
    {
      title: "Drama"
    }
  ]
}]
```

to get all documents where genre is Thriller
`db.movies.find( { "genres.title":"Thriller" } )`

`$size` - array size
`db.series.find({genres:{$size: 3}})`
will fetch all documents where genres has 3 elements

`$all` - exact match on array items
`db.boxOffice.find({genres : {$all:["action","thriller"]}})`
will fetch documents where genres have exactly `action` and `thriller` in it, irrespective of order in the array.

`$elemMatch`

```
[
  ...
  hobbies: [
    {
      title:"Sketching",
      frequency: 3
    },
    {
      title:"Reading",
      frequency: 7
    }
  ]
]
```

to get document where title is `Sketching` and `frequency` is >= 3
`db.users.find( {hobbies : { $elemMatch : {title: "Sketching", frequency : {$gte: 3}} }} )`

## Cursor

`find()` returns a cursor that can be used to read document one by one

```
const dbCursor = db.series.find()
dbCursor.next()
```

will fetch document whenever `next()` is used. To iterate over the cursor
`dbCursor.forEach(doc => printJSON(doc))`

`dbCursor.hasNext()` returns true if cursor has more document, else false.

### sorting data

```
[
  {
    ...
    runtime:30,
    rating:{
      average: 7.8
    }
  }
]
```

`db.movies.find().sort({"rating.average":1, runtime: -1 })`
specify 1 to sort in ascending order, -1 to sort in descending order.

### limiting data

`db.movies.find().skip(10).limit(10)`

will skip the first 10 docuements and fetch only 10 documents

### projection of data

`db.series.find({},{name:1, language:1, genres:1, rating:1, "schedule.time":1})`
will fetch properties specified with 1. `_id` will be included by default , unless explicitly set as 0.

When working with array,
`db.series.find({genres:"Drama"},{"genres.$":1})`

will return all the documents where `genres` array has `Drama` as an item, and will project `genres` with only `Drama` in it.

```
  { _id: ObjectId("6538fa32d412b575f36d313d"), genres: [ 'Drama' ] },
  { _id: ObjectId("6538fa32d412b575f36d313e"), genres: [ 'Drama' ] },
```

`$slice` - returns specified no of items in an array
`db.series.find({},{name:1, genres:{$slice:2}})`

will return data with `genres` having only first 2 elements

`db.series.find({},{name:1, genres:{$slice:[1,2]}})`
will skip the first item and get the next 2 items.

## Update documents

```
db.users.updateOne( {_id: ObjectId("123wdqx")}, {
  $set:{
    hobbies: [
      {title:"Sports", frequency:5},
      {title:"Cooking", frequency:7},
      ]
    }
})
```

will find a document with `id` 123wdqx and set the hobbies field to specified value.

`updateMany()` will update all the documents that matches the filter.

### updating many fields

```
db.users.updateOne({_id:ObjectId("213qad")},{
  $set: {
    age: 40, phone: 123456798
  }
})
```

### update value based on previous value

```
db.users.updateOne(
  {name:"ABC"},
  {
    $inc: {age: 1},
    $set: {phone: "123-456-789"}
  }
)
```

will increment `age` by 1 and set `phone` to specified value.

### $min, $max and $mul

`db.users.updateMany({name:"John"},{$min:{age: 32}})`

will find the users where name is `John` and set `age` to 32 iff `age` of user is greater than 32. i.e specified value must be less than current value. 32 is specified value here. So `age` of user will only get updated when the current `age` is greater than 32 here.

`db.users.updateMany({name:"John"},{$min:{age: 35}})`
will update the age to 35 only iff specified value is greater than 35.

`db.users.updateMany({name:"John"},{$min:{age: 1.1}})`
set user `age` by multiplying it by 1.1

### $unset - dropping fields from document

`db.users.updateMany({isSporty: true},{$unset:{phone:""}})`

this will remove `phone` field from document for users where `isSporty` is true.

### $rename - renaming field

`db.users.updateMany({},{$rename:{age: "totalAge"}})`

will rename `age` field to `totalAge` for all users.

### upsert

if no document is found during update operation, mongodb can create a new document is `upsert` flag is enabled.
`db.updateOne({name:"John"},{ $set:{age: 25, hobbies:[{title:"reading", frequency: 7}]}},{upsert:true})`

we are passing `upsert` as true, which will make mongodb create a new document if no document with `name`="John" was found.

### updating matched array elements

update document where a person has a hobby of sports with frequency >=3

```
db.users.updateMany({hobbies:{$elemMatch:{ title : "sports". frequency:{$gte:3} }}},
{
  $set:{ "hobbies.$.highFrequency": true }
})
```

this will find the document where `hobbies` has an item where `title` is sports and `frequency`>=3, and for that matched element of `hobbies`, add a new field `highFrequency` with value true;

### $[] - update all elements of an array

`db.users.updateMany({age:{$gte: 25}}, { $inc: {"hobbies.$[].frequency": -1} })`
will decrement the `frequency` of every item in `hobbies` array.

### $push - add element to array

`db.users.updateOne({name:"John"}, {$push:{hobbies: {title:"cooking", frequency:7}}})`
will push `{title:"cooking", frequency:7}` into `hobbies` array

### $addToSet - add element to array only if unique

`$addToSet` is similar to `$push`, but will not add element if it already exists.

### $pull - remove element to array

`db.users.updateOne({name:"John"},{$pull:{hobbies:{title:"Hiking"}}})`
will remove item from `hobbies` where `title` was `Hiking`

### $pop - remove array element

`db.users.updateOne({name:"John"},{$pop:{hobbies: 1}})`

will remove last element from `hobbies`. If `hobbies: -1`, will remove first element from array `hobbies`

## Delete operations

### delete 1 collection
`db.users.deleteOne({name:"Manual"})`
will delete 1 record where `name` is `Manual`.

### delete many collections
`db.users.deleteMany({age:{$gt : 25}})`
will delete every records where age > 25.

### truncate collection'
`db.users.deleteMany({})`

### drop collection
`db.users.drop()`



## index

### create index

`db.contacts.createIndex({"dob.age": 1})`
creates index on `dob.age`. 1 means indexes are sorted in ASC order, -1 means indexes are sorted in DESC order.

### compound index

`db.contacts.createIndex({"dob.age": 1, gender:1})`

### text index

If a text index is added on a string 'This is a must have book for science fiction fans.', mongodb will store and create indexes on the words seperately, so it's easy to search by keyword. Index will be added like `must`,`have`,`book`,`science`,`fiction`,`fans`

`db.books.createIndex({description:"text"})`

### drop index

`db.contacts.dropIndex({"dob.age":1})`

### get indexes on a document

`db.contacts.getIndexes()`
