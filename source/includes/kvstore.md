# KV Store (Key-Value Storage)

The KV Store is like Users/Game VFS (a game-scoped key-value store you can use to associate data with a game and/or a user) coupled
with ACL rights. Use it to store any game or gamer data you want to make writable and/or readable to a restricted group of users.

Because you might want to share those data between games, the KV Store is scoped by domains.

ACL rights simply define the lists of gamers authorized to perform certain actions related to a specific key. There are 3 types of rights:
- `r` (read): Allows gamers to get key's value and ACL rights
- `w` (write): Allows gamers to set key's value (but not ACL rights!)
- `a` (acl/delete): Allows gamers to change key's ACL rights setup and to delete the key

Each of those ACL rights can take one of the following values:
- `["gamerID1", "gamerID2", ...]`: An array of gamerIDs (all gamers with their gamerID in this array will be authorized for the corresponding
ACL right, the other ones won't)
- `"*"`: A wildcard string (all gamers will be authorized for the corresponding ACL right)

According to all this, an example "ACL setup object" would look like: `{r: "*", w: ["gamerID1", "gamerID2"], a: ["gamerID1"]}`

Meaning:
- Everyone can read the key
- Only gamer1 and gamer2 can write key's value
- Only gamer1 can change key's ACL rights

## Create key data

> To invoke a user batch in order to create a KV Store key, use this code:

```cpp
```

```csharp
using CotcSdk;

public class MyClass
{
    void InvokeCreateKvStoreKeyUserBatch()
    {
        string gamerID1 = "5ca70fbc...";
        string gamerID2 = "5ca70fbc...";

        Bundle kvStoreAcl = Bundle.CreateObject();
        kvStoreAcl["r"] = new Bundle("*");
        kvStoreAcl["w"] = Bundle.CreateArray(new Bundle[] { new Bundle(gamerID1), new Bundle(gamerID2) });
        kvStoreAcl["a"] = Bundle.CreateArray(new Bundle[] { new Bundle(gamerID1) });

        // Or: Bundle kvStoreAcl = Bundle.FromJson("{\"r\":\"*\",\"w\":[\"gamerID1\",\"gamerID2\"],\"a\":[\"gamerID1\"]}");

        Bundle batchParams = Bundle.CreateObject();
        batchParams["keyName"] = new Bundle("KvStoreKeyA");
        batchParams["keyValue"] = new Bundle("KvStoreValueA");
        batchParams["keyAcl"] = kvStoreAcl;

        // currentGamer is an object retrieved after one of the different Login functions.
        currentGamer.Batches.Domain("private").Run("KvStore_CreateKey", batchParams).Done(
            // You may want to check for success with: if (result["n"].AsInt() == 1)
            delegate(Bundle result) { Debug.Log("Success Create Key: " + result.ToString()); },
            delegate(Exception error) { Debug.LogError("Error Create Key: " + error.ToString()); }
        );
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function InvokeCreateKvStoreKeyUserBatch()
{
    const gamerID1 = "5ca70fbc...";
    const gamerID2 = "5ca70fbc...";

    const kvStoreAcl = { r: "*", w: [gamerID1, gamerID2], a: [gamerID1] };
    const batchParams = { keyName: "KvStoreKeyA", keyValue: "KvStoreValueA", keyAcl: kvStoreAcl };

    clan.withGamer(gamer).runBatch("private", "KvStore_CreateKey", batchParams, function(error, result)
    {
        if (error)
		    ConsoleLog("Error Create Key: " + JSON.stringify(error));
        // You may want to check for success with: if (result.n === 1)
	    else
		    ConsoleLog("Success Create Key: " + JSON.stringify(result));
    });
}
```

```http
POST /v1/gamer/batch/private/KvStore_CreateKey
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret: YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "r": "*",
    "w": ["gamerID1","gamerID2"],
    "a": ["gamerID1"]
}
```

> The related sample batch script (Javascript) to create the KV Store key:

```javascript--center
function __KvStore_CreateKey(params, customData, mod) {
    "use strict";
    // don't edit above this line // must be on line 3
    mod.debug("params.request ›› " + JSON.stringify(params.request));

    if (typeof params.request.keyAcl.r === "object") { params.request.keyAcl.r = mod.ObjectIDs(params.request.keyAcl.r); }
    if (typeof params.request.keyAcl.w === "object") { params.request.keyAcl.w = mod.ObjectIDs(params.request.keyAcl.w); }
    if (typeof params.request.keyAcl.a === "object") { params.request.keyAcl.a = mod.ObjectIDs(params.request.keyAcl.a); }

    return this.kv.create(this.game.getPrivateDomain(), params.user_id, params.request.keyName, params.request.keyValue, params.request.keyAcl)
    .then(function(result)
    {
        mod.debug("Success ›› " + JSON.stringify(result));
        return result;
    })
    .catch(function(error)
    {
        mod.debug("Error ›› " + error.name + ": " + error.message);
        throw error;
    });
} // must be on last line, no CR
```

As the KV Store feature is similar to the Game VFS one (as keys are scoped by game, not scoped by gamers), client SDKs are unauthorized
to create keys by themselves. As you can't set key's value before you created the key, you'll have to call for a user batch to create
it first. Moreover, in order to use the KV Store API to create a key from batch's script, you'll have to convert all string gamerIDs
into ObjectIDs.

### User batch call

This function is used to invoke a user batch inside a domain. The batch will receive a JSON with two keys. First
key is `domain` and the second key `request` contains the JSON which was passed as parameter to the function.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to invoke the user batch
batchName | String, required | the name of the user batch to invoke
param | JSON, optional | a JSON holding all the parameters your user batch will need to run

This function returns exactly what your user batch will return, JSON or value.

### KV Store key creation sample batch's expected parameters

Parameter | Type | Description
--------- | ---- | -----------
keyName | String, required | the name of the key to create
keyValue | String, required | the value of the key to create
keyAcl | Object (`r`, `w`, and `a` fields as Strings Array or "*" String), required | the ACL rights setup of the key to create

<aside class="notice">
In the case you're passing strings arrays as ACL right values, those arrays need to be converted into ObjectIDs arrays with
`mod.ObjectIDs(stringsArray)` (like in the sample code) before you can use them as Kv Store key creation API's ACL parameter.
</aside>

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "ok": 1,
    "n": 1,
    "opTime": "6679014173545857027"
}
```

<aside class="notice">
You should check for batch's returned value `n == 1` to ensure the key has been created with success (some fails would result to `n == 0`).
</aside>

## Get key data

> To get key data, use this code:

```cpp
```

```csharp
using CotcSdk;

