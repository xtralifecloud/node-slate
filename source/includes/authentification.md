# Authentication

Accounts can be managed in several ways:
1. Anonymous: the user doesn't need to enter any credentials in order to create this kind of  profile.
It is transparent for the user, but the disadvantage is it can not be reused on another device as is.
2. E-mail/password: users input their e-mail and a password of their choices (which is encrypted on
XtraLife servers) in order to authenticate.
3. Use of social networks: a generic system which can support any kind of social network. It uses the
identifier of the social network and a token authentification that is used by XtraLife to ensure the
identity is legit. Currently we support Facebook, Google+ and GameCenter but more can be added if
required.
4. External databases: for developers who already have their users in their own databases, it is
possible to create an XtraLife profile attached to these users. Developer can also provide a script
to authenticate users on their servers.

<aside class="notice">
JSON returned by any of the Login functions have **gamer_id** and **gamer_secret** fields,
which are internal credentials managed by XtraLife. It is a good practice to store these
two fields locally on the device after the first Login, so you can reuse on subsequent
logins, without worrying of the social network which was used.
</aside>

## Login Anonymous

> To login anonymously, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void LoginAnonymous()
    {
        CloudBuilder::CUserManager::Instance()->LoginAnonymous(NULL, MakeResultHandler(this, &MyGame::LoginHandler));
    }

    void LoginHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        //  If anonymous account creation succeeded, then we can dive in the data returned.
        if(aErrorCode == eErrorCode::enNoErr)
        {
            //  From the result, we get the embedded JSON, which could be displayed with a call to result->printFormatted();
            const CHJSON* json = aResult->GetJSON();
            //  Inside this JSON, we have lots of data that you can browse, for example here we'll grab profile and credentials.
            std::string id = json->GetString("gamer_id");
            std::string secret = json->GetString("gamer_secret");
            const CHJSON* profile = json->Get("profile");
            std::string name = profile->GetString("displayName");
        }
        else
            printf("Could not login due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
````

```cs
using CotcSdk;

public class MyClass
{
    void LoginAnonymous()
    {
        var cotc = FindObjectOfType<CotcGameObject>();

        cotc.GetCloud().Done(cloud => {
            cloud.LoginAnonymously()
            .Then(gamer => {
                Debug.Log("Signed in succeeded (ID = " + Gamer.GamerId + ")");
                Debug.Log("Login data: " + Gamer);
                Debug.Log("Server time: " + Gamer["servertime"]);
            })
            .Catch(ex => {
                // The exception should always be CotcException
                CotcException error = (CotcException)ex;
                Debug.LogError("Failed to login: " + error.ErrorCode + " (" + error.HttpStatusCode + ")");
            });
        });  
    }
}
```

```objectivec
void LoginAnonymous()
{
    XLGamer* gamer;
    
    gamer = [[XLGamer alloc] init];
    [gamer loginAnonymous:@{} completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *res) {
        if(!error) {
          NSLog(@"%@", [res description]);
        
          NSString *gamer_id = res[@"gamer_id"];
          NSString *gamer_secret = res[@"gamer_secret"];
        }
        else
          NSLog(@"Login failure: %@", [error description]);
    }];
}
```

```javascript
function LoginAnonymous()
{
  var clan = Clan("YourGameApiKey", "YourGameApiSecret", "sandbox");
  
  clan.loginAnonymous(null, function(error, gamer)
  {
    if(!error)
    {
      ConsoleLog("Display Name: " + gamer. profile.displayName);
    }
    else
    {
      ConsoleLog("Login error: " + JSON.stringify(error));
    }
  });
}
```

```http
POST /v1/login/anonymous
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
```

This function creates a new profile every time you call it. It's up to you to save the credentials
created the first time and reuse them later. The advantage is that it's completely transparent for
the player, but you can begin saving data for this profile, and convert this profile later to an
authenticated profile.

Parameter | Type | Description
--------- | ---- | -----------
options | JSON | An optional JSON which can contain the `thenBatch` key, if a batch is to be invoked just after the login process

This function returns a JSON with details from the created player including the `network`and `network ID`, the list
of games, different times (`server time`, registration `time`), the `profile`, `gamer_id`and `gamer_secret` which can
be used to login again later, ...

<aside class="success">
Result JSON:
</aside>

```json
{
  "network": "email",
  "networkid": "myEmail@gmail.com",
  "registerTime": "2017-01-09T14:41:27.423Z",
  "registerBy": "com.mycompany.mygame",
  "games": [
    {
      "appid": "com.mycompany.mygame",
      "ts": "2017-01-09T14:41:27.423Z",
      "lastlogin": "2017-01-13T00:00:00.000Z"
    }
  ],
  "profile": {
    "email": "myEmail@gmail.com",
    "displayName": "myEmail",
    "lang": "en"
  },
  "servertime": "2017-01-13T14:54:29.372Z",
  "gamer_id": "5873a117b69fa8c942c7df08",
  "gamer_secret": "c3b7c6fab599919b0c24487bf46d0e6069472df0"
}
```

## Login

> To login through a network, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void LoginNetwork()
    {
      CotCHelpers::CHJSON json;

      json.Put("network", "email");
      json.Put("id", "myEmail@gmail.com");
      json.Put("secret", "myPassword");
      CloudBuilder::CUserManager::Instance()->LoginNetwork(&json, MakeResultHandler(this, &MyGame::LoginNetworkHandler));
    }

    void LoginNetworkHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        //  If login succeeded, then we can dive in the data returned.
        if(aErrorCode == eErrorCode::enNoErr)
        {
            //  From the result, we get the embedded JSON, which could be displayed with a call to result->printFormatted();
            const CHJSON* json = aResult->GetJSON();
            //  Inside this JSON, we have lots of data that you can browse, for example here we'll grab profile and credentials.
            std::string id = json->GetString("gamer_id");
            std::string secret = json->GetString("gamer_secret");
            const CHJSON* profile = json->Get("profile");
            std::string name = profile->GetString("displayName");
        }
        else
            printf("Could not login due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
````

```cs
using CotcSdk;

public class MyClass
{
    void LoginNetwork()
    {
        var cotc = FindObjectOfType<CotcGameObject>();

        cotc.GetCloud().Done(cloud => {
            Cloud.Login(
                network: "email",
                networkId: "myEmail@gmail.com",
                networkSecret: "myPassword")
            .Done(gamer => {
                Debug.Log("Signed in succeeded (ID = " + Gamer.GamerId + ")");
                Debug.Log("Login data: " + Gamer);
                Debug.Log("Server time: " + Gamer["servertime"]);
            })
            .Catch(ex => {
                // The exception should always be CotcException
                CotcException error = (CotcException)ex;
                Debug.LogError("Failed to login: " + error.ErrorCode + " (" + error.HttpStatusCode + ")");
            });
        });  
    }
}
```

```objectivec
void LoginNetwork()
{
    XLGamer* gamer;
    
    gamer = [[XLGamer alloc] init];
    [gamer loginNetwork:@"email" id:@"myEmail@gmail.com" secret:@"myPassword" preventRegistration: false
    options:@{} completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *res) {
        if(!error) {
          NSLog(@"%@", [res description]);
        
          NSString *gamer_id = res[@"gamer_id"];
          NSString *gamer_secret = res[@"gamer_secret"];
        }
        else
          NSLog(@"Login failure: %@", [error description]);
    }];
}
```

```javascript
function LoginNetwork()
{
  var clan = Clan("YourGameApiKey", "YourGameApiSecret", "sandbox");

  clan.login("email", "myEmail@gmail.com", "myPassword", nil, function(error, gamer)
  {
    if(!error)
    {
      ConsoleLog("Display Name: " + gamer. profile.displayName);
    }
    else
    {
      ConsoleLog("Login error: " + JSON.stringify(error));
    }
  });
}
```

```http
POST /v1/login
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret

BODY
{
  "network" : "email",
  "id" : "myEmail@gmail.com",
  "secret" : "myPassword",
  "options" : {
                "preventRegistration" : false
              }
}
```

This function allows a player to log in the XtraLife servers. If the credentials have never been used
yet, then it will create a new profile. It's up to you then to save the `gamer_id` and `gamer_secret`
if you want to log the player in automatically the next time. If the credentials have already been used,
then the player is simply authenticated.
When a new profile is created, the HTTP result code returned is 201, and it's 200 otherwise, if the profile
already exists.

Parameter | Type | Description
--------- | ---- | -----------
network | String, required | can be any of ["anonymous", "email", "facebook", "googleplus", "gamecenter"]
id | String, required | the user ID for this network
secret | String, required | the user secret/token for this network
options | JSON | An optional JSON which can contain the `thenBatch` and `preventRegistration` keys. `thenBatch` is used to invoke a batch just after the login process, and `preventRegistration` to control if Login can create new profiles

<aside class="warning">
In the C++ SDK, you have to use "facebookId", "googleplusId" and "gamecenterId".
</aside>

This function returns a JSON with details from the logged in player including the `network`and `network ID`, the list
of games, different times (`server time`, registration `time`), the `profile`, the list of `devices`the user has used,
the list of `games` the user has already used, `gamer_id`and `gamer_secret` which can be used to login again later, ...
<aside class="notice">
Since the profile may already exists, there is another key which can bereturned in the JSON. If this profile
has some keys in its GamerVFS, then these keys will be inserted just after the `profile`key, and ordered by domains.
</aside>

> Result JSON:

```json
{
  "network": "email",
  "networkid": "myEmail@gmail.com",
  "registerTime": "2017-01-09T14:41:27.423Z",
  "registerBy": "com.clanofthecloud.cloudbuilder",
  "games": [
    {
      "appid": "com.clanofthecloud.cloudbuilder",
      "ts": "2017-01-09T14:41:27.423Z",
      "lastlogin": "2017-01-13T00:00:00.000Z"
    }
  ],
  "profile": {
    "email": "myEmail@gmail.com",
    "displayName": "myEmail",
    "lang": "en"
  },
  "devices": [
    {
      "id": "5A2D6EC6-21C8-47AA-BC74-95DB1D6DFDAD",
      "osname": "iOS",
      "osversion": "10.2",
      "name": "Michael’s MacBook Air",
      "model": "x86_64",
      "version": "1",
      "ts": "2017-01-16T13:14:29.274Z",
      "ip": "127.0.0.1"
    }
  ],
  "domains": [
    {
      "domain": "private",
      "fs": {
        "Test": "Value Test"
      }
    }
  ],
  "servertime": "2017-01-13T14:54:29.372Z",
  "gamer_id": "5873a117b69fa8c942c7df08",
  "gamer_secret": "c3b7c6fab599919b0c24487bf46d0e6069472df0"
}
```

## Resume Session

> To resume a session, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void ResumeSession()
    {
      CotCHelpers::CHJSON json;

      json.Put("id", "5873a117b69fa8c942c7df08");
      json.Put("secret", "c3b7c6fab599919b0c24487bf46d0e6069472df0");
      CloudBuilder::CUserManager::Instance()->ResumeSession(&json, MakeResultHandler(this, &MyGame::ResumeSessionHandler));
    }

    void ResumeSessionHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        //  If login succeeded, then we can dive in the data returned.
        if(aErrorCode == eErrorCode::enNoErr)
        {
            //  From the result, we get the embedded JSON, which could be displayed with a call to result->printFormatted();
            const CHJSON* json = aResult->GetJSON();
            //  Inside this JSON, we have lots of data that you can browse, for example here we'll grab profile and credentials.
            std::string id = json->GetString("gamer_id");
            std::string secret = json->GetString("gamer_secret");
            const CHJSON* profile = json->Get("profile");
            std::string name = profile->GetString("displayName");
        }
        else
            printf("Could not resume session due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
````

```cs
using CotcSdk;

public class MyClass
{
    void ResumeSession()
    {
        var cotc = FindObjectOfType<CotcGameObject>();

        cotc.GetCloud().Done(cloud => {
            Cloud.ResumeSession(
                gamerId: "5873a117b69fa8c942c7df08",
                gamerSecret: "c3b7c6fab599919b0c24487bf46d0e6069472df0")
            .Done(gamer => {
                Debug.Log("Signed in succeeded (ID = " + Gamer.GamerId + ")");
                Debug.Log("Login data: " + Gamer);
                Debug.Log("Server time: " + Gamer["servertime"]);
            })
            .Catch(ex => {
                // The exception should always be CotcException
                CotcException error = (CotcException)ex;
                Debug.LogError("Failed to login: " + error.ErrorCode + " (" + error.HttpStatusCode + ")");
            });
        });  
    }
}
```

```objectivec
void ResumeSession()
{
    XLGamer* gamer;
    
    gamer = [[XLGamer alloc] init];
    [gamer resumeSession:@"5873a117b69fa8c942c7df08" secret:@"c3b7c6fab599919b0c24487bf46d0e6069472df0"
      options:@{} completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *res) {
        if(!error) {
          NSLog(@"%@", [res description]);
        
          NSString *gamer_id = res[@"gamer_id"];
          NSString *gamer_secret = res[@"gamer_secret"];
        }
        else
          NSLog(@"Login failure: %@", [error description]);
    }];
}
```

```javascript
function ResumeSession()
{
  var clan = Clan("YourGameApiKey", "YourGameApiSecret", "sandbox");

  clan.resumeSession("5873a117b69fa8c942c7df08", "c3b7c6fab599919b0c24487bf46d0e6069472df0", function(error, gamer)
  {
    if(!error)
    {
      ConsoleLog("Display Name: " + gamer. profile.displayName);
    }
    else
    {
      ConsoleLog("Login error: " + JSON.stringify(error));
    }
  });
}
```

```http
POST /v1/login
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret

BODY
{
  "network" : "anonymous",
  "id" : "5873a117b69fa8c942c7df08",
  "secret" : "c3b7c6fab599919b0c24487bf46d0e6069472df0",
  "options" : {
                "preventRegistration" : true
              }
}
```

This function allows an existing player to log in the XtraLife servers. It needs XtraLife internal
credentials, so it is typically used after a first call to LoginAnonymous or Login, which can then
save the `gamer_id` and `gamer_secret` and reuse them automatically in ResumeSession.

Parameter | Type | Description
--------- | ---- | -----------
gamer_id | String, required | the internal XtraLife user ID
gamer_secret | String, required | the internal XtraLife secret
options | JSON | An optional JSON which can contain the `thenBatch` key. `thenBatch` is used to invoke a batch just after the login process

This function returns a JSON with details from the logged in player including the `network`and `network ID`, the list
of games, different times (`server time`, registration `time`), the `profile`, `gamer_id`and `gamer_secret` which can
be used to login again later, ...
<aside class="notice">
Since the profile may already exists, there is another key which can bereturned in the JSON. If this profile
has some keys in its GamerVFS, then these keys will be inserted just after the `profile`key, and ordered by domains.
</aside>

> Result JSON:

```json
{
  "network": "email",
  "networkid": "myEmail@gmail.com",
  "registerTime": "2017-01-09T14:41:27.423Z",
  "registerBy": "com.clanofthecloud.cloudbuilder",
  "games": [
    {
      "appid": "com.clanofthecloud.cloudbuilder",
      "ts": "2017-01-09T14:41:27.423Z",
      "lastlogin": "2017-01-13T00:00:00.000Z"
    }
  ],
  "profile": {
    "email": "myEmail@gmail.com",
    "displayName": "myEmail",
    "lang": "en"
  },
  "devices": [
    {
      "id": "5A2D6EC6-21C8-47AA-BC74-95DB1D6DFDAD",
      "osname": "iOS",
      "osversion": "10.2",
      "name": "Michael’s MacBook Air",
      "model": "x86_64",
      "version": "1",
      "ts": "2017-01-16T13:14:29.274Z",
      "ip": "127.0.0.1"
    }
  ],
  "domains": [
    {
      "domain": "private",
      "fs": {
        "Test": "Value Test"
      }
    }
  ],
  "servertime": "2017-01-13T14:54:29.372Z",
  "gamer_id": "5873a117b69fa8c942c7df08",
  "gamer_secret": "c3b7c6fab599919b0c24487bf46d0e6069472df0"
}
```

## Login with short code

> To login with a short code, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void LoginWithShortCode()
    {
      CotCHelpers::CHJSON json;

      json.Put("shortcode", "xxxxxxxx");
      CloudBuilder::CUserManager::Instance()->Logout(&json, MakeResultHandler(this, &MyGame::LoginWithShortCodeHandler));
    }

    void LoginWithShortCodeHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        //  If login with short code succeeded, then we can dive in the data returned.
        if(aErrorCode == eErrorCode::enNoErr)
        {
            //  From the result, we get the embedded JSON, which could be displayed with a call to result->printFormatted();
            const CHJSON* json = aResult->GetJSON();
            //  Inside this JSON, we have lots of data that you can browse, for example here we'll grab profile and credentials.
            std::string id = json->GetString("gamer_id");
            std::string secret = json->GetString("gamer_secret");
            const CHJSON* profile = json->Get("profile");
            std::string name = profile->GetString("displayName");
        }
        else
            printf("Could not login due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
````

```cs
using CotcSdk;

public class MyClass
{
    void LoginWithShortCode()
    {
        var cotc = FindObjectOfType<CotcGameObject>();

        cotc.GetCloud().Done(cloud => {
            Cloud.LoginWithShortCode("xxxxxxxx", null)
            .Done(gamer => {
                Debug.Log("Signed in succeeded (ID = " + Gamer.GamerId + ")");
                Debug.Log("Login data: " + Gamer);
                Debug.Log("Server time: " + Gamer["servertime"]);
            })
            .Catch(ex => {
                // The exception should always be CotcException
                CotcException error = (CotcException)ex;
                Debug.LogError("Failed to login: " + error.ErrorCode + " (" + error.HttpStatusCode + ")");
            });
        });  
    }
}
```

```objectivec
void LoginWithShortCode()
{
    XLGamer* gamer;
    
    gamer = [[XLGamer alloc] init];
    [gamer loginWithShortCode:@"xxxxxxxx" options:@{} completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *res) {
        if(!error) {
          NSLog(@"%@", [res description]);
        
          NSString *gamer_id = res[@"gamer_id"];
          NSString *gamer_secret = res[@"gamer_secret"];
        }
        else
          NSLog(@"Login failure: %@", [error description]);
    }];
}
```

```javascript
function LoginWithShortCode()
{
  var clan = Clan("YourGameApiKey", "YourGameApiSecret", "sandbox");

  clan.loginWithShortCode("xxxxxxxx", function(error, gamer)
  {
    if(!error)
    {
      ConsoleLog("Display Name: " + gamer. profile.displayName);
    }
    else
    {
      ConsoleLog("Login error: " + JSON.stringify(error));
    }
  });
}
```

```http
POST /v1/login
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret

BODY
{
  "network" : "restore",
  "id" : "",
  "secret" : "xxxxxxxx",
  "options" : {
                "preventRegistration" : true
              }
}
```

This function allows an existing player to login when he has forgotten his credentials. After getting
a short code generated by XtraLife servers, the short code can be sent as is to the LoginWithShortCode
function. Upon logging in, the developer will be able to retrieve the XtraLife internal credentials,
to save them and reuse them for later logins.

Parameter | Type | Description
--------- | ---- | -----------
shortcode | String, required | the shortcode generated in order to log in again

This function returns a JSON with details from the logged in player including the `network`and `network ID`, the list
of games, different times (`server time`, registration `time`), the `profile`, `gamer_id`and `gamer_secret` which can
be used to login again later, ...
<aside class="notice">
Since the profile may already exists, there is another key which can bereturned in the JSON. If this profile
has some keys in its GamerVFS, then these keys will be inserted just after the `profile`key, and ordered by domains.
</aside>

> Result JSON:

```json
{
  "network": "email",
  "networkid": "myEmail@gmail.com",
  "registerTime": "2017-01-09T14:41:27.423Z",
  "registerBy": "com.clanofthecloud.cloudbuilder",
  "games": [
    {
      "appid": "com.clanofthecloud.cloudbuilder",
      "ts": "2017-01-09T14:41:27.423Z",
      "lastlogin": "2017-01-13T00:00:00.000Z"
    }
  ],
  "profile": {
    "email": "myEmail@gmail.com",
    "displayName": "myEmail",
    "lang": "en"
  },
  "devices": [
    {
      "id": "5A2D6EC6-21C8-47AA-BC74-95DB1D6DFDAD",
      "osname": "iOS",
      "osversion": "10.2",
      "name": "Michael’s MacBook Air",
      "model": "x86_64",
      "version": "1",
      "ts": "2017-01-16T13:14:29.274Z",
      "ip": "127.0.0.1"
    }
  ],
  "domains": [
    {
      "domain": "private",
      "fs": {
        "Test": "Value Test"
      }
    }
  ],
  "servertime": "2017-01-13T14:54:29.372Z",
  "gamer_id": "5873a117b69fa8c942c7df08",
  "gamer_secret": "c3b7c6fab599919b0c24487bf46d0e6069472df0"
}
```

## Logout

> To log out from a session, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void Logout()
    {
      CloudBuilder::CUserManager::Instance()->Logout(MakeResultHandler(this, &MyGame::LogouHandler));
    }

    void LogouHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
          printf("Logout succeeded\n");
        else
          printf("Could not log out due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
````

```cs
using CotcSdk;

public class MyClass
{
    void Logout()
    {
        var cotc = FindObjectOfType<CotcGameObject>();

        cotc.GetCloud().Done(cloud => {
            Cloud.Logout()
            .Done(result => {
                Debug.Log("Logout succeeded");
            })
            .Catch(ex => {
                // The exception should always be CotcException
                CotcException error = (CotcException)ex;
                Debug.LogError("Failed to logout: " + error.ErrorCode + " (" + error.HttpStatusCode + ")");
            });
        });  
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function Logout()
{
  var clan = Clan("YourGameApiKey", "YourGameApiSecret", "sandbox");

  clan.withGamer(gamer).logout(function(error, logoutRes)
  {
    if(!error)
    {
      ConsoleLog("Logout succeeded");
    }
    else
    {
      ConsoleLog("Logout error: " + JSON.stringify(error));
    }
  });
}
```

```http
POST /v1/gamer/logout
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
}
```

This function allows an existing player to log out from the XtraLife servers. As such, it doesn't take any
parameters

Parameter | Type | Description
--------- | ---- | -----------

This function returns an empty JSON.

> Result JSON:

```json
{
}
```