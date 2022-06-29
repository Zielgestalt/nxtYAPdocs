
# Documentation


## Visual Studio code extension

The key porpouse of the SDK is to ease development of blockchain applications. For this reason it is key to have a visual studio extension that can autofill and describe the diffrent tokens, functions and structures that the SDK can do. If an extension is not possible then the SDK library must be written in a way that takes advantage of VS Code's descriptive features.

## Magic 

The magic function is key to the SDK's ease of use of. Users should be able to interact with any DeFi protocol without knowing the specific interface of that protocol. The users focus should only be on the tokens.

The result of this call can be a swap, deposit, withdrawal, lend, borrow or stake depending on the path and path restrictions.

```

from Yieldster import balancer, protocol

numberOfReturnTokens = balancer([{fromAddress: fromChain}], 
            [{sendToken: sendAmount}], 
            [{toAddress: toChain}], 
            [{receiveToken" receiveAmount}], 
            path = protocol.orderBalancer
            residual = min/max, 
            **kwargs) 

```

- fromAddress / fromChain: 
    - The address which the tokens are either held in or where the transaction tokens originate from, the fromAddress can be a Yieldster vault or a wallet address on a specific chain
    - If ommited the from address is any vault or address on any chain that is the most cost effective. For example borrowing can be done on Aave's mainnet or polygon chain
- fromToken / fromAmount: 
    - Name of the token should be autocompleted, and have description of the token in the comments
    - If name is not available then ERC20 token address should work as well.
- Path:
    - Path restriction that should be taken. Command such as "OrderBalancer", "None", "AAVE".
    - If ommited then the path will be the OrderBalancer
- residual
    - With min, minimizing the number of sendTokens sent, and recieve the exact number of receiveTokens, 
    - With max, maximize receiveTokens, while sending all of the send tokens
- ** config: future extendible options, passed in as python kwargs 
- numberOfReturnTokens
    - an array with objects of tokens, and the number of that token which was returned.

## function call and min/max

The following code calls order balancer function. 

```

balancer(sendTokens = {"USDC": 100}, receiveTokens = {"ETH": 0.3})

```

The code below is equivalent to the code above.

``` 

balancer({"self": "Ethereum"}, 
            {"self": "Ethereum"}, 
            {"USDC": 100}, 
            {"ETH": 0.3}, 
            path = protocol.orderBalancer, 
            residual = min) 

```

This function call is automatically categorized as a swap function because the fromAddress and toAddress are both address of the vault. Although the order balancer itself may take actions such as borrow, stake and other to achive the end result.

The "min" command minimizes the number of sendTokens sold, until 0.3 ETH is reached. While with the "max" command all of USDC would be sold to get the greatest amount of ETH. In this scenario the order balancer will minimize the number of USDC/DAI liquidated to purchase exactly 0.3 ETH. 

Number of receiveTokens serves as a limit order. If 0.3 ETH cannot be purchased then the transaction will fail. To make this order similar to a market order it should have a max residual with ETH amount set to 0. (more on this later)


``` 
balancer(sendTokens = [{"USDC": 100}, {"DAI": 50}], 
            receiveTokens = [{"ETH": 0.3}, {"MKR": 3}], 
            min) 
```

The code above shows that multiple tokens can be swapped for multiple tokens. The advantage of this is it provides us with an exponential number of arbitrauge oppertunities when executing the order.

The min command purchases the exact number of sendTokens while the amount of receiveTokens liquidated is minimized. While the max command liquidates all of the fromTokens and purchases as many toTokens as possible.

With the min command, the liquidation will be in proportion to the ratio of USDC/DAI, for every 1 DAI we will liquidate 2 USDC. The ratio of the residual will also be 2:1, i.e. 10 USDC to 5 DAI left after the transaction is complete. 

With the max command, if there is excess purchasing power tokens will be in the ratio of 10:1, i.e. 0.33ETH to 3.3 MKR or 0.6 ETH to 6 MKR.

## Object oriented

