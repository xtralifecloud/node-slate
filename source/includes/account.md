# Account

The easiest way to create a profile is to use the Anonymous one, since there is no interaction for the
player. Players do not always want to attach games to their social networks and generally prefer to wait
and trust the game before doing so.
XtraLife allows you to convert an anonymous profile at any time. This way, the player benefits from
being able to connect on several devices, and all the data he has accumulated is not lost once he
converts his profile.
A player can also link several social networks to his profile in order to aggregate friends in
XtraLife database. While a profile can be linked to several social networks, it will always remain
primilarily attached to a "master" social network.

## Convert

> To convert an anonymous user to a network, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void Convert()
    {
      CotCHelpers::CHJSON json;

      json.Put("network", "email");
      json.Put("id", "myEmail@gmail.com");
      json.Put("secret", "myPassword");
      CloudBuilder::CUserManager::Instance()->Convert(&json, MakeResultHandler(this, &MyGame::ConvertHandler));
    }

    void ConvertHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        //  Conversion succeeded.
        if(aErrorCode == eErrorCode::enNoErr)
        {
            //  From the result, we get the embedded JSON, which could be displayed with a call to result->printFormatted();
            const CHJSON* json = aResult->GetJSON();
            //  Inside this JSON, we can display the network id used for conversion.
            std::string id = json->GetString("network_id");
        }
        else
            printf("Could not convert account due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void Convert()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Account.Convert(LoginNetwork.Email, "myEmail@gmail.com", "myPassword")
        .Done(convertRes => {
            Debug.Log("Convert succeeded: " + convertRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Failed to convert: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function Convert()
{
    clan.withGamer(gamer).convertTo("email", "myEmail@gmail.com", "myPassword", function(error, convertRes)
    {
      if(error)
		    ConsoleLog("Convert error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Convert succeeded: " + JSON.stringify(convertRes));
    });
}
```

```http
POST /v1/gamer/convert
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
  "network": "email",
  "id" : "myEmail@gmail.com",
  "secret" : "myPassword"
}
```

This function is used to convert an anonymous profile to a profile linked to a social network. The main advantage
is it will be easier when done to connect on another device with the same profile. The internal `gamer_id` and`
`gamer_secret` are not modified, so they can still be used if stored locally on a device.

Parameter | Type | Description
--------- | ---- | -----------
network | String, required | can be any of ["email", "facebook", "googleplus", "gamecenter"]
id | String, required | the user ID for this network
secret | String, required | the user secret/token for this network

<aside class="warning">
In the C++ SDK, you have to use "facebookId", "googleplusId" and "gamecenterId".
</aside>


This function returns a JSON with 2 keys. If the operation is a success, first key is `done` with
value `1`, and second key is `gamer` with value the same JSON as returned by one of the Login functions.
If the operation is a failure, the JSON returned contains the keys `name` and `message`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
	"done": 1,
	"gamer": {
		"network": "email",
		"networkid": "myEmail@gmail.com",
		"registerTime": "2017-01-16T13:10:01.857Z",
		"registerBy": "com.clanofthecloud.cloudbuilder",
		"games": [{
				"appid": "com.clanofthecloud.cloudbuilder",
				"ts": "2017-01-16T13:10:01.857Z",
				"lastlogin": "2017-01-16T00:00:00.000Z"
			}, {
				"appid": "cloud.xtralife.connect4",
				"ts": "2017-01-16T13:14:29.271Z",
				"lastlogin": "2017-01-16T00:00:00.000Z"
			}],
		"profile": {
			"email": "myEmail@gmail.com",
			"displayName": "myEmail",
			"lang": "en"
		},
		"devices": [{
				"id": "5A2D6EC6-21C8-47AA-BC74-95DB1D6DFDAD",
				"osname": "iOS",
				"osversion": "10.2",
				"name": "Michael’s MacBook Air",
				"model": "x86_64",
				"version": "1",
				"ts": "2017-01-16T13:14:29.274Z",
				"ip": "127.0.0.1"
			}],
		"gamer_id": "587cc6294877a1734ec07931",
		"gamer_secret": "0e1297e35769e783a2eaff21f787d127b3777bdf",
		"servertime": "2017-01-16T13:55:27.735Z"
	},
	"_error": 0,
	"_httpcode": 200
}
```
<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "Error",
  "message" : "UserExists"
}
```

## Link

> To link a profile to a social network, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void Link()
    {
      CotCHelpers::CHJSON json;

      json.Put("network", "facebookId");
      json.Put("id", "myFacebookID");
      json.Put("secret", "myFacebookToken");
      CloudBuilder::CUserManager::Instance()->Link(&json, MakeResultHandler(this, &MyGame::LinkHandler));
    }

    void LinkHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
            printf("Link succeeded\n");
        else
            printf("Link failed due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void Link()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Account.Link(LoginNetwork.Facebook, "myFacebookID", "myFacebookToken")
        .Done(linkRes => {
            Debug.Log("Link succeeded: " + linkRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Failed to link: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function Link()
{
    clan.withGamer(gamer).link("facebook", "myFacebookID", "myFacebookToken", function(error, linkRes)
    {
      if(error)
		    ConsoleLog("Link error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Link succeeded: " + JSON.stringify(linkRes));
    });
}
```

```http
POST /v1/gamer/link
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
  "network": "facebook",
  "id" : "myFacebookID",
  "secret" : "myFacebookToken"
}
```

This function is used to associate one, or several if called several times, social network. Doing this
enables for example to match friends from different social networks with profiles in the XtraLife database,
letting the developer aggregate friends in a single database.
 
Parameter | Type | Description
--------- | ---- | -----------
network | String, required | can be any of ["facebook", "googleplus", "gamecenter"]
id | String, required | the user ID for this network
secret | String, required | the user secret/token for this network

<aside class="warning">
In the C++ SDK, you have to use "facebookId", "googleplusId" and "gamecenterId".
</aside>

If the operation is a success, it returns a JSON with a single key, `done` with
value `1`.
If the operation is a failure, the JSON returned contains the keys `name` and `message`.

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
  "name" : "Error",
  "message" : "Already linked to facebook"
}
```

## Unlink

> To unlink a profile to a social network, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void Unlink()
    {
      CloudBuilder::CUserManager::Instance()->Unlink("facebookId", MakeResultHandler(this, &MyGame::UnlinkHandler));
    }

    void UnlinkHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
            printf("Unlink succeeded\n");
        else
            printf("Unlink failed due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void Unlink()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Account.Unlink(LoginNetwork.Facebook)
        .Done(unlinkRes => {
            Debug.Log("Unlink succeeded: " + unlinkRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Failed to unlink: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function Unlink()
{
    clan.withGamer(gamer).unlink("facebook", function(error, unlinkRes)
    {
      if(error)
		    ConsoleLog("Unlink error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Unlink succeeded: " + JSON.stringify(unlinkRes));
    });
}
```

```http
POST /v1/gamer/unlink
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
  "network": "facebook"
}
```

This function is used to remove the association between a profile and a social network.
 
Parameter | Type | Description
--------- | ---- | -----------
network | String, required | can be any of ["facebook", "googleplus", "gamecenter"]

<aside class="warning">
In the C++ SDK, you have to use "facebookId", "googleplusId" and "gamecenterId".
</aside>

If the operation is a success, it returns a JSON with a single key, `done` with
value `1`.
If the operation is a failure, the JSON returned contains the keys `name` and `message`.

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
  "name" : "Error",
  "message" : "Not linked to facebook"
}
```

## Change e-mail address

> To change e-mail address, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void ChangeEmail()
    {
      CloudBuilder::CUserManager::Instance()->ChangeEmail("myNewEmail@gmail.com", MakeResultHandler(this, &MyGame::ChangeEmailHandler));
    }

    void ChangeEmailHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
            printf("Change e-mail succeeded\n");
        else
            printf("Change e-mail failed due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void ChangeEmail()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Account.ChangeEmailAddress("myNewEmail@gmail.com")
        .Done(changeEmailRes => {
            Debug.Log("Change e-mail succeeded: " + changeEmailRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Failed to change e-mail: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function ChangeEmail()
{
    clan.withGamer(gamer).changeEmail("myNewEmail@gmail.com", function(error, changeEmailRes)
    {
      if(error)
		    ConsoleLog("Change e-mail error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Change e-mail succeeded: " + JSON.stringify(changeEmailRes));
    });
}
```

```http
POST /v1/gamer/email
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
  "email": "myNewEmail@gmail.com"
}
```

This function is used to modify the e-mail address of a profile. It will modify the credentials used
to log in. It will not modify the e-mail address in the `profile` section of the user.
 
<aside class="notice">
This can only be applied to a profile with social network `email`.
</aside>

Parameter | Type | Description
--------- | ---- | -----------
newEmailAddress | String, required | The new e-mail which should be used to log in again with this profile

If the operation is a success, it returns a JSON with a single key, `done` with
value `1`.
If the operation is a failure, the JSON returned contains the keys `name` and `message`.

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
  "name" : "InvalidLoginTokenError",
  "message" : "The received login token is invalid"
}
```

## Change password

> To change password, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void ChangePassword()
    {
      CloudBuilder::CUserManager::Instance()->ChangePassword("myNewPassword", MakeResultHandler(this, &MyGame::ChangePasswordHandler));
    }

    void ChangePasswordHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
            printf("Change password succeeded\n");
        else
            printf("Change ChangePassword failed due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void ChangePassword()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Account.ChangePassword("myNewPassword")
        .Done(changePasswordRes => {
            Debug.Log("Change password succeeded: " + changePasswordRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Failed to change password: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
void ChangePassword()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods
    
    [gamer changePassword:@"newpassword" completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *changePasswordRes) {
        if(error == nil)
        {
            NSLog(@"Change password succeeded: %@", [changePasswordRes description]);
        }
        else
        {
            NSLog(@"Failed to change password: %@", [changePasswordRes description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function ChangePassword()
{
    clan.withGamer(gamer).changePassword("myNewPassword", function(error, changePasswordRes)
    {
      if(error)
		    ConsoleLog("Change e-mail error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Change e-mail succeeded: " + JSON.stringify(changePasswordRes));
    });
}
```

```http
POST /v1/gamer/password
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
  "password": "myNewPassword"
}
```

This function is used to modify the password of a profile. After changing the password, the user will
not be able to log in again with the former password. The new one which was sent needs to be used.
 
<aside class="notice">
This can only be applied to a profile with social network `email`.
</aside>

 
Parameter | Type | Description
--------- | ---- | -----------
newPassword | String, required | The new password which should be used to log in again with this profile

If the operation is a success, it returns a JSON with a single key, `done` with
value `1`.
If the operation is a failure, the JSON returned contains the keys `name` and `message`.

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
  "name" : "InvalidLoginTokenError",
  "message" : "The received login token is invalid"
}
```

## Fetch outline

> To fetch outline, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void FetchOutline()
    {
      CloudBuilder::CUserManager::Instance()->Outline(MakeResultHandler(this, &MyGame::OutlineHandler));
    }

    void OutlineHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
            printf("Outline: %s\n", aResult->GetJSON()->printFormatted().c_str());
        else
            printf("Outline failed due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void FetchOutline()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Profile.Outline()
        .Done(outlineRes => {
            Debug.Log("Outline: " + outlineRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Outline failed due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function FetchOutline()
{
    clan.withGamer(gamer).outline(function(error, outlineRes)
    {
      if(error)
		    ConsoleLog("Outline failed due to error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Outline: " + JSON.stringify(outlineRes));
    });
}
```

```http
GET /v1/gamer/outline
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function fetches an outline of the currently logged in user. Basically returns all available
data about the user, including all domains he has been playing on. This can be used to avoid issuing
multiple requests on startup (one for the profile, games, etc.).
Non exhaustive list of fields include: `network`, `networkid`, `networksecret`, `registertime`,
`games`(array), `profile`, `devices` (array), `domains` (array).
 
Parameter | Type | Description
--------- | ---- | -----------

If the operation is a success, it returns a JSON with a single key, `outline` with value
a JSON containing all the fields described above.
If the operation is a failure, the JSON returned contains the keys `name` and `message`.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "outline": {
    "network": "email",
    "networkid": "michael.elbaki.bis@laposte.net",
    "registerTime": "2017-01-09T14:41:27.423Z",
    "registerBy": "com.clanofthecloud.cloudbuilder",
    "games": [
      {
        "appid": "com.clanofthecloud.cloudbuilder",
        "ts": "2017-01-09T14:41:27.423Z",
        "lastlogin": "2017-01-18T00:00:00.000Z"
      }
    ],
    "profile": {
      "email": "michael_elbaki@laposte.net",
      "displayName": "michael_elbaki",
      "lang": "en"
    },
    "domains": [
      {
        "domain": "private",
        "fs": {
          "Test": "toto"
        }
      }
    ],
    "servertime": "2017-01-18T11:04:25.314Z"
  }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name" : "InvalidLoginTokenError",
  "message" : "The received login token is invalid"
}
```

## Reset password with e-mail

> To reset password, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void ResetPassword()
    {
      CotCHelpers::CHJSON json;

      json.Put("email", "myEmail@gmail.com");
      json.Put("from", "support@myCompany.com");
      json.Put("title", "Reset your password");
      json.Put("body", "You can login with this shortcode: [[SHORTCODE]]");
      CloudBuilder::CUserManager::Instance()->MailPassword(&json, MakeResultHandler(this, &MyGame::MailPasswordHandler));
    }

    void MailPasswordHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
            printf("Short code sent\n");
        else
            printf("Short code sending failed due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```cs
using CotcSdk;

public class MyClass
{
    void ResetPassword()
    {
        // cloud is an object retrieved at the beginning of the game through the CotcGameObject object.

        string email = "myEmail@gmail.com";
        string from = "support@myCompany.com";
        string title = "Reset your password";
        string body = "You can login with this shortcode: [[SHORTCODE]]";
        cloud.SendResetPasswordEmail(email, from, title, body)
        .Done(resetPasswordRes => {
            Debug.Log("Short code sent");
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Short code sending failed due to error: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objectivec
void ResetPassword()
{
    XLGamer* gamer = [[XLGamer alloc] init];
    NSString* body = @"You can login with this shortcode: [[SHORTCODE]]";
    NSString* html = @"You can login with this shortcode: <b>[[SHORTCODE]]</b>";

    [gamer mailShortCode:@"myEmail@gmail.com" from:@"support@xtralife.cloud" withTitle:@"Password from XtraLife" withBody:body withHTML:html completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *resetPasswordRes) {
        if(error == nil)
        {
            NSLog(@"Short code sent: %@", [resetPasswordRes description]);
        }
        else
        {
            NSLog(@"Short code sending error: %@", [resetPasswordRes description]);
        }
    }];
}
```

```javascript
var clan; // clan was retrieved previously with a constructor to `Clan`

function ResetPassword()
{
    var email = "myEmail@gmail.com";
    var from = "support@myCompany.com";
    var title = "Reset your password";
    var body = { body: "You can login with this <b>[[SHORTCODE]]</b>", html: true };
    clan.sendResetMailPassword(email, from, title, body, function(error, resetPasswordRes)
    {
      if(error)
		    ConsoleLog("Short code sending failed due to error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Short code sent: " + JSON.stringify(outlineRes));
    });
}
```

```http
POST /v1/login/myEmail@gmail.com
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret

BODY
{
    "from" : "support@myCompany.com",
    "title" : "Reset your password",
    "body" : "You can login with this shortcode: [[SHORTCODE]]"
    "html" : "You can login with this shortcode: <b>[[SHORTCODE]]</b>"
}
```

This function can be used to send an e-mail to a user registered with `email` network in order to
help him recover his/her password. The user will receive an e-mail containing a short code. This
short code can be used in the LoginWithShortCode method.
 
<aside class="notice">
To effectively pass the new shortcode generated, you need to insert the tag [[SHORTCODE]] inside
the body of the e-mail. This string will be matched when next logging in with a short code, not
using any credentials.
</aside>

Parameter | Type | Description
--------- | ---- | -----------
email | String, required | the e-mail of the user who needs to reset his password
from | String, required | the e-mail sender
title | String, required | the title of the e-mail being sent
body | String, required | the body of the e-mail being sent
html | String, optional | a body in HTML format

If the operation is a success, it returns a JSON with a single key, `done` with value 1.

---
@
<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "done": 1
}
```
