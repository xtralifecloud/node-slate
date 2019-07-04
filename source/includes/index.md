# Indexing

If you want to search for something, you probably want to use the indexing API. You can use it to index gamers (for match making),
or matches, or anything. It's free-form, so it can apply to every use case. It works in parallel to other APIs. For instance, you
could create a match, then index it to allow searching matches. Or you could index gamer's properties for fast retrieval.So the API
is very simple and quite generic.

**Note:**<br>
The indexing API is an unauthenticated API: you don't have to be logged in as a user to use it.

## Set an index

> To insert or modify an index, use this code:

```cpp
#include "CIndexManager.h"

class MyGame
{
    void SetIndex()
    {
        CotCHelpers::CHJSON index, properties, payload;

        properties.Put("ratioVictory", 50);
        properties.Put("XP", 325);
        properties.Put("level", 12);
        payload.Put("someData", "playerData");
        index.Put("domain", "private");
        index.Put("index", "playerMatchMakingIndex");
        index.Put("objectid", "587f5d844877a1734ec079e6");
        index.Put("properties", properties.Duplicate());
        index.Put("payload", payload.Duplicate());
        CloudBuilder::CIndexManager::Instance()->IndexObject(&index, MakeResultHandler(this, &MyGame::SetIndexDone));
    }

    void SetIndexDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Index set: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not set index due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void SetIndex()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        Bundle properties = Bundle.CreateObject("ratioVictory", 50, "XP", 325, "level", 12);
        Bundle payload = Bundle.CreateObject("someData", "playerData");
        cloud.Index("dummyIndex", "private").IndexObject("587f5d844877a1734ec079e6", properties, payload).Done(setIndexRes => {
            Debug.Log("Index set: " + setIndexRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not set index: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
#import "XLIndex.h"

void SetIndex()
{
    XLIndex* index = [[XLIndex alloc] initWithDomain:@"private" name:@"dummyIndex"];

    NSMutableDictionary* properties = [NSMutableDictionary  dictionaryWithDictionary: @{@"ratioVictory" : @50, @"world" : @325, @"level" : @12 }];
    NSMutableDictionary* payload = [NSMutableDictionary  dictionaryWithDictionary: @{@"someData" : @"playerData"}];
    [index indexObject:payload withProperties:properties withObjectId:@"587f5d844877a1734ec079e6" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *setIndexRes) {
        if(error == nil)
        {
            NSLog(@"Index set: %@", [setIndexRes description]);
        }
        else
        {
            NSLog(@"Could not set index: %@", [error description]);
        }
    }];
}
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`

function SetIndex()
{
    properties = { ratioVictory : 50, XP : 325, level : 12 };
    payload = { someData, "playerData" };
    clan.indexes("private").set("dummyIndex", "587f5d844877a1734ec079e6", properties, payload, function(error, setIndexRes)
    {
      if(error)
		    ConsoleLog("Could not set index: " + JSON.stringify(error));
	    else
		    ConsoleLog("Index set: " + JSON.stringify(setIndexRes));
    });
}
```

```http
POST /v1/index/{domain}/{indexName} HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret

BODY
{
    "id" : "587f5d844877a1734ec079e6",
    "properties" : {
        "ratioVictory" : 50
        "XP" : 325
        "level" : 12
    },
    "payload" : {
        "someData" : "playerData"
    }
}
```

This function is used to add or update an object in an index. You can have as many indexes as you need: one for gamers' properties,
one for matches, one for finished matches... It only depends on what you want to search. The first time you set an index, it will be
a creation (with version 1 and created flag set to true in the result), and next time it will simply update the index and increment
the version number.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to create or update the index
indexName | String, required | used to scope your index (for example `matchIndex`, `userIndex`, ...)
id | String, required | a string holding the index you want to set.
properties | JSON, required | a JSON object whose attributes will be indexed and searchable. These are the "indexed fields" of the object. These properties are typed! So when one property is an int, it must always be an int, or an error will be thrown upon insertion.
payLoad | JSON, required | a JSON object. It's not indexed, but its contents are returned in searches.

This function returns a JSON with the five keys including `_index` (internal to XtraLife), `_type` which is your index name, `_id`
which is your index, `_version` and `created`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "_index": "com.yourcompany.yourgame.yourdomain",
    "_tyoe": "dummyIndex",
    "_id": "587f5d844877a1734ec079e6",
    "_version": 1,
    "created": true,
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "TypeError",
  "message" : "Cannot set property 'payload' of undefined"
}
```