The prefered way to call the order balancer is to pass in objects created by Yieldster. This way the developer will be assisted with autocomplete and ensure that they do not type in the wrong token name.

Take the following example

```

balancer([{"0x": "Ethereum", "0x": "Polygon"}, 
            [{"0x": "Ethereum", "0x": "Polygon}, 
            {"USDC": 100}, 
            {"USDC": 100}, 
            path="orderBalancer", 
            min) 

```

The alternative would be below using objects.

```

balancer([self.address.ethereum, self.address.polygon], 
            [self.address.ethereum, self.address.polygon], 
            {token.USDC: 100}, 
            {token.USDC: 100}, 
            path=protocol.orderBalancer, 
            min) 

```


## Virtual funds

An alternative way is to buy and sell against a virtual funds. The advantage of using this system is that it works well with many developers and quants are already familiar with the traditional limit orders. Having them convert over their logic to that of the order balancer can be a challange.

In this scenerio the excess value is kept in form of the sell assets and not in USD or ETH. this similar to the min command, the excess value will be in proportion to the number of tokens sold. 

Consider the following exmaple:

```

from Yieldster import order

order.sell("USDC", 100) 

order.sell("USDT", 300) 

order.buy("ETH", 0.3, chain.polygon) // market order

```

It will have an equavilient execution as the code below

```

balancer({"self": "ethereum"}, 
            {"self": "polygon"}, 
            [{"USDC": 100}, {"USDT": 300}],
            {"ETH": 0.3}, 
            path="orderBalancer", 
            min) 

```

unlike with the balancer, if the 


With the virtual dollar a limit price can also be placed, this is so the entire order is not fulfilled if the price is beyond the limit price.

```

order.sell("DAI", 600, limit = 0.99, chain.polygon) // limit order

order.buy("ETH", 0.2, limit = 3000, chain.polygon) // limit order

```

## Arbitrage

``` 

balancer({"self": "Polygon"}, 
            {"0x": "Polygon"}, 
            {"USDC": 100})

```

The code above sends 100 USDC to another address on the polygon network. During this transfer the orderbalancer will take advantage of arbigrauge oppertunities if one exist. This will reduce the amount of funds withdrawn from the wallet.


``` 

balancer({"self": "Polygon"}, 
            {"self": "Polygon"}, 
            {"USDC": 100}, 
            {"USDC": 100}, 
            path="orderBalancer", 
            min) 

```

As in the code above takes advantage of arbitrage opportunities within a vault.

``` 

balancer([{"self": "Ethereum", "self": "Polygon"}, 
            [{"self": "Ethereum", "self": "Polygon}, 
            {"USDC": 100}, 
            {"USDC": 100}, 
            path="orderBalancer", 
            max) 

```

The above code takes advantage of arbitrage opportunities between different chain. With fromAddress and toAddress set to self and fromChain and toChain set as the same value. This might be for a use who is a long term investor and is agnostic to where the assets are held in the short term.


``` 

from Yieldster import transfer

transfer([{"self": "Ethereum", "self": "Polygon"}, 
            [{"0x": "Ethereum", "0x": "Polygon}, 
            {"USDC": 100}, 
            {"USDC": 100}, 
            min)

```

An alternative syntax is the code above


## Lending

AAVE is a lending platform. You can deposit and withdraw collateral, and against those collateral borrow assets or lend assets.

```
balancer(toAddres = {"Aave": "any"}, fromToken = {"USDC": 100}) 
```

The above code does one of two actions depending on the state of the lending protocol:
- Deposits 100 USDC into AAVE. Keep in mind that funds deposited into AAVE are automatically lent out.
- Pay back loan If the user had borrowed 100 USDC

```
balancer(fromAddress = {"Aave": "any"}, fromToken = {"ETH": 0.3}) 
```

The above code will take one of two actions depending on the state of the lending protocol:
- Withdraw 100 USDC if fund are available
- Borrow 100 USDC if no fund are available


```
balancer(toAddress = {"Aave": "any"}, fromToken = {"USDC": 100}, toToken = {"ETH": 0.3}) 
```

