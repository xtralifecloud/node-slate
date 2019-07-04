# Profile

Player's profile holds information like his/her display name or avatar's URL. It has the advantage to have its data
returned in numerous APIs where it's relevant (e.g. when you're asking for leaderboard's scores, you'll get all
returned gamers with their profile data too).

Profile data is global (shared across all games), so if you wish to store per-game profile data you should use your
own data structure stored in a Gamer VFS key instead (and adapt your API calls in batches to return it if needed).

The acceptable profile fields are: `avatar`, `email`, `displayName`, `firstName`, `lastName`, `lang`, `addr1`,
`addr2`, and `addr3`.

## Get profile data

> To get profile data, use this code:

```cpp
```

```csharp
using CotcSdk;

public class MyClass
{
    void GetProfileData()
    {
        // currentGamer is an object retrieved after one of the different Login functions.
        currentGamer.Profile.Get()
        .Done(profileRes => {
            Debug.Log("Profile data: " + profileRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get profile data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetProfileData()
{
    clan.withGamer(gamer).profile().get(function(error, profileRes)
    {
        if (error)
		    ConsoleLog("Get profile data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Profile data: " + profileRes);
    });
}
```

```http
GET /v1/gamer/profile
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret: YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

Retrieves player's profile data.

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
	"displayName": "Gamer A",
	"lang": "en",
	"avatar": "http://www.example.com/youravatar.png",
	"email": "test@email.com",
	"firstName": "Jane",
	"lastName": "Doe",
	"addr1": "Address 1",
	"addr2": "Address 2",
	"addr3": "Address 3"
}
```

## Set profile data

> To set profile data, use this code:

```cpp
```

```csharp
using CotcSdk;

public class MyClass
{
    void SetProfileData()
    {
		Bundle profileUpdates = Bundle.CreateObject();
		profileUpdates["displayName"] = new Bundle("New Nickname");
		profileUpdates["firstName"] = new Bundle("New Firstname");

        // currentGamer is an object retrieved after one of the different Login functions.
        currentGamer.Profile.Set(profileUpdates)
        .Done(profileRes => {
            Debug.Log("Profile data set: " + profileRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not set profile data due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetProfileData()
{
	const profileUpdates = { displayName: "New Nickname", firstName: "New Firstname" };

    clan.withGamer(gamer).profile().set(profileUpdates, function(error, setProfileRes)
    {
        if (error)
		    ConsoleLog("Set profile data error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Profile data set: " + setProfileRes);
    });
}
```

```http
POST /v1/gamer/profile
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret: YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{"displayName":"New Nickname","firstName":"New Firstname"}
```

Updates player's profile data. Only the given acceptable fields will be updated, the other ones will be kept untouched.

Parameter | Type | Description
--------- | ---- | -----------
profileUpdates | JSON, required | the profile fields to update

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
	"done": 1,
	"profile": {
		"displayName": "New Nickname",
		"lang": "en",
		"avatar": "http://www.example.com/youravatar.png",
		"email": "test@email.com",
		"firstName": "New Firstname",
		"lastName": "Doe",
		"addr1": "Address 1",
		"addr2": "Address 2",
		"addr3": "Address 3"
	}
}
```