## Get an index

> To insert or modify an index, use this code:

```cpp
#include "CIndexManager.h"

class MyGame
{
    void GetIndex()
    {
        CotCHelpers::CHJSON json;

        json.Put("domain", "private");
        json.Put("index", "dummyIndex");
        json.Put("objectid", "587f5d844877a1734ec079e6");
        CloudBuilder::CIndexManager::Instance()->IndexObject(&json, MakeResultHandler(this, &MyGame::GetIndexDone));
    }

    void GetIndexDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Got index: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not get index due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void GetIndex()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        cloud.Index("dummyIndex", "private").GetObject("587f5d844877a1734ec079e6")
            .Done(getIndexRes => {
                Debug.Log("Got index: " + getIndexRes.ToString());
            }, ex => {
                // The exception should always be CotcException
                CotcException error = (CotcException)ex;
            Debug.LogError("Could not get index: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
            });
    }
}
```

```objective_c
#import "XLIndex.h"

void GetIndex()
{
    XLIndex* index = [[XLIndex alloc] initWithDomain:@"private" name:@"dummyIndex"];

    [index fetchObject:@"587f5d844877a1734ec079e6" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *getIndexRes) {
        if(error == nil)
        {
            NSLog(@"Got index: %@", [getIndexRes description]);
        }
        else
        {
            NSLog(@"Could not set index: %@", [error description]);
        }
    }];
}
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`

function GetIndex()
{
    clan.indexes("private").get("dummyIndex", "587f5d844877a1734ec079e6", function(error, getIndexRes)
    {
      if(error)
		    ConsoleLog("Could not get index: " + JSON.stringify(error));
	    else
		    ConsoleLog("Got index: " + JSON.stringify(getIndexRes));
    });
}
```

```http
GET /v1/index/{domain}/{indexName} HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
```

This function is uded to retrieve a single index entry, by id. Of course, you should use the search API instead if you want to
find more than one element.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to retrieve the index
indexName | String, required | used to scope your index (for example `matchIndex`, `userIndex`, ...)
id | String, required | a string holding the index you want to retrieve.

This function returns a JSON with the following keys: `_index` (internal to XtraLife), `_type` which is your index name, `_id`
which is your index, `_version`, `found` which is equal to `true` and `source` which is a JSON object containing your `properties`
and the `payload`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "_index": "com.yourcompany.yourgame.yourdomain",
  "_type": "dummyIndex",
  "_id": "587f5d844877a1734ec079e6",
  "_version": 2,
  "found": true,
  "_source": {
    "ratioVictory": 50,
    "XP": 325,
    "level": 12,
    "payload": {
      "someData": "playerData"
    }
  }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "NotFound",
  "message" : "Not found"
}
```

## Delete an index

> To delete an index, use this code:

```cpp
#include "CIndexManager.h"

class MyGame
{
    void DeleteIndex()
    {
        CotCHelpers::CHJSON json;

        json.Put("domain", "private");
        json.Put("index", "dummyIndex");
        json.Put("objectid", "587f5d844877a1734ec079e6");
        CloudBuilder::CIndexManager::Instance()->DeleteObject(&json, MakeResultHandler(this, &MyGame::DeleteIndexDone));
    }

    void DeleteIndexDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Index deleted: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not delete index due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void DeleteIndex()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        cloud.Index("dummyIndex", "private").DeleteObject("587f5d844877a1734ec079e6")
        .Done(deleteIndexRes => {
            Debug.Log("Index deleted: " + deleteIndexRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not delete index: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
#import "XLIndex.h"

void DeleteIndex()
{
    XLIndex* index = [[XLIndex alloc] initWithDomain:@"private" name:@"dummyIndex"];

    [index deleteObject:@"587f5d844877a1734ec079e6" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *deleteIndexRes) {
        if(error == nil)
        {
            NSLog(@"Index deleted: %@", [deleteIndexRes description]);
        }
        else
        {
            NSLog(@"Could not delete index: %@", [error description]);
        }
    }];
}
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`

