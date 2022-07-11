# Docker Compose MongoDB

Install MongoDB with Docker Compose

## Run project

```
docker-compose up -d
```

MongoDB will run with port `27018`.

You can change the port however you want. Ex: 27019,..etc..

## Stop project

Without remove the container.

```
docker-compose stop
```

Remove the container.

```
docker-compose down
```

## MongoDB credential (for database admin)

- Username: root
- Password: rootpassword

## How to connect to MongoDB

Via mongo Shell

```
mongo admin -u root -p root123456
```

# Các truy vấn cơ bản trong MongoDB

### 1. Mongo Shell

`Mongo Shell` là một JavaScript interface tương tác với MongoDB. Bạn có thể sử dụng mongo shell để `truy vấn` và `cập nhật dữ liệu` cũng như thực hiện các `hoạt động quản trị`.

Thực một số lệnh truy vấn cơ bản trong mongo shell. Đầu tiên, tôi sẽ sử dụng lệnh `mongoimport --help` để xem tất cả documents trong mongo shell.
```
admin@TuanNM MINGW64 /d/Drive/Documens
$ mongoimport --help
Usage:
  mongoimport <options> <file>

Import CSV, TSV or JSON data into MongoDB. If no file is provided, mongoimport reads from stdin.

See http://docs.mongodb.org/manual/reference/program/mongoimport/ for more information.
```

Tôi sẽ thực hiện một số thao tác `CRUD` trong trình Mongo Shell. Để làm như vậy, tôi có thể sử dụng các lệnh import trong Mongo. Các lệnh import Mongo có thể thực hiện công việc import dữ liệu nằm trong file `.tsv, .csv, .json`, v.v.

### 2. Insert

Cách 1: Tạo một collections có tên `Numbers` và insert 26 000 row vào đó.

    > show dbs
    admin       0.000GB
    config      0.000GB
    local       0.000GB
    mongotest   0.001GB
    mylearning  0.000GB

    > use mongo test
    switched to db mongotest

    > for (i = 0; i <= 26000; i++) {
        db.Numbers.insert({
            "number": i
        })
    }
    WriteResult({ "nInserted" : 1 })


Import data từ file `JSON`. Tạo một file json với tên `colors.json` có cấu trúc như sau:

    [
        {
            "color":"black",
            "category":"hue",
            "type":"primary",
            "code":{
                "rgba":[
                    255,
                    255,
                    255,
                    1
                ],
                "hex":"#000"
            }
        },
        {
            "color":"white",
            "category":"value",
            "code":{
                "rgba":[
                    0,
                    0,
                    0,
                    1
                ],
                "hex":"#FFF"
            }
        },
        {
            "color":"red",
            "category":"hue",
            "type":"primary",
            "code":{
                "rgba":[
                    255,
                    0,
                    0,
                    1
                ],
                "hex":"#FF0"
            }
        },
        {
            "color":"blue",
            "category":"hue",
            "type":"primary",
            "code":{
                "rgba":[
                    0,
                    0,
                    255,
                    1
                ],
                "hex":"#00F"
            }
        },
        {
            "color":"yellow",
            "category":"hue",
            "type":"primary",
            "code":{
                "rgba":[
                    255,
                    255,
                    0,
                    1
                ],
                "hex":"#FF0"
            }
        },
        {
            "color":"green",
            "category":"hue",
            "type":"secondary",
            "code":{
                "rgba":[
                    0,
                    255,
                    0,
                    1
                ],
                "hex":"#0F0"
            }
        }
    ]

Sau đó thực hiện `import` nó vào mongo. Open terminal tại nơi chứa file `colors.json`

    $ mongoimport --db mongo_color --collection colors --jsonArray --file colors.json

Result:

    2019-11-26T11:12:38.063+0700    connected to: mongodb://localhost/
    2019-11-26T11:12:38.082+0700    6 document(s) imported successfully. 0 document(s) failed to import.

- `--db mongo_color`: tên database
- `--collection colors`: tên collection
- `--jsonArray`: nguồn đầu vào là một mảng JSON
- `--file colors.json`: tên file data

