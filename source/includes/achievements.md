# Achievements

XtraLife provides functionality allowing to save the progress and triggering of the user Achievements, like on most popular
platforms. This system is device independent and provides the tools required to notify XtraLifefrom progress and query of the
status achievements.
Achievements share functionality, and work with the Transaction system, since triggering achievements can be affected by the
transactions.

## List achievements

> To list achievements, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void ListAchievements()
    {
        XtraLife::CUserManager::Instance()->ListAchievements("private", XtraLife::MakeResultHandler(this, &MyGame::ListAchievementsHandler);
    }

    void ListAchievementsHandler(XtraLife::eErrorCode aErrorCode, const XtraLife::CCloudResult *aResult)
    {
        if(aErrorCode == XtraLife::eErrorCode::enNoErr)
        {
            const XtraLife::Helpers::CHJSON* json = aResult->GetJSON();
            printf("List of achievements: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not list achievements due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void ListAchievements()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Achievements.Domain ("private").List ().Done(listAchievementsRes => {
			foreach (var achievement in listAchievementsRes) {
				Debug.Log(achievement.Key + " : " + achievement.Value.Config.ToString());
			}
		}, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not list achievements: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function ListAchievements()
{
    clan.withGamer(gamer).achievements("private").list(function(error, listAchievementsRes)
    {
      if(error)
		    ConsoleLog("Could not list achievements: " + JSON.stringify(error));
	    else
		    ConsoleLog("Achievements: " + JSON.stringify(listAchievementsRes));
    });
}
```

```http
GET /v1/gamer/achievements/{domain} HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

```javascript--server
function __ListAchievements(params, customData, mod) {
    "use strict";
    // don't edit above this line // must be on line 3
  
    return this.achievement.getUserAchievements(this.game.getPrivateDomain(), params.user_id)
    .then(res => {
  	    mod.debug("Achievements: " + JSON.stringify(res));
        return res;
    })
  .catch(error => {
  	  throw new Error("Could not list achievements: " + error);
  });
} // must be on last line, no CR
```

This function is used to get a list of all the achievements for a domain.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to fetch the achievements definitions.

This function returns a JSON with the key `achievements`. The value associated to this key is a JSON list of achievements having as key the
name of each achievement. Each value for achievement is another JSON with keys `type` (only possible value currently is `limit`), `config`
which is defined by keys `unit` (linked to transactions) and `maxValue`, which is the value used to unlock the achievement, and `progress`
which is a number between 0 and 1 representing the progression before unlocking. If you have set some custom information for some
achievements, there will be another key `gamerdata` containing the information which was set previously.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "achievements": {
    "Kill 10 Monsters": {
      "type": "limit",
      "config": {
        "unit": "KilledMonsters",
        "maxValue": "10"
      },
      "progress": 0.8
    },
    "Kill 50 Monsters": {
      "type": "limit",
      "config": {
        "unit": "KilledMonsters",
        "maxValue": "50"
      },
      "progress": 0.35
    },
    "Discover Secret Path": {
      "type": "limit",
      "config": {
        "unit": "SecretPathDiscovered",
        "maxValue": "1"
      },
      "progress": 0
    }
  }
}
```

## Set custom information

> To set custome information for an achievement/user, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void SetAchievementData()
    {
        XtraLife::Helpers::CHJSON json, data;
        data.Put("OneShot", true);
        json.Put("domain", "private");
        json.Put("name", "Kill 10 Monsters");
        json.Put("data", data.Duplicate());
        XtraLife::CUserManager::Instance()->SetAchievementData(&json, XtraLife::MakeResultHandler(this, &MyGame::SetAchievementDataHandler);
    }

    void SetAchievementDataHandler(XtraLife::eErrorCode aErrorCode, const XtraLife::CCloudResult *aResult)
    {
        if(aErrorCode == XtraLife::eErrorCode::enNoErr)
        {
            const XtraLife::Helpers::CHJSON* json = aResult->GetJSON();
            printf("Achievement custom information: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not get achievement custom information due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void SetAchievementData()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Achievements.Domain ("private").AssociateData ("TestAchievement", Bundle.CreateObject("OneShot", true)).Done(setAchievementDataRes => {
			Debug.Log("Custom information" + setAchievementDataRes.GamerData);
			}
		}, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not set custom information: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function SetAchievementData()
{
    clan.withGamer(gamer).achievements("private").associateData("TestAchievement", { OneShot : true}, function(error, setAchievementDataRes)
    {
      if(error)
		    ConsoleLog("Could not set custom information: " + JSON.stringify(error));
	    else
		    ConsoleLog("Custom information: " + JSON.stringify(setAchievementDataRes));
    });
}
```

```http
GET /v1/gamer/achievements/{domain}/{achievement}/gamerdata HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "OneShot" : true
}
```

```javascript--server
function __SetAchievementData(params, customData, mod) {
    "use strict";
    // don't edit above this line // must be on line 3
  
    return this.achievement.modifyUserAchievementData(this.game.getPrivateDomain(), params.user_id, "TestAchievement", { OneShot : true})
    .then(res => {
        mod.debug("Custom information: " + JSON.stringify(res));
        return res;
    })
    .catch(error => {
  	    throw new Error("Could not set custom information: " + error);
    });
} // must be on last line, no CR
```

This function allows to associate some custom information to an achievement for the user. You can call it multiple times with different JSONs,
and they will be merged.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which the achievement will have associated data
name | String, required | the achievement to associate data to
data | JSON, required | the data to associate with the achievement

The JSON returned contains the key `achievement`and the value is a JSON containing both the achievement definition
(as retrieved in List Achievements) and a `gamerdata` key with all the previously attached custom information.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "achievement": {
    "type": "limit",
    "config": {
      "unit": "Gold",
      "maxValue": "10"
    },
    "progress": 1,
    "gamerData": {
      "unhidden": true,
      "Test": "Achievement",
      "key": "value",
      "OneShot": true
    }
  }
}
```
