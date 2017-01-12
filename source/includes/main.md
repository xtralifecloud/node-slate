# Introduction

Welcome to the XtraLife API documentation!

Here you will find APIs for C++, C#, ObjectiveC, Javascript as well as REST in case you're using a language we do not support yet.

You have access to code examples in the right area, and you can switch easily the programming language by browsing the tabs in the top right.

# Initialisation

> To setup the SDK, use this code:

```cpp
#include "CClan.h"

void MyGame::Startup()
{
    CotCHelpers::CHJSON json;
    json.Put("key", "YourGameApiKey");
    json.Put("secret", "YourGameApiSecret");
    json.Put("env", "sandbox");
    CloudBuilder::CClan::Instance()->Setup(&json,
      CloudBuilder::MakeResultHandler(this, &MyGame::SetupDone));
}

void MyGame::SetupDone(CloudBuilder::eErrorCode errorCode, const CloudBuilder::CCloudResult *result)
{
    if (errorCode == CloudBuilder::enNoErr)
    {
        printf("Setup complete! %s\n",
          result->GetJSON()->printFormatted().c_str());
    }
    else
    {
        printf("Setup failed with message %s!\n",
          result->GetErrorString());
    }
}
```

```cs
// With Unity, there is no code involved in setting up the environment,
// since it is done directly in the Settings Editor of the Clan of the Cloud object.
```

```javascript
//in your HTML file:
<script src="js/bundle.min.js" type="text/javascript"></script>

//in your Javascript file:
var clan = Clan("YourGameApiKey", "YourGameApiSecret", "sandbox");
```

```objectivec
#import "XLClan.h"

[[XLClan sharedInstance] configureWithApp:@"YourGameApiKey"
  andKey:@"YourGameApiSecret" endpoint:@"sandbox" timeout:XLDefaultTimeout];
```

```html
There is no Setup to do when accessing REST APIs directly. Identification of the game
will be done instead by the headers of every request you send to the servers:
x-apikey:YourGameApiKey
x-apisecret:YourGameApiSecret
```

Before making any actual request to the servers, you have to give the credentials
of the game you want to access with the SDK. Three parameters are needed to describe
the environment you want to work on.

Parameter | Type | Description
--------- | ---- | -----------
key | string | Unique identifier for your game. It is generated and can be retrieved from your [Account developer page](https://account.clanofthecloud.com).
secret | string | Authentication secret for your game. It is generated and can be retrieved from your [Account developer page](https://account.clanofthecloud.com).
env | string | Used to access the correct environment. Use "sandbox" when developing your game, and "prod" when publishing your game.

# Authentication

You have several ways to authenticate your players. The easiest and transparent way is to use
an `Anonymous` account. Or you can ask your players to authenticate through `email/password` or
through a `social network`. Currently, we support `Facebook`, `Google+` and `GameCenter`, but the
system has been designed to support any other social network or even your `own user database` if you
already have one.

Anonymous accounts can at any time be converted to an authenticated account by calling the
Convert function. This is useful if you want to store data right from the start, without
asking the player to authenticate when he launches the game for the first time. You will
only propose the Convert process when the player feels confident with the game.

JSON returned by any of the Login functions have **gamer_id** and **gamer_secret** fields,
which are internal credentials managed by XtraLife. It is a good practice to store these
two fields locally on the device after the first Login, so you can reuse on subsequent
logins, without worrying of the social network which was used.

A single profile can also be linked to several social networks. This is useful if you wish
to merge friends from several networks in XtraLife friends database. While a profile can be
linked to several social networks, it will always remain primilarily attached to a "master"
social network.

## Login Anonymous

> To login anonymously, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void AnonymousLogin()
    {
        CloudBuilder::CUserManager::Instance()->LoginAnonymous(NULL, MakeResultHandler(this, &MyGame::LoginHandler));
    }

    void LoginHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        //  If anonymous account creation went well, then we can dive in the data returned.
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
void AnonymousLogin()
{
  var cotc = FindObjectOfType<CotcGameObject>();

  cotc.GetCloud().Done(cloud => {
    cloud.LoginAnonymously()
      .Then(gamer => {
        Debug.Log("Signed in successfully (ID = " + Gamer.GamerId + ")");
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
```

```objectivec
void AnonymousLogin()
{
    XLGamer* gamer;
    
    gamer = [[XLGamer alloc] init];
    [gamer loginAnonymous:@{} completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *res) {
        NSLog(@"%@", [res description]);
        
        NSString *gamer_id = res[@"gamer_id"];
        NSString *gamer_secret = res[@"gamer_secret"];
    }];
}
```