Hoặc cách insert từ mongo shell

    $ mongo
    > use mongo_color
    switched to db mongo_color
    > db.colors.insert({
        "color": "custome",
        "category": "hue",
        "type": "primary",
        "code": {
            "rgba": [255, 1, 255, 1],
            "hex": "#FF1"
        }
    })

# 3. Update

Update một row trong mongoDB thì như thế nào?

Tôi sẽ thêm một property mới `manuallyCreated` với value là `true` vào color `custome` đã insert ở trên.

```
> db.colors.update({
    "color": "custome"
}
, {
    $set: {
        "manuallyCreated": "True"
    }
})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

Truy vấn các color có `manuallyCreated` là `true`

    > db.colors.find({"manuallyCreated":"True"})

    { "_id" : ObjectId("5ddca6b6993d1592c2ace363"), "color" : "custome", "category" : "hue", "type" : "primary", "code" : { "rgba" : [ 255, 1, 255, 1 ], "hex" : "#FF1" }, "manuallyCreated" : "True" }


Update color name `custome` thành `blackpink`, điều kiện là một là `ObjectId`:

    > db.colors.update({
        "_id": ObjectId("5ddca6b6993d1592c2ace363")
    }, {
        $set: {
            "color": "blackpink"
        }
    })

    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

    db.colors.find({"manuallyCreated":"True"})

    { "_id" : ObjectId("5ddca6b6993d1592c2ace363"), "color" : "blackpink", "category" : "hue", "type" : "primary", "code" : { "rgba" : [ 255, 1, 255, 1 ], "hex" : "#FF1" }, "manuallyCreated" : "True" }

Trường hợp không set một value cho property cụ thể, thì nó bị ghi đè:

    db.colors.update({"color":"red"},{"color":"test"})
    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

    db.colors.find({"color":"test"})
    { "_id" : ObjectId("5ddca636c0ca555ba6529b59"), "color" : "test" }

Một số ví dụ về update user collections:

    db.users.update(
        {
            age: {
                $gt: 25
            }
        },
        {
            $set: {
                status: "Active"
            }
        },
        {
            multi: true
        }
    )

    db.users.update(
        {
            status: "Active"
        },
        {
            $inc: {
                age: 3
            }
        },
        {
            multi: true
        }
    )

- `multi: true`: nếu có dữ liệu phù hợp điều kiện, thì sẽ update tất cả thay vì một
- `$gt`: greater than
- `$lt`: less than
- `$inc`: ví dụ age = age + 3 để tăng giá trị của một field được chỉ định.

# 4. Truy vấn `select` trong MongoDB

## 1. Find() & Pretty format trong mongo shell:

```
> db.COLLECTION_NAME.find()
```

Example:
```
db.colors.find()
```

Pretty formating:

    db.colors.find().pretty()
    {
            "_id" : ObjectId("5ddcd6a60bd1004f4f9e83d5"),
            "color" : "blue",
            "category" : "hue",
            "type" : "primary",
            "code" : {
                    "rgba" : [
                            0,
                            0,
                            255,
                            1
                    ],
                    "hex" : "#00F"
            }
    }


## 2. Toán tử

Xem docs - https://docs.mongodb.com/manual/reference/operator/query/

| Operation | Syntax | Example | SQL |
| --- | --- | --- | --- |
| Equality | `{<key>:<value>}` | `{status: "Active"}` | `where kind = 'rat'` |
| Less Than | `{<key>:{$lt:<value>}}` | `{age: {$lt: 2}}` | `where age < 2` |
| Less Than Equals | `{<key>:{$lte:<value>}}` | `{age: {$lte: 2}}` | `where age <= 2` |
| Greater Than | `{<key>:{$gt:<value>}}` | `{age: {$gt: 2}}` | `where age > 2` |
| Greater Than Equals | `{<key>:{$gte:<value>}}` | `{age: {$gte: 2}}` | `where age >= 2` |
| Not Equals | `{<key>:{$ne:<value>}}` | `{age: {$ne: 2}}` | `where age != 2` |
| In | `{<key>:{$in:[<value1>, <value2>, ...]}}` | `{age: {$in: [1, 2, 3]}}` | `where age in (1, 2, 3)` |

## 3. Columns

```
db.COLLECTION_NAME.find(QUERY, COLUMNS)
```
Example:
```
db.users.find(
    {
        status: "A"
    },
    {
        user_id: 1,
        status: 1,
        _id: 0
    }
)
```

## 4. AND

```
db.COLLECTION_NAME.find({key1:value1, key2:value2})
```

Example:
```
db.users.find({
    age: {
        $gte: 25
    },
    status: 'Active'
})
```

## 4. OR

```
db.COLLECTION_NAME.find({$or: [{key1: value1}, {key2:value2}]})
```

Example:
```
db.users.find({
    $or: [{
        status: 'Active'
    }, {
        status: 'UnActive'
    }]
})
```
## 5. Limit & Offset

Limit
```
db.users.find().limit(1)

