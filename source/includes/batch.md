# Batch

If you need to extend some features for your game which do not exist in the current set of XtraLife features, or if you wish
to secure some parts of the gameplay (in case the client would be hacked), batch feature is a good candidate for you. A batch
is a piece of Javascript code that you upload to our servers and which are available through several ways. The most easy way
to invoke a batch is to call them directly from the client, with a set of parameters.
There are two kinds of batch that can be invoked from the client:

1. Unauthenticated batch: can be called even when no player has logged into the game. The only default parameter passed to the
batch is the domain on which the batch is called
2. Authenticated batch: in order to be invoked, a user has to be logged in the game. The domain parameter is still passed to the
batch, as well as the `gamer_id` which identifies which user is logged in. From there, the batch can have access to the user data

## Game batch

> To invoke a game batch, use this code:

```cpp
#include "CGameManager.h"

class MyGame
{
    void InvokeGameBatch()
    {
        XtraLife::Helpers::CHJSON config, param;

        config.Put("domain", "private");
        config.Put("name", "myGameBatch");
        param.Put("myParam1", "Some value");
        param.Put("myParam2", 1000);
        XtraLife::CGameManager::Instance()->Batch("private", XtraLife::MakeResultHandler(this, &MyGame::InvokeGameBatchHandler);
    }

    void InvokeGameBatchHandler(XtraLife::eErrorCode aErrorCode, const XtraLife::CCloudResult *aResult)
    {
        if(aErrorCode == XtraLife::eErrorCode::enNoErr)
        {
            const XtraLife::Helpers::CHJSON* json = aResult->GetJSON();
            printf("Result of game batch: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not invoke game batch due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void InvokeGameBatch()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        Bundle param = Bundle.CreateObject("myParam1", "Some value", "myParam2", 1000);
        cloud.Game.Batches.Domain("private").Run("myGameBatch", param)
        .Done(invokeGameBatchRes => {
            Debug.Log("Result of game batch: " + invokeGameBatchRes.ToString());
		}, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not invoke game batch: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`

function InvokeGameBatch()
{
    clan.runBatch("private", "myGameBatch", { myParam1 : "Some value", myParam2 : 1000 }, function(error, invokeGameBatchRes)
    {
      if(error)
		    ConsoleLog("Could not invoke game batch: " + JSON.stringify(error));
	    else
		    ConsoleLog("Result of game batch: " + JSON.stringify(invokeGameBatchRes));
    });
}
```

```http
POST /v1/batch/{domain}/{batchName} HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret

BODY
{
    "myParam1" : "Some value"
    "myParam2" : 1000
}
```

```javascript--server
function __InvokeGameBatch(params, customData, mod) {
    "use strict";
    // don't edit above this line // must be on line 3
  
    return this.game.runBatch(this.game.getPrivateDomain(), "myGameBatch", { myParam1 : "Some value", myParam2 : 1000 })
    .then(res => {
        mod.debug("Result of game batch: " + JSON.stringify(res));
        return res;
    })
    .catch(error => {
  	    throw new Error("Could not invoke game batch: " + error);
    });
} // must be on last line, no CR

// A variant of this feature is available in the server side scripts
// which allows to lock a resource before calling the batch.
function __InvokeGameBatchWithLock(params, customData, mod) {
    "use strict";
    // don't edit above this line // must be on line 3
  
    return this.game.runBatchWithLock(this.game.getPrivateDomain(), "myGameBatch", { myParam1 : "Some value", myParam2 : 1000 }, "resourceToLock")
    .then(res => {
        mod.debug("Result of game batch: " + JSON.stringify(res));
        return res;
    })
    .catch(error => {
  	    throw new Error("Could not invoke game batch: " + error);
    });
} // must be on last line, no CR
```

This function is used to invoke a game batch inside a domain. The batch will receive a JSON with two keys. First
key is `domain` and the second key `request` contains the JSON which was passed as parameter to the function.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to invoke the game batch
batchName | String, required | the name of the game batch to invoke
param | JSON, optional | a JSON holding all the parameters your game batch will need to run

This function returns exactly what your game batch will return, JSON or value.

---

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "HookError",
  "message" : "Hook com.mycompany.mygame.domain/__unavailableBatch does not exist",
}
```

## User batch

> To invoke a user batch, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void InvokeUserBatch()
    {
        XtraLife::Helpers::CHJSON config, param;

        config.Put("domain", "private");
        config.Put("name", "myUserBatch");
        param.Put("myParam1", "Some value");
        param.Put("myParam2", 1000);
        XtraLife::CUserManager::Instance()->Batch("private", XtraLife::MakeResultHandler(this, &MyGame::InvokeUserBatchHandler);
    }

    void InvokeUserBatchHandler(XtraLife::eErrorCode aErrorCode, const XtraLife::CCloudResult *aResult)
    {
        if(aErrorCode == XtraLife::eErrorCode::enNoErr)
        {
            const XtraLife::Helpers::CHJSON* json = aResult->GetJSON();
            printf("Result of user batch: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not invoke user batch due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void InvokeUserBatch()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        Bundle param = Bundle.CreateObject("myParam1", "Some value", "myParam2", 1000);
        currentGamer.Batches.Domain("private").Run("myUserBatch", param)
        .Done(invokeUserBatchRes => {
            Debug.Log("Result of user batch: " + invokeUserBatchRes.ToString());
		}, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not invoke user batch: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function InvokeUserBatch()
{
    clan.withGamer(gamer).runBatch("private", "myUserBatch", { myParam1 : "Some value", myParam2 : 1000 }, function(error, invokeUserBatchRes)
    {
      if(error)
		    ConsoleLog("Could not invoke user batch: " + JSON.stringify(error));
	    else
		    ConsoleLog("Result of user batch: " + JSON.stringify(invokeUserBatchRes));
    });
}
```

```http
POST /v1/gamer/batch/{domain}/{batchName} HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "myParam1" : "Some value"
    "myParam2" : 1000
}
```

This function is used to invoke a user batch inside a domain.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to invoke the game batch
batchName | String, required | the name of the game batch to invoke
param | JSON, optional | a JSON holding all the parameters your game batch will need to run

This function is used to invoke a user batch inside a domain. The batch will receive a JSON with three keys. First
key is `domain`, the second key is `request` and contains the JSON which was passed as parameter to the function. The
last key is `gamer_id` and is the identifier of the logged in user.

---

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "HookError",
  "message" : "Hook com.mycompany.mygame.domain/__unavailableBatch does not exist",
}
```
