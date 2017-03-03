# Friends

XtraLife lets your users manage their friends in the game. Users can be declared friends, can be unfriended, or even blacklisted.
Because user accounts can be linked to social networks, you have the possibility to match friends from different social networks
with XtraLife users. This lets developers aggregate friends from many social networks in a single database.

## List a user friends

> To list a user friends, use this code:

```cpp
#include "CTribeManager.h"

class MyGame
{
    void ListFriends()
    {
        CloudBuilder::CTribeManager::Instance()->ListFriends("private", MakeResultHandler(this, &MyGame::ListFriendsHandler);
    }

    void ListFriendsHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("List of friends: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not list friends due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void ListFriends()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Community.Domain("private").ListFriends()
        .Done(listFriendsRes => {
            Debug.Log("List of friends: " + listFriendsRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not list friends: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function ListFriends()
{
    clan.withGamer(gamer).friends("private").get(function(error, listFriendsRes)
    {
      if(error)
		    ConsoleLog("Could not list friends: " + JSON.stringify(error));
	    else
		    ConsoleLog("List of friends: " + JSON.stringify(listFriendsRes));
    });
}
```

```http
GET /v2.6/gamer/friends/{domain}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to get a list of all the user friends.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which friends need to be fetched

This function returns a JSON with the key `friends`. The value associated to this key is a JSON array of objects containing `gamer_id` and `profile` sections.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "friends": [
    {
      "gamer_id": "563a747bfcc7e918540a9937",
      "profile": {
        "email": "myEmail@gmail.com",
        "firstName": "John",
        "lastName": "Doe",
        "displayName": "John",
        "lang": "en"
      }
    }
  ]
}
```

## List blacklisted users

> To list blacklisted users, use this code:

```cpp
#include "CTribeManager.h"

class MyGame
{
    void Blacklisted()
    {
        CloudBuilder::CTribeManager::Instance()->BlacklistFriends("private", MakeResultHandler(this, &MyGame::BlacklistedHandler);
    }

    void BlacklistedHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("List of blacklisted users: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not list blacklisted users due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void Blacklisted()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Community.Domain("private").ListFriends(true)
        .Done(listFriendsRes => {
            Debug.Log("List of blacklisted users: " + listFriendsRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not list blacklisted users: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function Blacklisted()
{
    clan.withGamer(gamer).friends("private").getBlacklisted(function(error, getBlacklistedRes)
    {
      if(error)
		    ConsoleLog("Could not list blacklisted users: " + JSON.stringify(error));
	    else
		    ConsoleLog("List of blacklisted users: " + JSON.stringify(getBlacklistedRes));
    });
}
```

```http
GET /v2.6/gamer/friends/{domain}?status=blacklist
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to get a list of all the blacklisted users.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which friends need to be fetched

This function returns a JSON with the key `blacklisted`. The value associated to this key is a JSON array of objects containing `gamer_id` and `profile` sections.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "blacklisted": [
    {
      "gamer_id": "563a747bfcc7e918540a9937",
      "profile": {
        "email": "myEmail@gmail.com",
        "firstName": "John",
        "lastName": "Doe",
        "displayName": "John",
        "lang": "en"
      }
    }
  ]
}
```

## Modify relationship status

> To modify relationship status, use this code:

```cpp
#include "CTribeManager.h"

class MyGame
{
    void AddFriend()
    {
        CHJSON json;
        json.Put("id", "563a747bfcc7e918540a9937");
        json.Put("status", "add");
        json.Put("domain", "private");
        CloudBuilder::CUserManager::Instance()->ChangeRelationshipStatus(&json, MakeResultHandler(this, &MyGame::AddFriendHandler));
    }

    void AddFriendHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Relationship modified: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not modify relationship due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void AddFriend()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Community.Domain("private").ChangeRelationshipStatus("563a747bfcc7e918540a9937", FriendRelationshipStatus.Add, null)
        .Done(addFriendRes => {
            Debug.Log("Relationship modified: " + listTransactionsRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not modify relationship: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function AddFriend()
{
    clan.withGamer(gamer).friends("private").status("563a747bfcc7e918540a9937", "add", function(error, addFriendRes)
    {
      if(error)
		    ConsoleLog("Could not modify relationship: " + JSON.stringify(error));
	    else
		    ConsoleLog("Relationship modified: " + JSON.stringify(addFriendRes));
    });
}
```

```http
GET /v2.6/gamer/friends/{domain}?{gamer_id}?{status}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to modify the relationship between two users. The different statuses are:

1. No relationship
2. Friends
3. Blacklisted

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to change status between users
gamer_id | String, required | the internal id of the user who is involved in the relationship
status | String, required | the new status between the two users. Possible values are `add`, `blacklist` and `forget`

This function returns a JSON with the key `done`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "done" : 1
}
```

<aside class="warning">
Result JSON in case of faliure:
</aside>

```json
{
  "name" : "BadGamerID",
  "message" : "A passed gamer ID is invalid"
}
```

## Match network friends

> To match network friends, use this code:

```cpp
#include "CTribeManager.h"

class MyGame
{
    void ListNetworkFriends()
    {
        CHJSON json, user, details;
        details.Put("name", "John Doe");    // Put whatever you want in this JSON
        user.Put("Gcj0YsioS5Bj", details.Duplicate());  // Associate the previous JSON with the id from a social network
        json.Put("friends", user.Duplicate());   // Build your JSON with friends to send to our servers
        json.Put("domain", "private");
        json.Put("automatching", false);
        CloudBuilder::CTribeManager::Instance()->ListNetworkFriends(&json, MakeResultHandler(this, &MyGame::ListNetworkFriendsHandler);
    }

    void ListNetworkFriendsHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Network friends: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not list network friends due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void ListNetworkFriends()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Community.Domain("private").ListNetworkFriends()
        .Done(listNetworkFriendsRes => {
            Debug.Log("Network friends: " + listNetworkFriendsRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not list network friends: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function ListNetworkFriends()
{
    clan.withGamer(gamer).friends("private").networkFriends("facebook", , true, function(error, listNetworkFriendsRes)
    {
      if(error)
		    ConsoleLog("Could not list network friends: " + JSON.stringify(error));
	    else
		    ConsoleLog("Network friends: " + JSON.stringify(listNetworkFriendsRes));
    });
}
```

```http
POST /v2.12/gamer/friends/{domain}?{network}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "friends" : {
        "657277247810080" : { "data" : "Test Friends" }
    },
    "automatching" : true
}
```

This function is used to match friends from a social network with XtraLife users. By passing the list of friends in a social network from a user, the function
will return the users which were matched with their whole XtraLife profile. Additionally, you can ask for automatically friend status for all the matched users.
This is helpful for synchronizing friends in XtraLife with friends from social networks.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which friends need to be matched
friends | JSON,required | a JSON object containing a list of identifiers in the social network with associated arbitrary data
automatching | boolean, optional | use it to automatically synchronise friends from the social networks

This function returns a JSON with the key for the considered social network. The value is a list of users indexed by social network identifier and the XtraLife
user data.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "friends": [
    {
      "gamer_id": "563a747bfcc7e918540a9937",
      "profile": {
        "email": "myEmail@gmail.com",
        "firstName": "John",
        "lastName": "Doe",
        "displayName": "John",
        "lang": "en"
      }
    }
  ]
}
```