function DeleteIndex()
{
    clan.indexes("private").get("dummyIndex", "587f5d844877a1734ec079e6", function(error, deleteIndexRes)
    {
      if(error)
		    ConsoleLog("Could not delete index: " + JSON.stringify(error));
	    else
		    ConsoleLog("Index deleted: " + JSON.stringify(deleteIndexRes));
    });
}
```

```http
DELETE /v1/index/{domain}/{indexName} HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
```

This function is uded to delete a single index entry, by id.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to delete the index
indexName | String, required | used to scope your index (for example `matchIndex`, `userIndex`, ...)
id | String, required | a string holding the index you want to delete.

This function returns a JSON with the following keys: `found`which is equal to `true`, `_index` (internal to XtraLife),
`_type` which is your index name, `_id` which is your index, and `_version`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "found": true,
  "_index": "com.yourcompany.yourgame.yourdomain",
  "_type": "dummyIndex",
  "_id": "587f5d844877a1734ec079e6",
  "_version": 3,
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "Error",
  "message" : "Not Found"
}
```

## Search an index

> To search an index, use this code:

```cpp
#include "CIndexManager.h"

class MyGame
{
    void SearchIndex()
    {
        CotCHelpers::CHJSON json, *sort;

        sort = CotCHelpers::CHJSON::Array();
        sort->Add(new CotCHelpers::CHJSON("ratioVictory"));
        sort->Add(new CotCHelpers::CHJSON("level"));
        json.Put("domain", "private");
        json.Put("index", "dummyIndex");
        json.Put("query", "XP:325");
        json.Put("sort", sort);
        json.Put("limit", 5);
        json.Put("skip", 0);
        CloudBuilder::CIndexManager::Instance()->Search(&json, MakeResultHandler(this, &MyGame::SearchIndexDone));
    }

    void SearchIndexDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Index search: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not search index due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void SearchIndex()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        List<string> sort = new List<string>;
        sort.Add("ratioVictory");
        sort.Add("level");
        cloud.Index("dummyIndex", "private").Search("XP:325", sort, 5, 0)
        .Done(searchIndexRes => {
            Debug.Log("Index search: " + searchIndexRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not search index: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
#import "XLIndex.h"

void SearchIndex()
{
    XLIndex* index = [[XLIndex alloc] initWithDomain:@"private" name:@"dummyIndex"];

    [index search:@"XP:325" sortBy:[[NSArray alloc] initWithObjects:@"ratioVictory", @"level", nil] withLimit:5 withOffset:0 completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *searchIndexRes) {
        if(error == nil)
        {
            NSLog(@"Index search: %@", [searchIndexRes description]);
        }
        else
        {
            NSLog(@"Could not search index: %@", [error description]);
        }
    }];
}
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`

function SearchIndex()
{
    clan.indexes("private").search("dummyIndex", "XP:325", ["ratioVictory", "level"], 0, 5, function(error, searchIndexRes)
    {
      if(error)
		    ConsoleLog("Could not search index: " + JSON.stringify(error));
	    else
		    ConsoleLog("Index search: " + JSON.stringify(searchIndexRes));
    });
}
```

```http
POST /v1/index/{domain}/{indexName}/search?{query}&{sort}&{from}&{max} HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret

BODY
{
}
```

This function is used to search through indexes. It allows you to make complex queries. See the Elastic documentation to learn the
full syntax for the query parameter. It's easy and quite powerful.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to search through indexes
indexName | String, required | used to scope your index (for example `matchIndex`, `userIndex`, ...)
query | String, required | the query used to search indexes
sort | JSON, optional | a JSON array containing properties to use for sorting results
from | Int, optional | the index of the first result
max | Int, optional | the number of results to return

This function returns a JSON with the following keys: `total` which is the total number of results, `max_score`, as returned by
ElasticSearch and `hits`, a JSON array whose elements are the usual index description as well as a `sort` entry if you have
provided a sort option in your query.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "total": 6,
  "max_score": null,
  "hits": [
    {
      "_index": "com.yourcompany.yourgame.yourdomain",
      "_type": "dummyIndex",
      "_id": "1",
      "_score": null,
      "_source": {
        "ratioVictory": 20,
        "XP": 325,
        "level": 12,
        "payload": {
          "name": "Captain America",
          "lastPlayed": 1433428652428
        }
      },
      "sort": [
        20,
        12
      ]
    },
    {
      "_index": "com.yourcompany.yourgame.yourdomain",
      "_type": "dummyIndex",
      "_id": "2",
      "_score": null,
      "_source": {
        "ratioVictory": 20,
        "XP": 325,
        "level": 15,
        "payload": {
          "name": "Captain America",
          "lastPlayed": 1433428652428
        }
      },
      "sort": [
        20,
        15
      ]
    }
  ]
}
```
