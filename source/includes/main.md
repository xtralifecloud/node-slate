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
    XtraLife::Helpers::CHJSON json;
    json.Put("key", "YourGameApiKey");
    json.Put("secret", "YourGameApiSecret");
    json.Put("env", "sandbox");
    XtraLife::CClan::Instance()->Setup(&json, XtraLife::MakeResultHandler(this, &MyGame::SetupDone));
}

void MyGame::SetupDone(XtraLife::eErrorCode errorCode, const XtraLife::CCloudResult *result)
{
    if (errorCode == XtraLife::eErrorCode::enNoErr)
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

```csharp
// With Unity, there is no code involved in setting up the environment,
// since it is done directly in the Settings Editor of the Clan of the Cloud object.
```

```javascript--client
//in your HTML file:
<script src="js/bundle.min.js" type="text/javascript"></script>

//in your Javascript file:
var clan = Clan("YourGameApiKey", "YourGameApiSecret", "sandbox");
```

```objective_c
#import "XLClan.h"

[[XLClan sharedInstance] configureWithApp:@"YourGameApiKey"
  andKey:@"YourGameApiSecret" endpoint:@"sandbox" timeout:XLDefaultTimeout];
```

```http
There is no Setup to do when accessing REST APIs directly. Identification of the game
will be done instead by the headers of every request you send to the servers:
```

```javascript--client
There is no Setup to do when accessing APIs in server side scripts. The context is
already decided by the game calling the script from a client application:
```

Before making any actual request to the servers, you have to give the credentials
of the game you want to access with the SDK. Three parameters are needed to describe
the environment you want to work on.

Parameter | Type | Description
--------- | ---- | -----------
key | string | Unique identifier for your game. It is generated and can be retrieved from your [Account developer page](https://account.clanofthecloud.com).
secret | string | Authentication secret for your game. It is generated and can be retrieved from your [Account developer page](https://account.clanofthecloud.com).
env | string | Used to access the correct environment. Use "sandbox" when developing your game, and "prod" when publishing your game.
