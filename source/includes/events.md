# Events

XtraLife lets you send and receive messages between gamers. You can use this feature for inter-application messaging too,
and use it to send any kind of json message. Ordered delivery is guaranteed, and acknowledgement (or auto-acknowledgement)
of each message is required. Events are optionally associated to push notifications in the case the recipient is offline.

## Send an event

> To send an event, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void SendEvent()
    {
        XtraLife::Helpers::CHJSON json;

        json.Put("message", "Here is a gift to you!");
        json.Put("currency", 100);
        XtraLife::CUserManager::Instance()->PushEvent("private", "587f5d844877a1734ec079e6", &json, XtraLife::MakeResultHandler(this, &MyGame::SendEventDone));
    }

    void SendEventDone(XtraLife::eErrorCode aErrorCode, const XtraLife::CCloudResult *aResult)
    {
        if(aErrorCode == XtraLife::eErrorCode::enNoErr)
        {
            const XtraLife::Helpers::CHJSON* json = aResult->GetJSON();
            printf("Event sent: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not send event due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void SendEvent()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        Bundle evt = Bundle.CreateObject("message", "Here is a gift to you!", "currency", 100);
        currentGamer.Community.Domain("private").SendEvent("587f5d844877a1734ec079e6", evt)
           .Done(sendEventRes => {
               Debug.Log("Event sent: " + sendEventRes.ToString());
           }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not send event: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
#import "XLGamer.h"

void SendEvent()
{
    // gamer is a XLGamer instance obtained by a call to one of the Login methods

    NSDictionary* event = [NSDictionary dictionaryWithObjectsAndKeys:@"Here is a gift to you!", @"message",
                        [NSNumber numberWithInteger: 100], @"currency", nil];
    [gamer sendEvent:event toGamer:@"587f5d844877a1734ec079e6" inDomain:@"private"
            completionHandler:^(NSError *error, NSInteger statusCode, NSDictionary *res) {
        if(error == nil)
        {
            NSLog(@"Event sent: %@", [res description]);
        }
        else
        {
            NSLog(@"Could not send event: %@", [error description]);
        }
    }];
    
}
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function SendEvent()
{
    const evt = { message : "Here is a gift to you!", currency : 100 };
    clan.withGamer(gamer).events("private").send("587f5d844877a1734ec079e6", evt, null, function(error, sendEventRes)
    {
      if(error)
		    ConsoleLog("Could not send event: " + JSON.stringify(error));
	    else
		    ConsoleLog("Event sent: " + JSON.stringify(sendEventRes));
    });
}
```

```http
POST /v1/gamer/event/{domain}/{gamer_id} HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
    "message" : "Here is a gift to you!",
    "currency" : 100
}
```

```javascript--server
function __SendEvent(params, customData, mod) {
    "use strict";
    // don't edit above this line // must be on line 3
  
    const evt = { message : "Here is a gift to you!", currency : 100 };
    return this.game.sendEvent(this.game.getPrivateDomain(), mod.ObjectID("587f5d844877a1734ec079e6"), evt)
    .then(res => {
        mod.debug("Event sent: " + JSON.stringify(res));
        return res;
    })
    .catch(error => {
  	    throw new Error("Could not send event: " + error);
    });
} // must be on last line, no CR
```

This function is used to send a message to another user. It can be any user in the game, no need to be a friend to be able
to exchange messages. The content of the message is completely free form, you can put anything you want in the JSON which
will be sent. It's up to the developer to format the different JSONs according to their needs, for every kind of message
that users can exchange (1 to 1 message, sending of lives or virtual currencies, ...).

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to send the event
gamer_id | String, required | the user who will receive the event
event | JSON, required | a free form JSON containing any kind of data

This function returns a JSON based on the one you sent. The original JSON is embedded in an `event` key. Some fields are added including
`type` with value "user", `from`, the identifier of the sender, `to`, the identifier of the recipient, `name` which is the display name
of the sender, as well as a `id` key whose value is an identifier generated by XtraLife (optionally used for "acking" reception of the message).

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "type" : "user",
  "event" : {
      "message": "Here is a gift to you!",
    "currency": 100
  },
  "from" : "587f5d844877a1734ec079e6",
  "to" : "541ad1f01cfd7f984741f9b1",
  "name" : "John Doe",
  "id": "ff423836-9874-4c8e-a879-d0d6ff8a79d8"
}
```

## Receive messages

> To receive messages, use this code:

```cpp
#include "CUserManager.h"

struct EventHandler : XtraLife::CEventListener
{
    // Function called every time a message is received
    virtual void onEventReceived(const char *aDomain, const XtraLife::CCloudResult *aEvent)
    {
        printf("Received event on domain %s: %s", aDomain, aEvent->GetJSON()->printFormatted().c_str());
    }
    
    virtual void onEventError(XtraLife::eErrorCode aErrorCode, const char *aDomain, const XtraLife::CCloudResult *result)
    {
        printf("Received error %d on domain %s: %s", aErrorCode, aDomain, result->GetJSON()->printFormatted().c_str());
    }
};

class MyGame
{
    // Before being able to receive messages, you have to register a handler which will process the incoming messages. This can be
    // done as soon as the user has been authenticated
    void SetEventHandler()
    {
        EventHandler* handler = new EventHandler;
        CloudBuilder::CUserManager::Instance()->RegisterEventListener("private", handler);
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    // Function called every time a message is received
    private void EventHandler(DomainEventLoop sender, EventLoopArgs e)
    {
        Debug.Log("Received event of type " + e.Message.Type + ": " + e.Message.ToJson());
    }

    // Before being able to receive messages, you have to register a handler which will process the incoming messages. This can be
    // done as soon as the user has been authenticated
    public void SetEventHandler()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        DomainEventLoop loop;
        loop = currentGamer.StartEventLoop();
        loop.ReceivedEvent += EventHandler;
    }
}
```

```objective_c
#import "XLGamer.h"

@interface MyClass : NSObject<EventLoopDelegate>
- (void) SetEventHandler;
@end

@implementation Game

- (bool) onReceivedEvent:(NSDictionary*)event forDomain:(NSString *)domain {
    NSLog(@"Received event for domain %@: %@", domain, event);

    return true;
}

- (void) SetEventHandler {
    // gamer is a XLGamer instance obtained by a call to one of the Login methods

    gamer.delegate = self;
}

@end
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`

function listenForEvent()
{
    clan.withGamer(gamer).events("private").receive("auto", function(error, listenForEventRes)
    {
      if(error)
		    ConsoleLog("Timeout reached: " + JSON.stringify(error));
	    else
		    ConsoleLog("Got event: " + JSON.stringify(listenForEventRes));
    });
}
```

```http
GET /v1/gamer/event/{domain}?ack=auto&timeout=60000 HTTP/1.1
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
```

This function is uded to receive messages which are sent to the user.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to receive the events
ack | String, required | used to acknowledge the latest message identifier, better to use `auto` for simplified process
timeout | Int, required | the number of milliseconds, before returning a timeout

<aside class="notice">
The different SDKs will handle the timeout automatically for you. You do not have to manage it. Instead, just register your event handler,
and it will be called as soon as a message is ready to be processed. Managing the timeout by yourself is necessary if you use javascript
or the REST APIs directly. In that case, you will receive a `204` error meaning no messages were available during the lapse of time you
provided, and then you can submit again a request.
</aside>
