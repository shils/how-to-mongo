## Mongo Shell

### Pretty print by default

**Versions**: 3.0 - 4.2

Add the following line to your `mongorc.js` to pretty print the results of non-aggregation cursors by default:

```javascript
DBQuery.prototype._prettyShell = true;
```

One would expect that a corresponding setting exists for aggregation cursors, but I'm not sure what it is.

### View authenticated user(s) and their roles

Note that this only returns authentication information for the current shell.

```
> db.runCommand({ connectionStatus: 1 })
{
	"authInfo" : {
		"authenticatedUsers" : [
			{
				"user" : "foo",
				"db" : "admin"
			}
		],
		"authenticatedUserRoles" : [
			{
				"role" : "dbAdminAnyDatabase",
				"db" : "admin"
			}
		]
	},
	"ok" : 1
}
```

### View server configuration settings

```
> db.runCommand({ getCmdLineOpts: 1 })
{
	"argv" : [
		"mongod",
		"-f",
		"/usr/local/etc/mongod.conf"
	],
	"parsed" : {
		"config" : "/usr/local/etc/mongod.conf",
		"net" : {
			"bindIp" : "127.0.0.1"
		},
		"processManagement" : {
			"fork" : true
		},
		"storage" : {
			"dbPath" : "/usr/local/var/mongodb-data"
		},
		"systemLog" : {
			"destination" : "file",
			"logAppend" : true,
			"path" : "/usr/local/var/log/mongodb/mongo.log"
		}
	},
	"ok" : 1
}
```

## Indexes

### Re-create an index with different options

1. Create a new index that 'covers' the index you wish to re-create. A simple way to do this is to create a compound index with the existing index's fields as a prefix followed by a fake field. For example, to cover an index on `{ state: 1, city: 1 }`, you could create an index on `{ state: 1, city: 1, x: 1 }`. 
2. Drop the index that you'll be re-creating. To minimize the performance impact of losing a 'warm' index, you can run a query with the covering index as a [hint](https://docs.mongodb.com/manual/reference/operator/meta/hint/) prior to running `dropIndex`.
3. Re-create the index with the desired options.
4. Drop the index with the fake field that you created in step 1.

## Querying

### Array length greater than/less than

To find all documents where a particular array type field (`arrayField` in this example) has **at least** `3` elements:

```
db.collection.find({ 'arrayField.2': { $exists: true } })
```

Finding all documents where `arrayField` has **at most** 3 elements is similar:

```
db.collection.find({ 'arrayField.3': { $exists: false } })
```