public class MyClass
{
    void GetKvStoreKeyValue()
    {
        // currentGamer is an object retrieved after one of the different Login functions.
        currentGamer.KvStore.Domain("private").GetValue("KvStoreKeyA")
        .Done(getKeyValueRes => {
            Debug.Log("Key data: " + getKeyValueRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get key data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetKvStoreKeyValue()
{
    clan.withGamer(gamer).kv("private").get("KvStoreKeyA", function(error, getKeyValueRes)
    {
        if (error)
		    ConsoleLog("Get key data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Key data: " + getKeyValueRes);
    });
}
```

```http
GET /v1/gamer/kv/{domain}/{key}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret: YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

Retrieves an individual key from the key-value store if the gamer calling this API is granted `read right` for this key.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to retrieve the key data. By default, if not sent, it will be fetched in the `private` domain
key | String, required | the key to retrieve inside the KV Store. If not present, a 404 error will be returned

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "_id": "5cb0a3a2...",
    "domain": "com.your.game",
    "key": "KvStoreKeyA",
    "value": "ValueA",
    "acl": {
        "r": "*",
        "w": ["5ca70fbc...","5ca70fbc..."],
        "a": ["5ca70fbc..."]
    },
    "cdate": 1555080098222,
    "udate": 1555080098222
}
```

## Set key data

> To set key data, use this code:

```cpp
```

```csharp
using CotcSdk;

public class MyClass
{
    void SetKvStoreKeyValue()
    {
        Bundle value = new Bundle("NewValueA");

        // currentGamer is an object retrieved after one of the different Login functions.
        currentGamer.KvStore.Domain("private").SetValue("KvStoreKeyA", value)
        .Done(setKeyValueRes => {
            Debug.Log("Key data set: " + setKeyValueRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not set key data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function SetKvStoreKeyValue()
{
    clan.withGamer(gamer).kv("private").set("KvStoreKeyA", "NewValueA", function(error, setKeyValueRes)
    {
        if (error)
		    ConsoleLog("Set key data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Key data set: " + getKeyValueRes);
    });
}
```

```http
POST /v1/gamer/kv/{domain}/{key}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret: YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{"value":"NewValueA"}
```

Sets the value of an individual key from the key-value store if the gamer calling this API is granted `write` right for this key.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to set the key data. By default, if not sent, it will be saved in the `private` domain
key | String, required | the key to set inside the KV Store
value | JSON, required | the value to store in the key data

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "ok": 1,
    "nModified": 1,
    "n": 1,
    "lastOp": "6679025873036771329",
    "electionId": "5c959711..."
}
```

<aside class="notice">
You should check for request's returned value `n == 1` to ensure the key has been modified with success (some fails would result to `n == 0`).
</aside>

## Change ACL key data

> To set ACL key data, use this code:

```cpp
```

```csharp
using CotcSdk;

public class MyClass
{
    void SetKvStoreKeyAcl()
    {
        string gamerID3 = "5ca70fbc...";
        string gamerID4 = "5ca70fbc...";

        Bundle kvStoreAcl = Bundle.CreateObject();
        kvStoreAcl["r"] = new Bundle("*");
        kvStoreAcl["w"] = Bundle.CreateArray(new Bundle[] { new Bundle(gamerID3), new Bundle(gamerID4) });
        kvStoreAcl["a"] = Bundle.CreateArray(new Bundle[] { new Bundle(gamerID3) });

        // Or: Bundle kvStoreAcl = Bundle.FromJson("{\"r\":\"*\",\"w\":[\"gamerID3\",\"gamerID4\"],\"a\":[\"gamerID3\"]}");

        // currentGamer is an object retrieved after one of the different Login functions.
        currentGamer.KvStore.Domain("private").ChangeACL("KvStoreKeyA", kvStoreAcl)
        .Done(setKeyAclRes => {
            Debug.Log("Key ACL set: " + setKeyAclRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not set key ACL due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function SetKvStoreKeyAcl()
{
    const gamerID3 = "5ca70fbc...";
    const gamerID4 = "5ca70fbc...";

    const kvStoreAcl = { r: "*", w: [gamerID3, gamerID4], a: [gamerID3] };

    clan.withGamer(gamer).kv("private").acl("KvStoreKeyA", kvStoreAcl, function(error, setKeyAclRes)
    {
        if (error)
		    ConsoleLog("Set key ACL error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Key ACL set: " + setKeyAclRes);
    });
}
```

```http
POST /v1/gamer/kv/{domain}/{key}/acl
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret: YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{"acl":{"r":"*","w":["5ca70fbc...","5ca70fbc..."],"a":["5ca70fbc..."]}}
```

Changes ACL rights setup of an individual key from the key/value store if the gamer calling this API is granted `acl/delete` right for this key.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to set the key ACL. By default, if not sent, it will be saved in the `private` domain
key | String, required | the key to set inside the KV Store
acl | JSON, required | the value to store as the key ACL

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "ok": 1,
    "nModified": 1,
    "n": 1,
    "lastOp": "6679031538098634753",
    "electionId": "5c959711..."
}
```

<aside class="notice">
You should check for request's returned value `n == 1` to ensure the key has been modified with success (some fails would result to `n == 0`).
</aside>

## Delete key data

> To delete key data, use this code:

```cpp
```

```csharp
using CotcSdk;

public class MyClass
{
    void DeleteKvStoreKeyValue()
    {
        // currentGamer is an object retrieved after one of the different Login functions.
        currentGamer.KvStore.Domain("private").DeleteKey("KvStoreKeyA")
        .Done(deleteKeyValueRes => {
            Debug.Log("Key data delete: " + deleteKeyValueRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not delete key data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function DeleteKvStoreKeyValue()
{
    clan.withGamer(gamer).kv("private").del("KvStoreKeyA", function(error, deleteKeyRes)
    {
        if (error)
		    ConsoleLog("Delete key data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Key data delete: " + deleteKeyRes);
    });
}
```

```http
DELETE /v1/gamer/kv/{domain}/{key}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret: YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

Removes an individual key from the key-value store if the gamer calling this API is granted `acl/delete` right for this key.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to delete the key data. By default, if not sent, it will be deleted in the `private` domain
key | String, required | the key to delete inside the KV Store

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "ok": 1,
    "n": 1,
    "lastOp": "6679028565981265921",
    "electionId": "5c959711..."
}
```

<aside class="notice">
You should check for request's returned value `n == 1` to ensure the key has been deleted with success (some fails would result to `n == 0`).
</aside>
