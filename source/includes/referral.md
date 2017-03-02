# Referral

XtraLife provides a way for users to encourage their friends to download the game and reward the user and their friends automatically, to help
with developing the virality of the game. In this case, the user bringing other players in the game is called the godfather, and the players
joining the game through a referral code are called godchildren.

## Get a referral code

> To get a referral code, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void GetReferralCode()
    {
        CloudBuilder::CUserManager::Instance()->GetGodfatherCode("private", MakeResultHandler(this, &MyGame::GetReferralCodeHandler);
    }

    void GetReferralCodeHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Referral code: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not get referral code due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void GetReferralCode()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Godfather.Domain ("private").GenerateCode ().Done(getReferralCodeRes => {
			Debug.Log("Referral code: " + getReferralCodeRes);
		}, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get referral code: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetReferralCode()
{
    clan.withGamer(gamer).referral("private").getCode(function(error, getReferralCodeRes)
    {
      if(error)
		    ConsoleLog("Could not get referral code: " + JSON.stringify(error));
	    else
		    ConsoleLog("Referral code: " + JSON.stringify(getReferralCodeRes));
    });
}
```

```http
PUT /v2.6/gamer/referral/{domain}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to generate a referral code. This generated code can then be shared on social networks to bring new players in the game.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to generate a referral code.

This function returns a JSON with the key `godfathercode`. The value associated to this key is a short code to be used by other users to declare
who did refer them to the game.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "godfathercode": "nYCIOQ5v"
}
```

## Use a referral code

> To use a referral code, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void UseReferralCode()
    {
        CHJSON json, reward, transaction;
        transaction.Put("Gold", 100);
        reward.Put("transaction", transaction.Duplicate());
        reward.Put("domain", "private");
        reward.Put("description", "This is a transaction reward");
        json.Put("reward", reward.Duplicate());
        json.Put("domain", "private");
        CloudBuilder::CUserManager::Instance()->SetGodfather("nYCIOQ5v", &json, MakeResultHandler(this, &MyGame::UseReferralCodeHandler);
    }

    void UseReferralCodeHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Use referral code: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Use referral code failed due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void UseReferralCode()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        Bundle transaction = Bundle.CreateObject("Gold", 100);
        Bundle bundle = Bundle.CreateObject("transaction", transaction, "description", "This is a transaction reward", "domain", "private");
        currentGamer.Godfather.Domain ("private").UseCode ("nYCIOQ5v", bundle).Done(useReferralCodeRes => {
			Debug.Log("Use referral code: " + useReferralCodeRes.GamerData);
			}
		}, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not use referral code: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function UseReferralCode()
{
    var transaction = { Gold : 100};
    var reward = { transaction, descritpion : "This is a transaction reward", domain : "private" };
    clan.withGamer(gamer).referral("private").UseCode("nYCIOQ5v", reward, function(error, useReferralCodeRes)
    {
      if(error)
		    ConsoleLog("Could not use referral code: " + JSON.stringify(error));
	    else
		    ConsoleLog("Referral code succeeded: " + JSON.stringify(useReferralCodeRes));
    });
}
```

```http
POST /v2.6/gamer/godfather/{domain}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "godfather" : "nYCIOQ5v",
    "reward" : {
        "transaction" : {
            "Gold" : "100"
        },
        "description" : "This is a transaction reward",
        "domain" : "private"
    }
}
```

This function associates a user to his godfather by using a shortcode generated previously by the godfather. When the relationship is
established, a transaction is created for the godfather, in order to reward him for bringing a new user in the game.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which the relationship is established
godfather | String, required | the code used to identify the godfather
reward | JSON, optional | a transaction which can be passed to reward the godfather with virtual currencies

This function returns a JSON with the key `done`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "done": 1
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
    "name" : "MissingData",
    "message" : "The body doesn't contain the required field: godfather"
}
```

## Get a user godfather

> To get a user godfather, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void GetGodfather()
    {
        CloudBuilder::CUserManager::Instance()->GetGodfather("private", MakeResultHandler(this, &MyGame::GetGodfatherHandler);
    }

    void GetGodfatherHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Godfather: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not get godfather due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void GetGodfather()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Godfather.Domain ("private").GetGodfather ().Done(getGodfatherRes => {
			Debug.Log("Godfather: " + getGodfatherRes);
		}, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get godfather: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetGodfather()
{
    clan.withGamer(gamer).referral("private").getGodfather(function(error, getGodfatherRes)
    {
      if(error)
		    ConsoleLog("Could not get godfather: " + JSON.stringify(error));
	    else
		    ConsoleLog("Godfather: " + JSON.stringify(getGodfatherRes));
    });
}
```

```http
GET /v2.6/gamer/godfather/{domain}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to retrieve a user's godfather.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which we want to retrieve the godfather.

This function returns a JSON with the key `godfather`. The value associated to this key is a JSON object containing keys `gamer_id` and `profile` which
identify the godfather.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "godfather": {
    "gamer_id": "587f5d844877a1734ec079e6",
    "profile": {
      "email": "myEmail@gmail.com",
      "displayName": "myEmail",
      "lang": "en"
    }
  }
}
```

## Get godchildren

> To get a user godchildren, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void GetGodchildren()
    {
        CloudBuilder::CUserManager::Instance()->GetGodchildren("private", MakeResultHandler(this, &MyGame::GetGodchildrenHandler);
    }

    void GetGodchildrenHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Godchildren: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not get godchildren due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void GetGodchildren()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Godfather.Domain ("private").GetGodchildren ().Done(getGodchildrenRes => {
			Debug.Log("Godchildren: " + getGodchildrenRes);
		}, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get godchildren: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetGodchildren()
{
    clan.withGamer(gamer).referral("private").getGodchildren(function(error, getGodchildrenRes)
    {
      if(error)
		    ConsoleLog("Could not get godchildren: " + JSON.stringify(error));
	    else
		    ConsoleLog("Godchildren: " + JSON.stringify(getGodchildrenRes));
    });
}
```

```http
GET /v2.6/gamer/godchildren/{domain}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to retrieve a user's godchildren.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which we want to retrieve the godchildren.

This function returns a JSON with the key `godchildren`. The value associated to this key is a JSON array of objects containing keys `gamer_id` and `profile`
which identify the godchildren.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "godchildren": [
    {
      "gamer_id": "5873a117b69fa8c942c7df08",
      "profile": {
        "email": "myEmail@gmail.com",
        "displayName": "myEmail",
        "lang": "en"
      }
    },
    {
      "gamer_id": "5892e8be4d5c8b801309a40c",
      "profile": {
        "email": "john.doe@gmail.com",
        "firstName": "John",
        "lastName": "Doe",
        "avatar": "https://scontent.xx.fbcdn.net/v/t1.0-1/c0.0.50.50/p50x50/10373519_10152260884362633_225418314873465507_n.jpg?oh=f72ec709df01600de8567cdb45641242&oe=590497B3",
        "displayName": "John Doe",
        "lang": "en"
      }
    }
  ]
}
```
