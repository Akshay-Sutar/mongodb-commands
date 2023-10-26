## Open terminal and type
`mongosh` to start mongo

## DB operations
`show dbs` will list all the database

`use shop` will select db shop. It will create a new one if not already present.

`db.dropDatabase()` will drop the current database

`db.books.drop()` will drop the collection books 

## to insert a document in a collection
`db.products.insertOne({ name:"book",price:299})`
this will create `products` collection if not already created and insert a document in it.

to insert many documents at once
`db.products.insertMany([{ name:"book",price:299},{ name:"book",price:399}])`

## to list all documents
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

## to delete one record
`db.flightData.deleteOne({departureAirport:"TXL"})`
this will delete the first document where departureAirport is TXL.

`db.flightData.deleteMany({marker:"toDelete"})`
this will delete all records where marker value is toDelete.

## to update one record
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
this will return documents with only book and price key, _id and all other keys will be excluded.

## updating nested data
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
j is journal. Journal is basically like a todo task list of server which has list of write operation it needs to perform. default is true, which means document will be written in memory, then journal  and then to the disk.

`db.books.insertOne({name:"Jack Reacher"},{writeConcern:{w:1, j: true, wtimeout: 1}})`
wtimeout is write timout 

## importing data
`mongoimport tv-shows.json -d moviesDatabase -c moviesCollection --jsonArray --drop`

will import tv-shows.json data into `moviesDatabase` database and `moviesCollection` collection. `--jsonArray` specifies that it's bulk insert of documents. `--drop` will drop an existing `moviesCollection` collection, create a new `moviesCollection` and insert in it, else data will be appended in `moviesCollection`.

# Read opertions
## filter
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