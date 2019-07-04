# Transactions

Transactions are used to keep track of any inventory or portfolio or wallet. Its basic functionality can be used in various contexts.
In the case of an inventory, transactions will add or remove units from it. When used as a portfolio (for virtual currencies),
transactions add or remove units too. Transactions are not limited to integers, they can perfectly handle float values.

**How domains work with transactions**<br>
Transactions, portfolios and inventories can be shared between games, like most data supported by XtraLife platform, by using domains.
The *private* domain, the domain by default, is private to the game, but you can create other domains shared between games so your
virtual currencies will be accessible from multiple games. This feature can be used to share a virtual currency between some or all
of your games, or to team up with other game developers to implement cross-marketing or any other incentive.

## Post a transaction

> To post a transaction, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void PostTransaction()
    {
        CHJSON json, transaction;
        // This transaction sells 10 "Gold" from the wallet (negative value), and buys 50 "Silver" (positive value)
        transaction.Put("Gold", -10);
        transaction.Put("Silver", 50);
        json.Put("domain", "private");
        json.Put("description", "Swap of Gold and Silver");
        json.Put("transaction", transaction.Duplicate());
        CloudBuilder::CUserManager::Instance()->TransactionExtended("matchPattern", 10, 0, MakeResultHandler(this, &MyGame::PostTransactionHandler));
    }

    void PostTransactionHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Transaction posted: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not post transaction due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void PostTransaction()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        // This transaction sells 10 "Gold" from the wallet (negative value), and buys 50 "Silver" (positive value)
        Bundle transaction = Bundle.CreateObject("Gold", -10, "Silver", 50);
        currentGamer.Transactions.Domain("private").Post(transaction, "Swap of Gold and Silver")
        .Done(postTransactionRes => {
            Debug.Log("Transaction posted: " + postTransactionRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Failed post transaction: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function PostTransaction()
{
    clan.withGamer(gamer).transactions("private").create({ Gold : -10, Silver : 50},  "Swap of Gold and Silver", function(error, postTransactionRes)
    {
      if(error)
		    ConsoleLog("Post transaction error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Post transaction succeeded: " + JSON.stringify(postTransactionRes));
    });
}
```

```http
POST /v2.2/gamer/tx/{domain}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret

BODY
{
	"transaction" : {
		"Gold" : -10,
		"Silver" : 50
	},
	"description" : "Swap of Gold and Silver"
}
```

This function is used to update the virtual currencies for the user. A transaction can be used to increment or decrement a single currency, for a buy or a sell.
You can also buy several currencies at the same time, just provide a JSON transaction with as many currencies as you wish, with the different values for each
currency. You can also do swaps of currencies if you pass several currencies, and some of them are negative. This way, you can in a single call, buy different
currencies by using another currency (or maybe several). The negative values always decrement a currency (a sell), and positive values always increment a
currency (a buy).
Note that if you sell more currency than you currently have, the transaction will be rejected by the server for insufficient funds.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which to post the transaction
transaction | JSON, required | a JSON object containing the list of currencies to update, and the corresponding amounts (positive to credit, negative to debit)
description | String, optional | a string which can contain anything you wish, can be used to store the context of the transaction process

This function returns a JSON with 2 keys. First key is `balance` and contains a list of all the user virtual currencies, with the current amount for each, and
the second key is `achievements`, which is a list of all the achievements eventually unlocked by the transaction being processed. You can see more about it in
the Achievements section.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "balance": {
    "Bronze": 1000,
    "Gold": 60,
    "Silver": 200
  },
  "achievements": {
    "Silver Earning": {
      "type": "limit",
      "config": {
        "unit": "Silver",
        "maxValue": "200"
      },
      "progress": 1
    }
  }
}
```

<aside class="warning">
Result JSON in case of failure:
</aside>

```json
{
  "name": "BalanceInsufficient",
  "message": "balance is not high enough for transaction"
}
```

## List user transactions

> To list user transactions, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void ListTransactions()
    {
        CHJSON json;
        json.Put("unit", "Silver");
        json.Put("skip", 0);
        json.Put("limit", 4);
        CloudBuilder::CUserManager::Instance()->TxHistory("private", &json, MakeResultHandler(this, &MyGame::ListTransactionsHandler));
    }

    void ListTransactionsHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("List of transactions: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not list transactions due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void ListTransactions()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Transactions.Domain("private").History("Silver", 4, 0)
        .Done(listTransactionsRes => {
            Debug.Log("List of transactions: " + listTransactionsRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not list transactions: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function ListTransactions()
{
    clan.withGamer(gamer).transactions("private").history("Silver", 0, 4, function(error, listTransactionsRes)
    {
      if(error)
		    ConsoleLog("List transaction error: " + JSON.stringify(error));
	    else
		    ConsoleLog("List of transactions: " + JSON.stringify(listTransactionsRes));
    });
}
```

```http
GET /v1/gamer/tx/{domain}?{unit}{skip}{limit}
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to fetch an history of all the transactions made previously by a user in a specific domain. If you do not
fill the unit parameter, than all the transactions will be returned. If a unit is filled, then only transactions involving this currency
will be returned (it will return transactions involving only this currency and multi currencies transactions as well). The total number
of transactions is returned as well, which is not necessarily the number you asked, if you need to use pagination.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to fetch transactions
unit | String, optional | the unit you want to filter transactions with. If not filled, all transactions will be considered
skip | Integer, optional | used to access other pages if there are too many results
limit | Integer, optional | the maximum number of transactions to be returned with this call. If there are more, then call the function again for different pages

This function returns a JSON with the following keys: `servertime` is the current time, `count` is the total number of transactions, not just
the number returned in the JSON, and `history` is a JSON array containing all the transactions returned. Each transaction in turn contain the
following keys: `domain`, `ts` which is the timestamp when the transaction was made, `tx` describing the units and values of the transaction and
`desc`, the information string that was sent at the moment of the transaction. 

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "history": [
    {
      "domain": "com.clanofthecloud.cloudbuilder.azerty",
      "ts": "2017-01-31T12:02:08.718Z",
      "tx": {
        "Gold": -10,
        "Silver": 50
      },
      "desc": "Swap of Gold and Silver"
    },
    {
      "domain": "com.clanofthecloud.cloudbuilder.azerty",
      "ts": "2017-01-31T11:59:01.145Z",
      "tx": {
        "Gold": -10,
        "Silver": 50
      },
      "desc": "Swap of Gold and Silver"
    },
    {
      "domain": "com.clanofthecloud.cloudbuilder.azerty",
      "ts": "2017-01-31T11:58:51.812Z",
      "tx": {
        "Gold": -10,
        "Silver": 50
      },
      "desc": "Swap of Gold and Silver"
    },
    {
      "domain": "com.clanofthecloud.cloudbuilder.azerty",
      "ts": "2017-01-31T08:57:31.865Z",
      "tx": {
        "Gold": -10,
        "Silver": 50
      },
      "desc": "Swap of Gold and Silver"
    }
  ],
  "servertime": "2017-01-31T12:56:36.918Z",
  "count": 7
}
```

## Virtual currencies balance

> To get virtual currencies balance, use this code:

```cpp
#include "CUserManager.h"

class MyGame
{
    void GetBalance()
    {
        CloudBuilder::CUserManager::Instance()->Balance("private", MakeResultHandler(this, &MyGame::GetBalanceHandler));
    }

    void GetBalanceHandler(eErrorCode aErrorCode, const CloudBuilder::CCloudResult *aResult)
    {
        if(aErrorCode == eErrorCode::enNoErr)
        {
            const CHJSON* json = aResult->GetJSON();
            printf("Current balance: %s\n", aResult->GetJSON()->print_formatted().c_str());
        }
        else
            printf("Could not get balance due to error: %d - %s\n", aErrorCode, aResult->GetErrorString());
    }
};
```

```csharp
using CotcSdk;

public class MyClass
{
    void GetBalance()
    {
        // currentGamer is an object retrieved after one of the different Login functions.

        currentGamer.Transactions.Domain("private").Balance()
        .Done(getBalanceRes => {
            Debug.Log("Current balance: " + getBalanceRes.ToString());
        }, ex => {
            // The exception should always be CotcException
            CotcException error = (CotcException)ex;
            Debug.LogError("Could not get balance: " + error.ErrorCode + " (" + error.ErrorInformation + ")");
        });
    }
}
```

```objective_c
```

```javascript--client
var clan; // clan was retrieved previously with a constructor to `Clan`
var gamer; // gamer was retrieved previously with a call to one of the Login methods from `Clan`

function GetBalance()
{
    clan.withGamer(gamer).transactions("private").balance(function(error, getBalanceRes)
    {
      if(error)
		    ConsoleLog("Balance error: " + JSON.stringify(error));
	    else
		    ConsoleLog("Current balance: " + JSON.stringify(getBalanceRes));
    });
}
```

```http
GET /v1/gamer/tx/{domain}/balance
Host: https://sandbox-api01.clanofthecloud.mobi
Content-Type: application/json
x-apikey: YourGameApiKey
x-apisecret:YourGameApiSecret
Authorization: Basic gamer_id:gamer_secret
```

This function is used to fetch the balance of all currencies for a player. For each currency, the current amount is also returned.
This balance is also returned after each call to post a transaction.

Parameter | Type | Description
--------- | ---- | -----------
domain | String, required | the domain in which you want to fetch the user balance of currencies

This function returns a JSON with one key for each currency the player has. The corresponding value is the amount held in this
currency by the user.

---

<aside class="success">
Result JSON in case of success:
</aside>

```json
{
  "Bronze": 1000,
  "Gold": 100,
  "Silver": 550
}
```