Consider the above code. Assume we are in a state where no deposits or loans exist. This code will first deposit 100USDC into AAVE and then borrow 0.3 ETH

```
balancer(toAddres = {"Aave": "any"}, fromToken = {"USDC": 100}) 

balancer(fromAddress = {"Aave": "any"}, fromToken = {"ETH": 0.3}) 
```

Alternatively these two transactions can be chain together.



```

balancer.Aave.deposit("USDC", 100, PositionObject ) // PositionObject is 7 lines below

balancer.Aave.deposit("USDC", 100) // automatically selects Aave V2 on mainnet

balancer.Aave.withdraw("ETH", 0.3, PositionObject) 

balancer.Aave.withdraw("ETH", 0.3) 

balancer.Aave.positions() -> [
    { // this is a PositionObject
        "vaultId": 1, 
        "chain": "polygon",  
        "assets": [
            {"USDC": 100}
        ], 
        "debts": [
            {"ETH": 0.3}
        ], 
        "margin": 30%
    }
] 

```

An easier syntax might be the code above


## Triggers:

Simple time schedule

```

from Yieldster.triggers import timeSchedule

# Entry function for the SDK
def initilize():
    timeSchedule(rebalance, "Monthly", offSet = 1) # first of every month

def rebalance():
    pass

```

Each vault would also be able to subscribe to web3 events. These events may be events on the vault itself or other smart contracts.

```

from Yieldster.triggers import webRequest, web3Event

def initilize():
    web3Event(depositOnChain, address = '0x', event="deposit(address user, uint amount)")

def depositOnChain(event):
    deposit(event.user, event.amount)

```

Since an http portal must be opened for each vault, we should also allow users to recive web request. In this way we are able to build fully fleged applications on top of the automation platform. 

```

from Yieldster.triggers import webRequest, web3Event

def initilize():
    webRequest(depositDelayed, url = "/deposit")

def depositDeplayed(request):
    deposit(request.user, request.amount)


```

Custom triggers can be created by the community and placed on the marketplace. Each trigger will be running in its own proccess. A vault will subscribe it by calling its function through the triggers library. A trigger is unsubscribed when the advisor shuts down, updates or if it does not respond to the trigger after a timeout period.

```

from Yieldster.triggers import priceDrop
from Yieldster.tokens import ETH

def initilize():
    priceDrop(marginCall, ETH, 0.3)

def marginCall():
    pass

```

## path


```

from Yieldster.path import pathClass

class myPath(pathClass):
    def execute(sendToken, receiveToken):
        # stuff
        # returns EVM bytecode
        return transactionPlan 

    def quote(sendToken, receiveToken):
        return 1
    
    self.sendTokens = []
    self.receiveTokens = []

```

The code above is the template for a path. 

execute is a simple function that takes in one tokens and returns the EVM bytecode on how that can be converted to anther token. 

quote will provide the estimated price range for how much receiveTokens can be expected.

The list of which tokens can be sent and received is registered in the database to be used by the order balancer.

If the path takes advantage of a centralized exchange then the path will purchase the token on the exchange and place them on the blockchain to be withdrawn by the order balancer. 

## Vault

A Yieldster vault listed on the automation platform must implement the following function. The vault classes exposes the appropriate http portal to each function call. For example  "/vault/<id>/deposit" will call the deposit function in the class myVault. 

```

from Yieldster.vault import VaultClass

class myVault(VaultClass):

    withdrawalAssets = [{token:chain}]
    depositAssets = [{token:chain}]
    whiteList = [{address:chain}]

    def deposit(address, amount):
        pass

    def withdrawal(address, amount):
        pass

    def swap():
        pass

    def addWhiteList(address):
        pass

    def removeWhiteList(address):
        pass
    
    isWhitelist(address):
        pass

```

- withdrawalAssets
    - a list of token addresses and their corresponding chain 
- depositAssets
    - a list of token addresses and their corresponding chain 
- whiteList
    - a list of addresses and their corresponding chain 
