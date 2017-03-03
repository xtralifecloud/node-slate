# Match

XtraLife provides a simple way to run matches between a set of users. The match system is designed around a centralized game
state stored on XtraLife servers, with users participating to the match making a move, updating the global game state and
notifying the other users on an asynchronous basis.

This means that this match system is better suited to turn by turn game, rather than real time games (for which you should
check XtraLife Real Time product) requiring more sophisticated systems on your servers.

As most other XtraLife functionalities, matches are scoped by domain, allowing to share data amongst multiple games.
You can find below a list of all fields, giving the complete definition of matches:

Field | Description | Value
----- | ----------- | -----
_id | String | identifier for the match, keep it safe to allow for joining it or fetching it later
creator | String | identifier of the user who created the game
domain | String | domain to which the match belongs; used for scoping purposes
description | String | description of the game
globalState | JSON | global state of the match, which may be used to reconstruct the game locally
customProperties | JSON | contains the properties of the match (restricted to only one level, with values being numbers or strings)
lastEventId | String | identifier of the last event, which must be passed by any player who wishes to make a move
maxPlayers | Int | maximum number of players who can join this game
events | JSON | list of events, cleared whenever a globalState is posted along with a move by one of the players
players | Array | list of identifiers of users participating to the matches
seed | Int | a random number generated upon match creation, to be used by players
shoe | Array | an array of objects that are shuffled when the match starts. You can put anything you want inside and use it as values for your next game
status | String | either `running` (meaning that the match is running), or `finished` (meaning that nobody can join the match or make moves anymore)

**Match properties**<br>
The response to most calls will only include basic properties of the match. In order to get the whole definition of a match,
including the global state and events list, you must either create, join or fetch a single match.

**Global state and events list**<br>
Note that global state and events list are dependent. XtraLife adds a new event only if there is no global state passed to
the move. This means that at any time, if you consider the global state, and apply all the events contained in the list, you
will reach the current state of the match.
Alternatively, if you pass a global state to a move operation, then when storing the new global state, we will discard all the
current list of events in the match. It's up to you then to store sufficient information and data in the global state to resume
the game without the need for the history of events.

**Delivery of messages**<br>
All moves from players and the broadcast to other players are handled by XtraLife messaging system, ensuring that all moves
are reliably delivered to recipients, even if some of them are not currently playing. Instead, these messages are kept inside
a queue and will be submitted again when users relaunch the game. You also have the possibility to associate a push notification
automatically for the platforms which support this feature.

**Working with the shoe**
Along with the seed element that is returned with the detailed version of the match (i.e. when joining or fetching it),
the shoe is another element that helps building random generator-based games.A shoe, as in Casino, is basically a container
of possible values that are returned in a random order as they are poked. It could represent the values of the cards for instance.
It is possible to have more than once the same element in the shoe (as when having two card sets). The shoe is posted by the
person who creates the game and then shuffled. The shoe will remain hidden, with no one having access to it, until the match is
finished. Players can draw one or more elements off the shoe by posting a request to this resource. The shoe is shared with all
players, meaning that any element from the shoe is returned only once to a player having requested it. When all items have been
drawn from the shoe and more items are requested, the existing serie is duplicated, shuffled and appended to the current shoe,
meaning an endless play can be considered. When the match finishes, fetching detailed info about the match will return the shoe.
It can be used by all players to check that the game has been fair: should one player have hacked the game, it is possible to
detect it by comparing the shoe to the actual moves.

## Create match

