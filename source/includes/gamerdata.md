# Users VFS (Virtual File System)

Users VFS is a key-value store you can use to associate data with a user. Use it to store state, preferences, ...
Any data your user would like to share among devices. You can either store text through Key/Value (each user has its dedicated
JSON with no predefined keys), or you can store binary objects. In the last case, we'll simply store the binary object in
AWS S3, and we'll fill the value with the AWS S3 url. You won't have to make complex operations since our SDKs handle that
transparently for you (except in Javascript).

Because you might want to share User VFS data between games you (or other developers) wrote, User VFS is scoped by domains.


## Get user data

> To get user data, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void GetUserValue()
    {
        CHJSON json;
        json.Put("key", "myKey");
        json.Put("domain", "private");
        CloudBuilder::CUserManager::Instance()->GetValue(&json, MakeResultHandler(this, &MyGame::GetUserValueHandler));
    }

    void GetUserValueHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* result = aResult->GetJSON()->Get("result");
            printf("User data: %s\n", result->print_formatted().c_str());
        }
        else
            printf("Could not get user data due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void GetUserValue()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.GamerVfs.Domain("private").GetValue("myKey")
        .Done(getUserValueRes =>
        {
            Bundle result = getUserValueRes["result"];
            Debug.Log("User data: ") + result;
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get user data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLGamerVFS.h"

void GetUserValue()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods

    XLGamerVFS* vfs = [[XLGamerVFS alloc] initWithGamer:gamer andDomain:@"private"];
    [vfs getValue:@"myKey" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *getUserValueRes) {
        if(error == nil)
        {
            NSLog(@"User data: %@", [getUserValueRes description]);
        }
        else
        {
            NSLog(@"Could not get user data: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetUserValue()
{
    clan.withGamer(gamer).gamervfs("private").getValue("myKey", function(error, getUserValueRes)
    {
      if(error)
		    ConsoleLog("Get user data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("User data: " + getUserValueRes.result);
    });
}
```

```http
GET /v3.0/gamer/vfs/{domain}/{key}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to retrieve some user data stored in the User VFS. If you want to retrieve a binary object, you should use
method GetBinary instead because otherwise you will only have the url to the binary object, not the binary object itself.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to retrieve the user data. By default, if not sent, it will be fetched in the `private` domain
key | String, optional | the key to retrieve inside the User VFS. If not present, then ALL the keys of the user will be returned.

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

## Set user data

> To set user data, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void SetUserValue()
    {
        CHJSON json;
        json.Put("key", "myKey");
        json.Put("data", "myValue)
        json.Put("domain", "private");
        CloudBuilder::CUserManager::Instance()->SetValue(&json, MakeResultHandler(this, &MyGame::SetUserValueHandler));
    }

    void SetUserValueHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            printf("User data set: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not set user data due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void SetUserValue()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        Bundle value = new Bundle("myValue");
        currentGamer.GamerVfs.Domain("private").SetValue("myKey", value)
        .Done(setUserValueRes =>
        {
            Debug.Log("User data set: ") + setUserValueRes;
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not set user data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLGamerVFS.h"

void SetUserValue()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods

    XLGamerVFS* vfs = [[XLGamerVFS alloc] initWithGamer:gamer andDomain:@"private"];
    [vfs setValue:@"myKey" withValue:@"myValue" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *setUserValueRes) {
        if(error == nil)
        {
            NSLog(@"User data set: %@", [setUserValueRes description]);
        }
        else
        {
            NSLog(@"Could not set user data: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function SetUserValue()
{
    clan.withGamer(gamer).gamervfs("private").setValue("myKey", "myValue", function(error, setUserValueRes)
    {
      if(error)
		    ConsoleLog("Set user data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("User data set: " + setUserValueRes.result);
    });
}
```

```http
PUT /v3.0/gamer/vfs/{domain}/{key}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
"myValue"
```

This function is used to store some user data in the User VFS. Storage uses JSON objects, which means that you can also pass a string
(like in the samples above) or a number, or a boolean. `CHJSON` in C++ can be used to store full JSON objects or simple types, just like
the `Bundle` in C#. For ObjectiveC, `NSDictionary` can not store simple types, which is why the ObjectiveC method take a `(id)` parameter
for the value, and you can pass it a `NSDictionary`, a `NSString` or a `NSNumber` (which also manages the boolean).

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to store the user data. By default, if not sent, it will be saved in the `private` domain
key | String, optional | the key to store inside the User VFS. If not passed, all keys will be replaced
value | JSON, required | the value to store in the user data

This function returns a JSON with 1 key. Key `done` contains 1.

<aside class="warning">
Be very careful if you do not pass any key to this method. Doing so will replace ALL the existing keys with the value passed. If value is a JSON, then
it will save all the keys present in this JSON. It is therefore adviced to use this feature with great care.
</aside>

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "done" : 1
}
```

## Delete user data

> To delete user data, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void DeleteUserValue()
    {
        CHJSON json;
        json.Put("key", "myKey");
        json.Put("domain", "private");
        CloudBuilder::CUserManager::Instance()->DeleteValue(&json, MakeResultHandler(this, &MyGame::DeleteUserValueHandler));
    }

    void DeleteUserValueHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            printf("User data deleted: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not delete user data due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void DeleteUserValue()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.GamerVfs.Domain("private").DeleteValue("myKey")
        .Done(deleteUserValueRes =>
        {
            Debug.Log("User data deleted: ") + deleteUserValueRes;
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not delete user data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLGamerVFS.h"

void DeleteUserValue()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods

    XLGamerVFS* vfs = [[XLGamerVFS alloc] initWithGamer:gamer andDomain:@"private"];
    [vfs deleteValue:@"myKey" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *deleteUserValueRes) {
        if(error == nil)
        {
            NSLog(@"User data deleted: %@", [deleteUserValueRes description]);
        }
        else
        {
            NSLog(@"Could not delete user data: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function DeleteUserValue()
{
    clan.withGamer(gamer).gamervfs("private").deleteValue("myKey", function(error, deleteUserValueRes)
    {
      if(error)
		    ConsoleLog("Delete user data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("User data deleted: " + deleteUserValueRes);
    });
}
```

```http
DELETE /v3.0/gamer/vfs/{domain}/{key}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to delete some user data stored in the User VFS.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to delete the user data. By default, if not sent, it will be deleted in the `private` domain
key | String, optional | the key to delete with its associated value inside the User VFS.

This function returns a JSON with 1 key. Key `done` contains 1.

<aside class="warning">
Be very careful if you do not pass any key to this method. Doing so will delete ALL the existing keys. It is therefore adviced to use this feature with great care.
</aside>

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "done" : 1
}
```

## Get user binary data

> To get user binary data, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void GetUserBinary()
    {
        CHJSON json;
        json.Put("key", "myBinaryKey");
        json.Put("domain", "private");
        CloudBuilder::CUserManager::Instance()->GetBinary(&json, MakeResultHandler(this, &MyGame::GetUserBinaryHandler));
    }

    void GetUserBinaryHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const void* buffer = result->BinaryPtr();
            size_t sizeBuffer = result->BinarySize();
        }
        else
            printf("Could not get user binary data due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void GetUserBinary()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.GamerVfs.Domain("private").GetBinary("myBinaryKey")
        .Done(getUserBinaryRes =>
        {
            // In case your binary data contains some text, you can transfer the byte[] result into a string
            string str = System.Text.Encoding.UTF8.GetString(getUserBinaryRes);
            Debug.Log(str);
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get user binary data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLGamerVFS.h"

void GetUserBinary()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods

    XLGamerVFS* vfs = [[XLGamerVFS alloc] initWithGamer:gamer andDomain:@"private"];
    [vfs getBinary:@"myBinaryKey" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *getUserBinaryRes) {
        if(error == nil)
        {
            NSData* text = [getUserBinaryRes objectForKey:@"result"];
            const char* str = text.bytes;
            NSLog(@"%s", str);
         }
        else
        {
            NSLog(@"Could not get user binary data: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetUserBinary()
{
    clan.withGamer(gamer).gamervfs("private").getValue("myBinaryKey", function(error, getUserBinaryRes)
    {
      if(error)
		    ConsoleLog("Get user binary data error: " + JSON.stringify(error));
	    else
            // This call returns the url where to download your binary data. Use it depending the context you are in (web page, nodejs, ...)
		    ConsoleLog("Url to binary data: " + getUserBinaryRes.result.myKey);
    });
}
```

```http
GET /v3.0/gamer/vfs/{domain}/{key}?binary
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to retrieve some user binary data stored in the User VFS.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to retrieve the user binary data. By default, if not sent, it will be fetched in the `private` domain
key | String, required | the key to retrieve inside the User VFS.

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
        "myKey" : { "https://s3-eu-west-1.amazonaws.com/cloudbuilder.binaries.sandbox/com.clanofthecloud.cloudbuilder.azerty/587f5d844877a1734ec079e6/Key-410f9942116a3159797b204219ed0c74a5a80dce" }
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

## Set user binary data

> To set user binary data, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void SetUserBinary()
    {
        CHJSON json;
        json.Put("key", "myBinaryKey");
        json.Put("domain", "private");
        const char* str = "This is some text that you want to store as binary. Could be a PNG, a MP3 or anything.";
        CloudBuilder::CUserManager::Instance()->SetBinary(&json, str, strlen(str), MakeResultHandler(this, &MyGame::SetUserBinaryHandler));
    }

    void SetUserBinaryHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            printf("User binary data set: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not set user binary data due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void SetUserBinary()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        var bytes = System.Text.Encoding.UTF8.GetBytes("This is some text that you want to store as binary. Could be a PNG, a MP3 or anything.");
        currentGamer.GamerVfs.Domain("private").SetBinary("myBinaryKey", bytes)
        .Done(setUserBinaryRes =>
        {
            Debug.Log("User binary data set: ") + setUserBinaryRes;
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not set user binary data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLGamerVFS.h"

void SetUserBinary()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods

    XLGamerVFS* vfs = [[XLGamerVFS alloc] initWithGamer:gamer andDomain:@"private"];
    const char* str = "This is some text that you want to store as binary. Could be a PNG, a MP3 or anything.";
    NSData* data = [NSData dataWithBytes:str length:strlen(str)];
    [vfs setBinary:@"myBinaryKey" withData:data completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *setUserBinaryRes) {
        if(error == nil)
        {
            NSLog(@"User binary data set: %@", [setUserBinaryRes description]);
         }
        else
        {
            NSLog(@"Could not set user binary data: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function SetUserBinary()
{
    clan.withGamer(gamer).gamervfs("private").setBinary("myBinaryKey", function(error, setUserBinaryRes)
    {
      if(error)
		    ConsoleLog("Set user binary data error: " + JSON.stringify(error));
	    else {
		    ConsoleLog("url to upload data: " + setUserBinaryRes.putURL);
		    ConsoleLog("url to download data: " + setUserBinaryRes.getURL);
        }
    });
}
```

```http
PUT /v3.0/gamer/vfs/{domain}/{key}?binary
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to store some user binary data in the User VFS.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to save the user binary data. By default, if not sent, it will be saved in the `private` domain
key | String, required | the key to store inside the User VFS to get the urls to upload and download data

This function returns a JSON with 3 keys. Key `done` contains 1 if a success, `putURL` is the url to use to upload the binary data, and `getURL`
is the url to use to download the data once it's been uploaded.

<aside class="notice">
If using REST APIs or Javascript, you will not set the binary data directly. Instead, you will get the url where to store it, so it's
up to you to provide the upload operation, depending on the context where you are. SDKs (C++, C# and ObjC) on the other hand, will hide that
for you and will do both operations for you. As such, SDKs only return `getURL` since the upload is already done and `putURL` is not necessary.
</aside>

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "putURL": "https://s3-eu-west-1.amazonaws.com/cloudbuilder.binaries.sandbox/com.clanofthecloud.cloudbuilder.azerty/587f5d844877a1734ec079e6/Key-410f9942116a3159797b204219ed0c74a5a80dce?AWSAccessKeyId=AKIAIUXHMDS643YAVENQ&Expires=1485774091&Signature=QKo9gVcua8qMxC0VmoMQ8tySF48%3D",
  "getURL": "https://s3-eu-west-1.amazonaws.com/cloudbuilder.binaries.sandbox/com.clanofthecloud.cloudbuilder.azerty/587f5d844877a1734ec079e6/Key-410f9942116a3159797b204219ed0c74a5a80dce",
  "done": 1
}```
