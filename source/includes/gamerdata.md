# Users VFS (Virtual File System)

Users VFS is a key-value store you can use to associate data with a user. Use it to store state, preferences, ...
Any data your user would like to share among devices.

Because you might want to share User VFS data between games you (or other developers) wrote, User VFS is scoped by domains.


## Get user data

> To get user data, use this code:

```cpp
#include "CTribeManager.h"

class MyGame
{
    void GetUserData()
    {
        CHJSON json;
        json.Put("key", "myKey");
        json.Put("domain", "private");
        CloudBuilder::CUserManager::Instance()->GetValue(&json, MakeResultHandler(this, &MyGame::GetValueHandler));
    }

    void GetValueHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
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
````

```cs
using CotcSdk;

public class MyClass
{
    void GetUserData()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        Bundle key = new Bundle;
        key[key] = "myKey";
        currentGamer.GamerVfs.Domain("private").GetValue(key)
        .Done(getUserDataRes =>
        {
            Bundle result = GetUserData["result"];
            Debug.Log("User data: ") + result;
        })
        .Catch(ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get user data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
```

```http
```

This function is used to retrieve some user data stored in the User VFS.

Parameter | Type | Description
--------- | ---- | -----------
key | JSON, required | a JSON describing which keys to retrieve inside the User VFS. Sending an empty JSON will return all the keys. Alternatively, the JSON can be an array of keys to be returned, or just a string if you want to retrieve a single key
domain | String, required | the domain in which you want to retrieve the user data. By default, if not sent, it will be fetched in the `private`domain

This function returns a JSON with 1 key. Key `result` contains the keys which were fetched with the
associated values.

> Result JSON in case of success:

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
