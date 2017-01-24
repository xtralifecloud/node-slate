# Leaderboards

This module allows the user to send scores to leaderboards, and retrieve those scores in different ways.
Leaderboards are dynamic, they do not need to be created in the Back Office dashboard, nor do they need
to be setup in any way before sending scores. If you send a score to a leaderboard which does not exist
yet, it will be automatically created on XtraLife servers, and then the score will be added.

Scores can be retrieved in many ways, depending on what you want to retrieve:
1. Global: the best scores are retrieved
2. Social: the best scores from the player and his friends are retrieved
3. Centered: the scores around the player best score (placed in the middle) are retrieved

Because you might want to share leaderboards between games you (or other developers) wrote,leaderboards
are scoped by domains.


## Post a score

> To post a score, use this code:

```cpp
#include "CGameManager.h"

class MyGame
{
    void PostScore()
    {
        CloudBuilder::CGameManager::Instance()->Score(10000, "intermediateMode", "hightolow", "context for score",
        false, "private", MakeResultHandler(this, &MyGame::PostScoreHandler));
    }

    void PostScoreHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* result = aResult->GetJSON();
            printf("Score posted: %s\n", result->print_formatted().c_str());
        }
        else
            printf("Score not posted due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
````

```cs
using CotcSdk;

public class MyClass
{
    void PostScore()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Scores.Domain("private").Post(10000, "intermediateMode", ScoreOrder.HighToLow,
        "context for score", false)
        .Done(postScoreRes =>
        {
            Debug.Log("Post score: ") + postScoreRes;
        })
        .Catch(ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not post score: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function PostScore()
{
    clan.withGamer(gamer).leaderboards("private").post(10000, "intermediateMode", "hightolow",
        "context for score", false, function(error, postScoreRes)
    {
      if(error)
		    ConsoleLog("Post score error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Post score succeeded: " + JSON.stringify(postScoreRes));
    });
}
```

```http
POST /v2.6/gamer/scores/{domain}/{leaderboard}{?order}{&force}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "score" : 10000,
    "info" : "context for score"
}
```

This function is used to send a score to a leaderboard.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which post the score
leaderboard | String, required | the leaderboard in which the score will be inserted
score | Long, required | the value to be inserted in the leaderboard
order | String, optional | used to compare scores. For the best scores to be on top, use `hightolow` (value by default). Value `lowtohigh` could be used in a racing game for example
info | String, optional | a string which will be recorded with the score. Can be used to save some context about how the score was achieved
force | Boolean, optional | used to force a lower score (according to order rule) to be saved even if a greater score has already been saved before

This function returns a JSON with 2 or 3 keys. If the score sent is a new high score for the player, then
key `done` is returned with value 1, and the key `rank` gives the rank of the score in the global leaderboard.
If the score sent by the player is not his best score, the key `done` is returned with value 0, a key "msg" is
filled with "this is not the highest score" and the field `rank` gives the rank of the score if it had been
inserted in the leaderboard.

> Result JSON in case of success:

```json
{
    "done" : 1,
    "rank" : 1
}
```

## Get rank

> To get rank for a score, use this code:

```cpp
#include "CGameManager.h"

class MyGame
{
    void GetRank()
    {
        CloudBuilder::CGameManager::Instance()->GetRank(10000, "intermediateMode", "private", MakeResultHandler(this, &MyGame::GetRankHandler));
    }

    void GetRankHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* result = aResult->GetJSON();
            printf("Rank for score: %d\n", result->GetInt("rank");
        }
        else
            printf("Could not get rank for score due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
````

```cs
using CotcSdk;

public class MyClass
{
    void GetRank()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Scores.Domain("private").GetRank(10000, "intermediateMode")
        .Done(getRankRes =>
        {
            Debug.Log("Rank for score: ") + getRankRes["rank"];
        })
        .Catch(ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get rank for score: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetRank()
{
    clan.withGamer(gamer).leaderboards("private").getRank(10000, "intermediateMode", function(error, getRankRes)
    {
      if(error)
		    ConsoleLog("Could not get rank for score: " + JSON.stringify(error));
	    else
		    ConsoleLog("Rank for score: " + getRankRes.rank);
    });
}
```

```http
PUT /v2.6/gamer/scores/{domain}/{leaderboard}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "score" : 10000,
}
```

This function is used to get the corresponding rank of a score inside a leaderboard.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which post the score
leaderboard | String, required | the leaderboard in which the score will be inserted
score | Long, required | the value to be inserted in the leaderboard

This function returns a JSON with the key `rank`, giving the rank of the score in the global leaderboard.

> Result JSON in case of success:

```json
{
    "rank" : 1
}
```

## Best high scores (global)

> To get the best high scores, use this code:

```cpp
#include "CGameManager.h"

class MyGame
{
    void BestHighScores()
    {
        CloudBuilder::CGameManager::Instance()->BestHighScore(10, 1, "intermediateMode", "private", MakeResultHandler(this, &MyGame::BestHighScoresHandler));
    }

    void BestHighScoresHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* result = aResult->GetJSON();
            printf("Score posted: %s\n", result->print_formatted().c_str());
        }
        else
            printf("Score not posted due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
````

```cs
using CotcSdk;

public class MyClass
{
    void BestHighScores()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Scores.Domain("private").List("intermediateMode", 10, 0)
        .Done(bestHighScoresRes =>
        {
            foreach(var score in bestHighScoresRes)
                Debug.Log(score.Rank + ". " + score.GamerInfo["profile"]["displayName"] + ": " + score.Value);
        })
        .Catch(ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get best high scores: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function BestHighScores()
{
    clan.withGamer(gamer).leaderboards("private").getHighscores("intermediateMode", 1, 10, function(error, bestHighScoresRes)
    {
      if(error)
		    ConsoleLog("Best high scores error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Best high scores: " + JSON.stringify(bestHighScoresRes));
    });
}
```

```http
GET /v2.6/gamer/scores/{domain}/{leaderboard}?page=1&count=10
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to retrieve the best global scores from a leaderboard.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which post the score
leaderboard | String, required | the leaderboard in which the score will be inserted
page | Int, optional | the page number, 1 by default
count | Int, optional | the max returned rows per page, 10 by default

This function returns a JSON with 1 key, the name of the leaderboard which was queried. The value associated includes
information about the page retrieved and number of pages, and then an array holding information about the different
scores: `score value` and `post time`, attached `info` as well as the player associated to the score: `gamer_id` and
`profile`.

> Result JSON in case of success:

```json
{
  "doc-easy": {
    "maxpage": 1,
    "page": 1,
    "rankOfFirst": 1,
    "scores": [
      {
        "score": {
          "timestamp": "2017-01-23T14:34:14.624Z",
          "score": 100000,
          "info": "context for score"
        },
        "gamer_id": "587f5d844877a1734ec079e6",
        "profile": {
          "email": "myEmail@gmail.com",
          "displayName": "myEmail",
          "lang": "en"
        }
      },
      {
        "score": {
          "timestamp": "2017-01-23T14:56:44.684Z",
          "score": 5000,
          "info": "context for score"
        },
        "gamer_id": "5873a117b69fa8c942c7df08",
        "profile": {
          "email": "john.doe@gmail.com",
          "displayName": "John Doe",
          "lang": "en"
        }
      }
    ]
  }
}
```