```

Limit và offset
```
db.pets.find().skip(1).limit(1)
```

## 6. Sort

Ascending Sort
```
db.users.find({
    status: "Active"
}).sort({
    name: 1
})
```

Descending Sort
```
db.users.find({
    status: "Active"
}).sort({
    name: -1
})
```

# Làm thế nào để backup và restore database trong MongoDB.

## Backup

Tạo một folder để chứa file backup.

- Command backup tất cả database:

    ```
    $ mongodump --out D:\data\mongodb_backup
    2019-11-26T17:18:38.604+0700    writing admin.system.users to
    2019-11-26T17:18:38.612+0700    done dumping admin.system.users (1 document)
    2019-11-26T17:18:38.612+0700    writing admin.system.version to
    2019-11-26T17:18:38.617+0700    done dumping admin.system.version (2 documents)
    2019-11-26T17:18:38.618+0700    writing mongotest.Numbers to
    2019-11-26T17:18:38.618+0700    writing mongo_color.colors to
    2019-11-26T17:18:38.618+0700    writing mongo_color.users to
    2019-11-26T17:18:38.618+0700    writing mongotest.users to
    2019-11-26T17:18:38.624+0700    done dumping mongotest.users (1 document)
    2019-11-26T17:18:38.925+0700    done dumping mongo_color.users (3 documents)
    2019-11-26T17:18:38.925+0700    done dumping mongo_color.colors (6 documents)
    2019-11-26T17:18:39.163+0700    done dumping mongotest.Numbers (52002 documents)
    ```

- Command backup với một database cụ thể:

    ```
    $ mongodump --db mongo_color --out d:\data\mongodb_backup
    2019-11-26T17:11:45.650+0700    writing mongo_color.colors to
    2019-11-26T17:11:45.652+0700    writing mongo_color.users to
    2019-11-26T17:11:45.659+0700    done dumping mongo_color.users (3 documents)
    2019-11-26T17:11:45.964+0700    done dumping mongo_color.colors (6 documents)
    ```

## Restore

Lệnh restore một database đã được backup:
```
mongorestore --db mongo_color d:\data\mongodb_backup\mongo_color
```

# End

------------------
## tuannguyen29
By default mongodb has no enabled access control, so there is no default user or password.

To enable access control, use either the command line option --auth or security.authorization configuration file setting.

You can use the following procedure or refer to Enabling Auth in the MongoDB docs.

Procedure
1. Start MongoDB without access control.

    ```
    mongod --port 27017 --dbpath /data/db1
    ```

2. Connect to the instance.

    mongo --port 27017

3. Create the user administrator.

    ```
    use admin
    db.createUser(
        {
            user: "tuannguyen",
            pwd: "Root@123",
            roles: [{
                role: "userAdminAnyDatabase",
                db: "admin"
            }]
        }
    )
    ```

4. Re-start the MongoDB instance with access control.

    mongod --auth --port 27017 --dbpath /data/db1

Authenticate as the user administrator.

    mongo --port 27017 -u "tuannguyen" -p "Roo@123" \
    --authenticationDatabase "admin"

https://docs.mongodb.com/php-library/current/tutorial/install-php-library/