- deposit
    - returns a bytecode that can be executed by metamask 
- withdrawal
    - returns a bytecode that can be executed by metamask 
    - Alternatively white list group can be added 

```

from Yieldster.vault import vaultClass
from Yieldster.whitelist import accreditedInvestor

class myVault(vaultClass):
    
    isWhitelist(address):
        return accreditedInvestor[address]

```

The following are static variables of the vault

```

from Yieldster.vault import vaultClass

class myVault(vaultClass):

    self.address
    self.address.chainName 
    self.NAV

```

- self.address:
    - An array/object of the vaults addresses on all chains approved by the APS
- self.address.chainName
    - An object with vault address and its corresponding chain
- self.NAV
    - the total value of all assets held in the vault
-

## Data 

The data is always pulled

Have data price split and dividend adjusted, fill forward stale date

```

from Yieldster.data import priceCurrent 

# initilize ...

def rebalance():
    if priceCurrent.ETH() > 2000:
        # buy
    else:
        # sell

```
A series of prices

```

from Yieldster.data import priceSeries

# initilize ...

def rebalance():
    # returns pandas dataframe 
    priceETH =  priceSeries.ETH(timePeriod = "daily") 
    if priceETH[-1] > priceETH[-7]:
        # sell

```

All data in the Yieldster.data class is returned as a pandas object. The class would make a request to the database to get the information required.


## Database

The Key value store records python dictionaries 

recording items

```

from Yieldster.store import get, set

## initialize

def deposit(user, amount):
    balance = get(user + ":balance")
    set(user, balance + amount)

```

Editing the key value store

```

from Yieldster.triggers import webrequest
from Yieldster.store import get, set

def initilize():
    webrequest(depositdelayed, url = "/edit")

def editsore(request):
    set(request.key, request.value) 

```

## secret Keys

A secret key might be an infura token, an ethereum address private key, or any other piece of data. Users are able to upload this key via the command line to the Yieldster automation platform.

```

from Yieldster import secretKeys

// initilize

def dangerFunction():
    return sercretKeys["name"]

```

## web3.py

Users should be able to call the blockchain without having to go through our order balancer. For this porpose, we expose the web3 python library to the user so they can directly call smart contracts. 


```

from web3 import Web3

# initilize 

def getBalance():
    w3 = Web3(Web3.HTTPProvider('http://127.0.0.1:8545')
    balance = w3.eth.getBalance("0x")
    return balance

```

Ideal but not neccery functionality: all smart contract interactions should go through our vault proxy. so a function such as ERC20.send("0x") would send 10 ERC20 tokens from the vault


## logging function

on the online platform there should be a place where users can view their logged data.

```

import logging

logging("message")

```

## backtesting

Not needed in the current version. Users can backtest their strategies by recording diffrent 

```
from Yieldster.backTesting import recordChart

```

## order histor


Not needed in the current version. 

```
from Yieldster.orderHistory import history

```

## commandline

Advisor developers should be able to upload their advisors via the command line. The question is whether we should allow them to upload multiple files via a zip folder, with one file as the entry file or only allow them to upload one file which contains the entire logic. 

```

$ Yieldster upload file.py

$ Yieldster upload folder.zip

```

Users should also be able to set secret keys which they will then be able to use later in the SDK.

```

$ Yieldster upload file.py

$ Yieldster upload folder.zip

```


To test their advisors, exposes the functions on a local host. All of the Yieldster Backend functions would all still remain functional, but we would provide them with a port that simulates rather than executions the order balancer calls.

```

Yieldster run file.py


```


Every sdk has a smart contract. It is either our vault or their custom smart contract. Their smart contract should implement the deposit and withdrawal function, some form of token transformation.

Question for developers:
- Should we limit external ULR request?
- How can the users test the app in their own enviroment. 
- Key value store vs relational database. Allow them to set their own database
- Yieldster.finance should be created as any other Advisor
- Should they have to initilize data sources they are going to use?

Token Documentation: 

We need data helper functions. Pass in data as class with raw data and helper functions 

