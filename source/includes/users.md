# Users

XtraLife APIs offer a few functions so you can query information about other players in the game.

## Find users

> To find users, use this code:

```cpp
#include "CTribeManager.h"

class MyGame
{
    void FindUsers()
    {
      CloudBuilder::CTribeManager::Instance()->ListUsers("matchPattern", 10, 0, MakeResultHandler(this, &MyGame::FindUsersHandler));
    }

    void FindUsersHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Matching profiles: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not find users due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void FindUsers()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        cloud.ListUsers("matchPattern", 10, 0)
        .Done(listUsersRes => {
            foreach (var userInfo in listUsersRes)
            {
                Debug.Log("User: " + userInfo.ToString());
            }
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Failed to list users: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function ListUsers()
{
    clan.withGamer(gamer).listUsers("matchPattern", 10, 0, function(error, listUsersRes)
    {
      if(error)
		    ConsoleLog("List users error: " + JSON.stringify(error));
	    else
		    ConsoleLog("List users succeeded: " + JSON.stringify(listUsersRes));
    });
}
```

```http
GET /v1/gamer?q=matchPattern&limit=10&skip=0
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
```

This function is used to retrieve a list of users from the XtraLife community. The match pattern will
be tested against e-mail address, display name and nick name from the profile.

Parameter | Type | Description
--------- | ---- | -----------
matchPattern | String, required | a string which is used to filter users
limit | Int, optional | the maximum number of users to be returned with the call
skip | Int, optional | the index of the first user to be returned

This function returns a JSON with 2 keys. First key is `count` and is the total number of users who
match the pattern sent to the request. Second key is `result` having as value an array of JSONs.
Each JSON contains fields `user_id`, `network`and `network_id` to identify the user, with the
`profile` JSON for this user.

<aside class="notice">
The `count` field is the total number of users, not the number of users returned in the JSON. If `count`
is 10 users and you passed a limit of 5, to get the next 5, you need to call it again with limit = 5 and
skip = 5. In Unity, the object returned is a PagedList<> so you don't need to call it again, the PagedList<>
object has helper functions to do it for you.
</aside>

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "count": 1,
  "result": [
    {
      "user_id": "563a747bfcc7e918540a9937",
      "network": "facebook",
      "networkid": "Aby0ZTdgUpsnS5Bj",
      "profile": {
        "email": "myEmail@gmail.com",
        "firstName": "John",
        "lastName": "Doe",
        "displayName": "John Doe",
        "lang": "en"
      }
    }
  ]
}
```

## Check user

> To check a user, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void CheckUser()
    {
      CloudBuilder::CUserManager::Instance()->UserExist("myEmail@gmail.com", "email", MakeResultHandler(this, &MyGame::UserExistHandler));
    }

    void UserExistHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("User: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not check user due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void CheckUser()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        cloud.UserExists("email", "myEmail@gmail.com")
        .Done(userExistsRes => {
            foreach (var userInfo in result)
            {
                Debug.Log("User: " + userExistsRes.ToString());
            }
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Failed to check user: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`

function CheckUser()
{
    clan.UserExists("email", "myEmail@gmail.com", function(error, userExistsRes)
    {
      if(error)
		    ConsoleLog("Check user error: " + JSON.stringify(error));
	    else
		    ConsoleLog("User: " + JSON.stringify(userExistsRes));
    });
}
```

```http
GET /v1/users/network/id
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
```

This function is used to check if a user is registered in XtraLife database by matching his network and
network_id.
<aside class="notice">
It can be called even if no user is currently logged in.
</aside>


Parameter | Type | Description
--------- | ---- | -----------
network | String, required | the network to check
id | String, required | the network id of the user to check

This function returns a JSON with the following keys: `network`, `networkid`, `registerTime`, `registerBy`,
`games` as an array, `profile` and `gamer_id`. 

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "network": "email",
  "networkid": "myEmail@gmail.com",
  "registerTime": "2017-01-18T12:20:20.869Z",
  "registerBy": "com.clanofthecloud.cloudbuilder",
  "games": [
    {
      "appid": "com.clanofthecloud.cloudbuilder",
      "ts": "2017-01-18T12:20:20.869Z",
      "lastlogin": "2017-01-18T00:00:00.000Z"
    }
  ],
  "profile": {
    "email": "myEmail@gmail.com",
    "displayName": "myEmail",
    "lang": "en"
  },
  "gamer_id": "587f5d844877a1734ec079e6"
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "BadGamerID",
  "message" : "A passed gamer ID is invalid"
}
```