> To create a match, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void CreateMatch()
    {
        CotCHelpers::CHJSON match, global, custom, shoe;
        int shoeArray[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        global.Put("startup", "globalState");
        custom.Put("newRules", 1);
        match.Put("domain", "private");
        match.Put("description", "Sample match for testing");
        match.Put("maxPlayers", 4);
        match.Put("globalState", global.Duplicate());
        match.Put("customProperties", custom.Duplicate());
        match.Put("shoe", CotCHelpers::CHJSON::Array(shoeArray, 10));
        CloudBuilder::CMatchManager::Instance()->CreateMatch(&match, MakeResultHandler(this, &MyGame::CreateMatchDone));
    }

    void CreateMatchDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Match created: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not create match due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void CreateMatch()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        Bundle[] array = new Bundle[10];
        int i = 1;
        for (; i <= 10; i++)
        {
            array[i-1] = new Bundle(i);
        }
        currentGamer.Matches.Domain("private").Create(3, "Sample match for testing", Bundle.CreateObject("startup", "globalState"), Bundle.CreateObject("newRules", 1), Bundle.CreateArray(array))
        .Done(createMatchRes => {
            Debug.Log("Match created: " + createMatchRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not create match: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void CreateMatch()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods
    
    XLMatch* match = [[XLMatch alloc] initWithGamer:gamer andDomain:@"private"];
    NSDictionary* custom = [NSDictionary dictionaryWithObjectsAndKeys:[NSNumber numberWithInteger: 1], @"newRules", nil];
    NSDictionary* state = [NSDictionary dictionaryWithObjectsAndKeys:@"globalState", @"statrtup", nil];
    NSArray* shoe = @[@1, @2, @3, @4, @5, @6, @7, @8, @9, @10];
    [match Create:@"Sample match for testing" maxPlayers:3 properties:custom globalState: state shoe:shoe
        completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *createMatchRes) {
        if(error == nil)
        {
            NSLog(@"Match created: %@", [res description]);
        }
        else
        {
            NSLog(@"Could not create match: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function CreateMatch()
{
    matchData = {
        description : "Sample match for testing",
        maxPlayers : 3,
        globalState : {
            startup : "globalState"
        },
        customProperties : {
            newRules : 1
        },
        shoe : [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    };
    clan.withGamer(gamer).matches("private").create(matchData, function(error, createMatchRes)
    {
      if(error)
		    ConsoleLog("Could not create match: " + JSON.stringify(error));
	    else
		    ConsoleLog("Match created: " + JSON.stringify(createMatchRes));
    });
}
```

```http
POST /v1/gamer/matches?{domain}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "description" : "Sample match for testing",
    "maxPlayers" : 3,
    "globalState" : {
        "startup" : "globalState"
    },
    "customProperties" : {
        "newRules" : 1
    },
    "shoe" : [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
}
```

This function is used to create a new match, which will then be available to invite other players. Once created,
any other request can be made (invite players, post a move, finish match, ...).

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to create the match
maxPlayers | Int, required | the maximum number of players which can participate in the match
description | String, optional | a string to describe the match
globalState | JSON, optional | a JSON where you can store anything you wish in order to store the state of the match
customProperties | JSON, optional | a "one level only" JSON containing only strings and numbers to associate properties to your match
shoe | JSON, optional | a JSON array of objects that are shuffled when the match starts. You can put anything you want inside and use it as values for your next match

This function returns a JSON with the key `match`, including a lot of information, like `_id` the identifier of the match, the creator
and its profile, the state of the match, the description, the global state, ... See below for a complete description of the result.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "match": {
    "domain": "private",
    "creator": {
      "gamer_id": "587f5d844877a1734ec079e6",
      "profile": {
        "email": "myEmail@gmail.com",
        "displayName": "myEmail",
        "lang": "en"
      }
    },
    "status": "running",
    "description": "Sample match for testing",
    "customProperties": {
      "newRules": 1
    },
    "maxPlayers": 3,
    "players": [
      {
        "gamer_id": "587f5d844877a1734ec079e6",
        "profile": {
          "email": "myEmail@gmail.com",
          "displayName": "myEmail",
          "lang": "en"
        }
      }
    ],
    "seed": 1775520418,
    "globalState": {
      "startup": "globalState"
    },
    "lastEventId": "589d9645f5973cc2647e289e",
    "events": [],
    "_id": "589d9645f5973cc2647e289f"
  }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "InvalidJSONBody",
  "message" : ""
}
```

## List matches

> To list matches, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void ListMatches()
    {
        CotCHelpers::CHJSON json;

        json.Put("domain", "private");
        json.Put("participating", true);
        json.Put("finished", true);
        json.Put("limit", 10);
        json.Put("skip", 0);
        CloudBuilder::CMatchManager::Instance()->ListMatches(&json, MakeResultHandler(this, &MyGame::ListMatchesDone));
    }

    void ListMatchesDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("List of matches: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not list matches due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void ListMatches()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Matches.Domain("private").List(true, false, true, false, 10, 0)
           .Done(listMatchesRes => {
               Debug.Log("List of matches: " + listMatchesRes.ToString());
           }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not list matches: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void ListMatches()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods
    
    XLMatch* match = [[XLMatch alloc] initWithGamer:gamer andDomain:@"private"];
    [match ListFromOffset:0 limit:10 participating:true invited:false finished:true full:false completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *listMatchesRes) {
        if(error == nil)
        {
            NSLog(@"List of matches: %@", [listMatchesRes description]);
        }
        else
        {
            NSLog(@"Could not list matches: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function ListMatches()
{
    clan.withGamer(gamer).matches("private").list({ participating : true, finished : true, limit : 10, skip : 0}, function(error, listMatchesRes)
    {
      if(error)
		    ConsoleLog("Could not list matches: " + JSON.stringify(error));
	    else
		    ConsoleLog("List of matches: " + JSON.stringify(listMatchesRes));
    });
}
```

```http
GET /v1/gamer/matches?{domain}{&participating}{&finished}{&invited}{&full}{&limit}{&skip}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to list matches with a few criteria based around the user. You should not use this function to manage
match making with other users. Instead, if you need a performing system to extract lists of matches based on your custom
criteria, you need to use XtraLife index features which is designed to handle this in a scalable way.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to list the matches
participating | Boolean, optional | use it if you want to retrieve only matches where the user is listed as a player
finished | Boolean, optional | use it if you want to list also matches which are already finished
invited | Boolean, optional | use when you want to retrieve matches where the user has been invited
full | Boolean, optional | use it to retrieve matches which are already full, that is where number of players is equal to the maximum number of players
limit | Int, optional | the maximum number of matches to be returned with this call
skip | Int, optional | the number of matches to skip before returning, used with `limit` for pagination.

This function returns a JSON with the keys `matches` and `count`. First key is associated to a JSON array of matches description,
and second one is the total number of matches, not the the number of matches in the array. Each match in the array includes `_id`,
`creator`, `status`, `description`, `customProperties` and `maxPlayers`. If you need more information about a match, you need
to load it independently. List only gives some of the information.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "matches": [
    {
      "_id": "589df75598a6d4427971d66f",
      "creator": {
        "gamer_id": "587f5d844877a1734ec079e6",
        "profile": {
          "email": "myEmail@gmail.com",
          "displayName": "myEmail",
          "lang": "en"
        }
      },
      "status": "running",
      "description": "Sample match for testing",
      "customProperties": {
        "newRules": 1
      },
      "maxPlayers": 3
    },
    {
      "_id": "589df42f98a6d4427971d66d",
      "creator": {
        "gamer_id": "587f5d844877a1734ec079e6",
        "profile": {
          "email": "myEmail@gmail.com",
          "displayName": "myEmail",
          "lang": "en"
        }
      },
      "status": "finished",
      "description": "Sample match for testing",
      "customProperties": {
        "newRules": 0
      },
      "maxPlayers": 4
    }
  ],
  "count": 2
}
```

## Load a match

> To load a match, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void LoadMatch()
    {
        CotCHelpers::CHJSON json;

        json.Put("id", "589df42f98a6d4427971d66d");
        CloudBuilder::CMatchManager::Instance()->FetchMatch(&json, MakeResultHandler(this, &MyGame::LoadMatchDone));
    }

    void LoadMatch(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Match loaded: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not load match due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void LoadMatch()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Matches.Fetch("589df42f98a6d4427971d66d")
           .Done(loadMatchRes => {
               Debug.Log("Match loaded: " + loadMatchRes.ToString());
           }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not load: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void LoadMatch()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods
    
    XLMatch* match = [[XLMatch alloc] initWithGamer:gamer andDomain:@"private"];
    [match Load:@"589df42f98a6d4427971d66d" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *loadMatchRes) {
        if(error == nil)
        {
            NSLog(@"Match loaded: %@", [loadMatchRes description]);
        }
        else
        {
            NSLog(@"Could not load match: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function LoadMatch()
{
    clan.withGamer(gamer).matches("private").get("589df42f98a6d4427971d66d", function(error, loadMatchRes)
    {
      if(error)
		    ConsoleLog("Could not load match: " + JSON.stringify(error));
	    else
		    ConsoleLog("Match loaded: " + JSON.stringify(loadMatchRes));
    });
}
```

```http
GET /v1/gamer/matches/{matchId}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to fully load a match with all its properties. Typically, you will first use List function or index
Manager to retrieve a list of possible matches and then once a user has selected a specific match, you will use this function
to retrieve all the properties in order to interact with it.

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | identifier of the match you want to retrieve

This function returns a JSON with the key `match`. The value associated to this key includes all the properties which define a
match: `_id`, `domain`, `creator`, `status`, `description`, `customProperties`, `maxPlayers`, `players` a JSON array of all
participating players, `invited`, a JSON array of all invited players in the match, `seed`, `globalState`, `lastEventId` and
`events`, a JSON array of the last events since the last globalState was set.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "match": {
    "_id": "589df75598a6d4427971d66f",
    "domain": "private",
    "creator": {
      "gamer_id": "587f5d844877a1734ec079e6",
      "profile": {
        "email": "myEmail@gmail.com",
        "displayName": "myEmail",
        "lang": "en"
      }
    },
    "status": "running",
    "description": "Sample match for testing",
    "customProperties": {
      "newRules": 1
    },
    "maxPlayers": 3,
    "players": [
      {
        "gamer_id": "587f5d844877a1734ec079e6",
        "profile": {
          "email": "myEmail@gmail.com",
          "displayName": "myEmail",
          "lang": "en"
        }
      }
    ],
    "seed": 1943816526,
    "globalState": {
      "startup": "globalState"
    },
    "lastEventId": "589df75598a6d4427971d66e",
    "events": []
  }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
    "name" : "InvalidMatch",
    "message" : "The match does not exist"
}
```

## Finish a match

> To finish a match, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void FinishMatch()
    {
        CotCHelpers::CHJSON json;

        json.Put("id", "589df75598a6d4427971d66f");
        json.Put("lastEventId", "589df75598a6d4427971d66e");
        CloudBuilder::CMatchManager::Instance()->FinishMatch(&json, MakeResultHandler(this, &MyGame::FinishMatchDone));
    }

    void FinishMatchDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Match finished: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not finish match due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void FinishMatch()
    {
        // match is a Match object retrieved after a call to GamerMatches::Create, GamerMatches::Fetch or GamerMatches::Join

        bool deleteMatch = false;
        match.Finish(deleteMatch, null)
        .Done(finishMatchRes => {
            Debug.Log("Match finished: " + finishMatchRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not finish match: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void FinishMatch()
{
    // match is a XLMatch instance obtained either by XLMatch.initWithGamer or XLMatch.findXLMatchWithID

    [match Finish:nil completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *finishMatchRes) {
        if(error == nil)
        {
            NSLog(@"Match finished: %@", [finishMatchRes description]);
        }
        else
        {
            NSLog(@"Could not finish match: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function FinishMatch()
{
    var id = "589df75598a6d4427971d66f";
    var lastEventId = "589df75598a6d4427971d66e";

    clan.withGamer(gamer).matches("private").finish(id, lastEventId, null, function(error, finishMatchRes)
    {
      if(error)
		    ConsoleLog("Could not finish match: " + JSON.stringify(error));
	    else
		    ConsoleLog("Match finished: " + JSON.stringify(finishMatchRes));
    });
}
```

```http
POST /v1/gamer/matches/{matchId}/finish?{lastEventId}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
}
```

This function is used to finish a match. Once this is done, you can consider the match as being frozen. You can not interact
with it in any way. No user can enter or leave it, no move can be done, the shoe can not be drawn. The only thing you can do
is deleting the match if you don't need it for later inspection.

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | the identifier of the match
lastEventId | String, required | the last event id returned by a successful request on the match (any operation altering the match will return a new last event id)

This function returns a JSON with the key `match`, including only the `_id`, the `status` which is now 'finished' and the
`lastEventId`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "match": {
        "_id": "589df75598a6d4427971d66f",
        "status": "finished",
        "lastEventId": "58a2f0bdf5973cc2647e29ca"
    }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "MissingParameter",
  "message" : "The parameter is invalid or absent: lastEventId"
}
```

## Delete a match

> To delete a match, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void DeleteMatch()
    {
        CotCHelpers::CHJSON json;

        json.Put("id", "589df75598a6d4427971d66f");
        CloudBuilder::CMatchManager::Instance()->DeleteMatch(&json, MakeResultHandler(this, &MyGame::DeleteMatchDone));
    }

    void DeleteMatchDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Match deleted: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not delete match due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void DeleteMatch()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Matches.Delete("589df75598a6d4427971d66f")
        .Done(deleteMatchRes => {
            Debug.Log("Match deleted: " + deleteMatchRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not delete match: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void DeleteMatch()
{
    // match is a XLMatch instance obtained either by XLMatch.initWithGamer or XLMatch.findXLMatchWithID

    [match Delete:^(NSError *error, NSInteger statusCode, NSDictionary *deleteMatchRes) {
        if(error == nil)
        {
            NSLog(@"Match deleted: %@", [deleteMatchRes description]);
        }
        else
        {
            NSLog(@"Could not delete match: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function DeleteMatch()
{
    clan.withGamer(gamer).matches("private").del("589df75598a6d4427971d66f", function(error, deleteMatchRes)
    {
      if(error)
		    ConsoleLog("Could not delete match: " + JSON.stringify(error));
	    else
		    ConsoleLog("Match deleted: " + JSON.stringify(deleteMatchRes));
    });
}
```

```http
DELETE /v1/gamer/matches/{matchId}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to delete a match. This is only possible if the match has been declared as finished before. Use this
method only if you don't need to access it later. It will be erased from XtraLife databases. Only the creator of a match
can delete it. No other user or even participating users are allowed to perform this operation.

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | the identifier of the match

This function returns a JSON with the key `done` and value 1.

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
  "name" : "MatchNotFinished",
  "message" : "This match needs to be finished in order to perform this operation"
}
```

## Invite a user to a match

> To invite a user to a match, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void InviteToMatch()
    {
        CotCHelpers::CHJSON json;

        json.Put("id", "589df75598a6d4427971d66f");
        json.Put("gamer_id", "582ce75598a6d4937152d69a");
        CloudBuilder::CMatchManager::Instance()->InvitePlayer(&json, MakeResultHandler(this, &MyGame::InviteToMatchDone));
    }

    void InviteToMatchDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Player invited: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not invite player due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void InviteToMatch()
    {
        // match is a Match object retrieved after a call to GamerMatches::Create, GamerMatches::Fetch or GamerMatches::Join

        match.InvitePlayer("582ce75598a6d4937152d69a")
        .Done(inviteToMatchRes => {
            Debug.Log("Player invited: " + inviteToMatchRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not invite player: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void InviteToMatch()
{
    // match is a XLMatch instance obtained either by XLMatch.initWithGamer or XLMatch.findXLMatchWithID

    [match InvitePlayer:@"582ce75598a6d4937152d69a" notification:nil completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *inviteToMatchRes) {
        if(error == nil)
        {
            NSLog(@"Player invited: %@", [inviteToMatchRes description]);
        }
        else
        {
            NSLog(@"Could not invite player: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function InviteToMatch()
{
    var id = "589df75598a6d4427971d66f";
    var friendId = "582ce75598a6d4937152d69a";

    clan.withGamer(gamer).matches("private").invite(id, friendId, null, function(error, inviteToMatchRes)
    {
      if(error)
		    ConsoleLog("Could not invite player: " + JSON.stringify(error));
	    else
		    ConsoleLog("Player invited: " + JSON.stringify(inviteToMatchRes));
    });
}
```

```http
POST /v1/gamer/matches/{matchId}/invite/{friendId}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
}
```

This function is used to invite a user to a match. It can be called at any time, just after creation or even after the
match has started. Only when the match is finished invitations are blocked.

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | the identifier of the match in which the user is invited
friendId | String, required | the identifier of the user you want to invite to the match

This function returns a JSON with the key `match`, including only the `_id`, the `status` and the `lastEventId`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "match": {
        "_id": "589df75598a6d4427971d66f",
        "status": "running",
        "lastEventId": "58a2f0bdf5973cc2647e29ca"
    }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "AlreadyInvitedToMatch",
  "message" : "The player is already invited to the match"
}
```

## Join a match

> To join a match, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void JoinMatch()
    {
        CotCHelpers::CHJSON json;

        json.Put("id", "589df75598a6d4427971d66f");
        CloudBuilder::CMatchManager::Instance()->JoinMatch(&json, MakeResultHandler(this, &MyGame::JoinMatchDone));
    }

    void JoinMatchDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Joined match: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not join match due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void JoinMatch()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Matches.Join("589df75598a6d4427971d66f")
        .Done(joinMatchRes => {
               Debug.Log("Joined match: " + joinMatchRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not join match: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void JoinMatch()
{
    XLMatch* match = [[XLMatch alloc] initWithGamer:gamer andDomain:@"private"];
    [match Join:@"589df75598a6d4427971d66f" notification:nil completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *joinMatchRes) {
        if(error == nil)
        {
            NSLog(@"Joined match: %@", [joinMatchRes description]);
        }
        else
        {
            NSLog(@"Could not join match: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function JoinMatch()
{
    clan.withGamer(gamer).matches("private").join("589df75598a6d4427971d66f", null, function(error, joinMatchRes)
    {
      if(error)
		    ConsoleLog("Could not join match: " + JSON.stringify(error));
	    else
		    ConsoleLog("Joined match: " + JSON.stringify(joinMatchRes));
    });
}
```

```http
POST /v1/gamer/matches/{matchId}/join
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
}
```

This function is used to join a match at any time. A user doesn't need to be invited to be able to join a match. It's up to
the developer to implement the logic if a user can join a match or not. Doing so will simply add the user joining the match
in the `players` properties of the match. It will also generate an event added to the array of events belonging to the match.
Two conditions will prevent a user from joining a match:

1. The maximum number of players is already reached, so unless another user leaves the match, nobody will be able to join
2. The match status has already been switched to `finished`, and it is now frozen, so nobody can join it

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | the identifier of the match to join

This function returns a JSON with the key `match`, with all the usual properties associated (See match description).

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "match": {
    "_id": "58a33021f5973cc2647e29f3",
    "domain": "private",
    "creator": {
      "gamer_id": "587f5d844877a1734ec079e6",
      "profile": {
        "email": "myEmail@gmail.com",
        "displayName": "myEmail",
        "lang": "en"
      }
    },
    "status": "running",
    "description": "Sample match for testing",
    "customProperties": {
      "newRules": 1
    },
    "maxPlayers": 3,
    "players": [
      {
        "gamer_id": "587f5d844877a1734ec079e6",
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
          "displayName": "John Doe",
          "lang": "en"
        }
      }
    ],
    "seed": 737848228,
    "globalState": {
      "startup": "globalState"
    },
    "lastEventId": "58a4299498a6d4427971d838",
    "events": [
      {
        "type": "match.join",
        "event": {
          "_id": "58a4299498a6d4427971d838",
          "playersJoined": [
            {
              "gamer_id": "5892e8be4d5c8b801309a40c",
              "profile": {
                "email": "john.doe@gmail.com",
                "firstName": "John",
                "lastName": "Doe",
                "displayName": "John Doe",
                "lang": "en"
              }
            }
          ]
        }
      }
    ]
  }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "BadMatchID",
  "message" : "This match does not exist or is not active"
}
```

## Dismiss an invite to a match

> To dismiss an invite to a match, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void DismissInvite()
    {
        CotCHelpers::CHJSON json;

        json.Put("id", "589df75598a6d4427971d66f");
        CloudBuilder::CMatchManager::Instance()->InvitePlayer(&json, MakeResultHandler(this, &MyGame::DismissInviteDone));
    }

    void DismissInviteDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Invite dismissed: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not dismiss invite due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void DismissInvite()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Matches.DismissInvitation("589df75598a6d4427971d66f")
        .Done(dismissInviteRes => {
            Debug.Log("Invite dismissed: " + dismissInviteRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not dismiss invite: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void DismissInvite()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods
    
    XLMatch* match = [[XLMatch alloc] initWithGamer:gamer andDomain:@"private"];
    [match DismissInvitation:@"589df75598a6d4427971d66f" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *dismissInviteRes) {
        if(error == nil)
        {
            NSLog(@"Invite dismissed: %@", [dismissInviteRes description]);
        }
        else
        {
            NSLog(@"Could not dismiss invite: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function DismissInvite()
{
    clan.withGamer(gamer).matches("private").dismiss(589df75598a6d4427971d66f, null, function(error, dismissInviteRes)
    {
      if(error)
		    ConsoleLog("Could not dismiss invite: " + JSON.stringify(error));
	    else
		    ConsoleLog("Invite dismissed: " + JSON.stringify(dismissInviteRes));
    });
}
```

```http
DELETE /v1/gamer/matches/{matchId}/invitation
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to decline an invitation to a match.

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | the identifier of the match in which the user wants to decline the invitation

This function returns a JSON with the key `match`, including only the `_id`, the `status` and the `lastEventId`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
    "match": {
        "_id": "589df75598a6d4427971d66f",
        "status": "running",
        "lastEventId": "58a2f0bdf5973cc2647e29ca"
    }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "BadMatchID",
  "message" : "This match does not exist or is not active"
}
```

## Make a move

> To make a move, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void PostMove()
    {
        CotCHelpers::CHJSON json, move, globalState;

        move.Put("pawn1", "A2-A4");
        globalState.Put("board"), "the board content";
        json.Put("id", "589df75598a6d4427971d66f");
        json.Put("lastEventId", "58a2f0bdf5973cc2647e29ca");
        json.Put("move", move.Duplicate());
        json.Put("globalState", globalState.Duplicate());
        CloudBuilder::CMatchManager::Instance()->JoinMatch(&json, MakeResultHandler(this, &MyGame::PostMoveDone));
    }

    void PostMoveDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Move posted: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not post move due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void PostMove()
    {
        // match is a Match object retrieved after a call to GamerMatches::Create, GamerMatches::Fetch or GamerMatches::Join

        Bundle move = Bundle.CreateObject("pawn1", "A2-A4");
        Bundle globalState = Bundle.CreateObject("board", "the board content");
        match.PostMove(move, globalState)
           .Done(postMoveRes => {
               Debug.Log("Move posted: " + postMoveRes.ToString());
            }, ex => {
                // The exception should always be CotcException
                CotcException error = (CotcException)ex;
               Debug.LogError("Could not post move: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
            });
    }
}
```

```objectivec
#import "XLMatch.h"

void PostMove()
{
    // match is a XLMatch instance obtained either by XLMatch.initWithGamer or XLMatch.findXLMatchWithID

    NSDictionary* move =  = [NSDictionary dictionaryWithObjectsAndKeys: @"A2-A4", @"pawn1", nil];
    NSDictionary* globalState =  = [NSDictionary dictionaryWithObjectsAndKeys: @"the board content", @"board", nil];
    [match PostMove:move newGlobalState:globalState notification:nil completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *postMoveRes) {
        if(error == nil)
        {
            NSLog(@"Move posted: %@", [postMoveRes description]);
        }
        else
        {
            NSLog(@"Could not post move: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function PostMove()
{
    var id = "589df75598a6d4427971d66f";
    var lastEventId = "58a2f0bdf5973cc2647e29ca";
    var move = { pawn1 : "A2-A4" };
    var globalState = { board : "the board content" };
    clan.withGamer(gamer).matches("private").move(id, lastEventId, move, globalState, null, function(error, postMoveRes)
    {
      if(error)
		    ConsoleLog("Could not post move: " + JSON.stringify(error));
	    else
		    ConsoleLog("Move posted: " + JSON.stringify(postMoveRes));
    });
}
```

```http
POST /v1/gamer/matches/{matchId}/move?{lastEventId}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "move" : {
        "pawn1" : "A2-A4"
    },
    "globalState" : {
        "board" : "the board content"
    }
}
```

This function is used to send a move to a match in order to progress into the game. A move is propagated to all other players
participating to this match. As said above, if a global state is passed, then all other previous events in the array for the
events history are discarded. In any case, the move which was just sent is appended to the events history, making it the sole
event if a global state was passed.
When a global state is passed, it does not automatically replace the existing global state. Instead, only already existing
keys are replaced. If you send keys which are not yet present in the current global state, they will be appended to the existing
ones. All other keys in the current global state, not present in the passed global state will not be affected.

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | the identifier of the match where to post a move
lastEventId | String, required | the last event id returned by a successful request on the match (any operation altering the match will return a new last event id)
move | JSON, required | a JSON holding the information about the move which was done inside the game
globalState | JSON, optional | a jSON where some global information can be stored and accessible from the match description

This function returns a JSON with the key `match`, including only the `_id`, the `status` and the `lastEventId`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "match": {
    "_id": "58a33021f5973cc2647e29f3",
    "status": "running",
    "lastEventId": "58a4299498a6d4427971d838"
  }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "InvalidJSONBody",
  "message" : ""
}
```

## Draw from the shoe

> To draw from the shoe, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void DrawFromShoe()
    {
        CotCHelpers::CHJSON json;

        json.Put("id", "589df75598a6d4427971d66f");
        json.Put("lastEventId", "58a2f0bdf5973cc2647e29ca");
        json.Put("count", 3);
        CloudBuilder::CMatchManager::Instance()->DrawFromShoe(&json, MakeResultHandler(this, &MyGame::DrawFromShoeDone));
    }

    void DrawFromShoeDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Draw from shoe: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not draw from shoe due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void DrawFromShoe()
    {
        // match is a Match object retrieved after a call to GamerMatches::Create, GamerMatches::Fetch or GamerMatches::Join

        match.DrawFromShoe(3)
           .Done(drawFromShoeRes => {
               Debug.Log("Draw from shoe: " + drawFromShoeRes.ToString());
            }, ex => {
                // The exception should always be CotcException
                CotcException error = (CotcException)ex;
                Debug.LogError("Could not draw from shoe: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
            });
    }
}
```

```objectivec
#import "XLMatch.h"

void DrawFromShoe()
{
    // match is a XLMatch instance obtained either by XLMatch.initWithGamer or XLMatch.findXLMatchWithID

    [match DrawFromShoe:move notification:nil completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *drawFromShoeRes) {
        if(error == nil)
        {
            NSLog(@"Draw from shoe: %@", [drawFromShoeRes description]);
        }
        else
        {
            NSLog(@"Could not draw from shoe: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function DrawFromShoe()
{
    var id = "589df75598a6d4427971d66f";
    var lastEventId = "58a2f0bdf5973cc2647e29ca";
    clan.withGamer(gamer).matches("private").move(id, lastEventId, 3, null, function(error, drawFromShoeRes)
    {
      if(error)
		    ConsoleLog("Could not draw from shoe: " + JSON.stringify(error));
	    else
		    ConsoleLog("Draw from shoe: " + JSON.stringify(drawFromShoeRes));
    });
}
```

```http
POST /v1/gamer/matches/{matchId}/shoe/draw?{lastEventId}&{count}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
}
```

This function is used to draw objects from the shoe, provided that one was passed at the time of the creation.

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | the identifier of the match in which to draw from the shoe
lastEventId | String, required | the last event id returned by a successful request on the match (any operation altering the match will return a new last event id)
count | Int, required | the number of objects to retrieve from the shoe

This function returns a JSON with the keys `match`, including only the `_id`, the `status` and the `lastEventId`, and also
`drawnItems` having as value a JSON array of objects retrieved from the shoe.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "match": {
    "_id": "58a43023f5973cc2647e2a6d",
    "status": "running",
    "lastEventId": "58a47d6298a6d4427971d8d3"
  },
  "drawnItems": [
    10,
    8
  ]
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "NoShoeInMatch",
  "message" : "Unable to draw from the shoe unless items have been put in it at creation"
}
```

## Leave a match

> To leave from a match, use this code:

```cpp
#include "CMatchManager.h"

class MyGame
{
    void LeaveMatch()
    {
        CotCHelpers::CHJSON json;

        json.Put("id", "589df75598a6d4427971d66f");
        CloudBuilder::CMatchManager::Instance()->LeaveMatch(&json, MakeResultHandler(this, &MyGame::LeaveMatchDone));
    }

    void LeaveMatchDone(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Left the match: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not leave the match due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void LeaveMatch()
    {
        // match is a Match object retrieved after a call to GamerMatches::Create, GamerMatches::Fetch or GamerMatches::Join

        match.Leave()
           .Done(leaveMatchRes => {
               Debug.Log("Left the match: " + leaveMatchRes.ToString());
           }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not leave the match: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
#import "XLMatch.h"

void LeaveMatch()
{
    // match is a XLMatch instance obtained either by XLMatch.initWithGamer or XLMatch.findXLMatchWithID

    [match Leave:nil completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *leaveMatchRes) {
        if(error == nil)
        {
            NSLog(@"Left the match: %@", [leaveMatchRes description]);
        }
        else
        {
            NSLog(@"Could notleave the match: %@", [error description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function LeaveMatch()
{
    clan.withGamer(gamer).matches("private").leave("589df75598a6d4427971d66f", null, function(error, leaveMatchRes)
    {
      if(error)
		    ConsoleLog("Could notleave the match: " + JSON.stringify(error));
	    else
		    ConsoleLog("Left the match: " + JSON.stringify(leaveMatchRes));
    });
}
```

```http
POST /v1/gamer/matches/{matchId}/leave
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
}
```

This function is used to leave a match. This is only possible if the status of the match is still `running`. If it has
already been switched to `finished`, leaving a match is not possible.

Parameter | Type | Description
--------- | ---- | -----------
matchId | String, required | the identifier of the match the user wants to leave

This function returns a JSON with the key `match`, including only the `_id`, the `status` and the `lastEventId`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "match": {
    "_id": "58a43023f5973cc2647e2a6d",
    "status": "running",
    "lastEventId": "58a47d6298a6d4427971d8d3"
  }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "BadMatchID",
  "message" : "This match does not exist or is not active"
}
```
