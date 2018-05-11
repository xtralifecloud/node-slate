# Game VFS (Virtual File System)

Game VFS is a key-value store you can use to associate data with a game. Use it to store levels, boards, textures, music...
Any data you would like your game to share among players. You can either store text through Key/Value (each game has its
dedicated JSON with no predefined keys), or you can store binary objects. In the last case, we'll simply store the binary
object in AWS S3, and we'll fill the value with the AWS S3 url. You won't have to make complex operations since our SDKs handle
that transparently for you (except in Javascript).

Because you might want to share Game VFS data between games you (or other developers) wrote, Game VFS is scoped by domains.

<aside class="notice">
For security reasons, there is no possibility to store or upload data from the client SDKs. If the client was hacked, this would
be too easy to completely ruin the game. Hence, you have two possibilities to store or modify GameVFS:<br/>

1. Make your changes directly from the Backoffice dashboard, by editing keys manually or by uploading files with the corresponding option.<br/>
2. You can also use the Backoffice APIs, which contrary to client APIs and SDKs, give you full access to all data (games, users, ...) in both reading and writing mode. You can couple it with Google Drive for example to automate the import of data from your game designers.<br/>

</aside>


## Get game data

> To get game data, use this code:

```cpp
#include "CGameManager.h"

class MyGame
{
    void GetGameValue()
    {
        CHJSON json;
        json.Put("key", "myKey");
        json.Put("domain", "private");
        CloudBuilder::CGameManager::Instance()->GetValue(&json, MakeResultHandler(this, &MyGame::GetGameValueHandler));
    }

    void GetGameValueHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* result = aResult->GetJSON()->Get("result");
            printf("Game data: %s\n", result->print_formatted().c_str());
        }
        else
            printf("Could not get game data due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void GetGameValue()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        cloud.Game.GameVfs.Domain("private").GetValue("myKey")
        .Done(getGameValueRes => {
            Bundle result = getGameValueRes["result"];
            Debug.Log("Game data: " + result.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get game data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLGameVFS.h"

void GetGameValue()
{
    XLGameVFS* vfs = [[XLGameVFS alloc] initWithDomain:@"private"];
    [vfs getValue:@"myKey" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *getGameValueRes) {
        if(error == nil)
        {
            NSLog(@"Game data: %@", [getGameValueRes description]);
        }
        else
        {
            NSLog(@"Could not get game data: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`

function GetGameValue()
{
    clan.vfs("private").getValue("myKey", function(error, getGameValueRes)
    {
      if(error)
		    ConsoleLog("Get game data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Game data: " + getGameValueRes.result);
    });
}
```

```http
GET /v3.0/vfs/{domain}/{key}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
```

This function is used to retrieve some game data stored in the Game VFS. If you want to retrieve a binary object, you should use
method GetBinary instead because otherwise you will only have the url to the binary object, not the binary object itself.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to retrieve the game data. By default, if not sent, it will be fetched in the `private` domain
key | String, optional | the key to retrieve inside the Game VFS. If not present, then ALL the keys of the game will be returned.

This function returns a JSON with 1 key. Key `result` contains the key(s) and value(s) which were requested.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "result" : {
        "key1" : { "value JSON 1" },
        "key2" : "value 2",
        "...." : { "...." },
        "keyN" : { "value JSON N" }
    }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
    "name" : "KeyNotFound",
    "message" : "The specified key could not be found"
}
```

## Get game binary data

> To get game binary data, use this code:

```cpp
#include "CGameManager.h"

class MyGame
{
    void GetGameBinary()
    {
        CHJSON json;
        json.Put("key", "myBinaryKey");
        json.Put("domain", "private");
        CloudBuilder::CGameManager::Instance()->GetBinary(&json, MakeResultHandler(this, &MyGame::GetGameBinaryHandler));
    }

    void GetGameBinaryHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const void* buffer = result->BinaryPtr();
            size_t sizeBuffer = result->BinarySize();
        }
        else
            printf("Could not get game binary data due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void GetGameBinary()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        cloud.Game.GameVfs.Domain("private").GetBinary("myBinaryKey")
        .Done(getGameBinaryRes => {
            // In case your binary data contains some text, you can transfer the byte[] result into a string
            string str = System.Text.Encoding.UTF8.GetString(getGameBinaryRes);
            Debug.Log(str);
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get game binary data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLGameVFS.h"

void GetGameBinary()
{
    XLGameVFS* vfs = [[XLGameVFS alloc] initWithDomain:@"private"];
    [vfs getBinary:@"myBinaryKey" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *getGameBinaryRes) {
        if(error == nil)
        {
            // In case your binary data contains some text, you can transfer ask the NSData the string it holds
            NSData* text = [getGameBinaryRes objectForKey:@"result"];
            const char* str = text.bytes;
            NSLog(@"%s: %s", "Binary retrieved: ", str);
        }
        else
        {
            NSLog(@"Could not get game data: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`

function GetGameBinary()
{
    clan.vfs("private").getValue("myBinaryKey", function(error, getGameBinaryRes)
    {
      if(error)
		    ConsoleLog("Get game binary data error: " + JSON.stringify(error));
	    else
            // This call returns the url where to download your binary data. Use it depending the context you are in (web page, nodejs, ...)
		    ConsoleLog("Url to binary data: " + getGameBinaryRes.result.myKey);
    });
}
```

```http
GET /v3.0/vfs/{domain}/{key}?binary
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
```

This function is used to retrieve some game binary data stored in the Game VFS.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to retrieve the game binary data. By default, if not sent, it will be fetched in the `private` domain
key | String, required | the key to retrieve inside the Game VFS.

This function returns a JSON with 1 key. Key `result` contains the key and url to download the data.

<aside class="notice">
If using REST APIs or Javascript, you will not get the binary data directly. Instead, you will get the url where the data is stored, so it's
up to you to go and fetch the data, depending on the context where you are. SDKs (C++, C# and ObjC) on the other hand, will hide that for you
and will return directly the binary buffer to your callback.
</aside>

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "result" : {
        "myKey" : { "https://s3-eu-west-1.amazonaws.com/cloudbuilder.binaries.sandbox/com.clanofthecloud.cloudbuilder.azerty/GAME/binkey-410f9942116a3159797b204219ed0c74a5a80dce" }
    }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
    "name" : "KeyNotFound",
    "message" : "The specified key could not be found"
}
```
