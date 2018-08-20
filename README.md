
Bittrex.Net is a .Net wrapper for the Bittrex API as described on [Bittrex](https://bittrex.com/Home/Api). It includes all features the API provides using clear and readable C# objects including 
* Reading market info
* Placing and managing orders
* Reading balances and funds

Next to that it adds some convenience features like:
* Access to the SignalR websocket, allowing for realtime updates
* Configurable rate limiting
* Autmatic logging

**If you think something is broken, something is missing or have any questions, please open an [Issue](https://github.com/JKorf/Bittrex.Net/issues)**

---
Also check out my other exchange API wrappers:
<table>
	<tr>
	<td>
		<a href="https://github.com/JKorf/Binance.Net">
			<img src="https://github.com/JKorf/Binance.Net/blob/master/Resources/binance-coin.png?raw=true">
		</a>
		<br />
		<a href="https://github.com/JKorf/Binance.Net">Binance</a>
	</td>
	<td>
		<a href="https://github.com/JKorf/Bitfinex.Net">
			<img src="https://github.com/JKorf/Bitfinex.Net/blob/master/Resources/icon.png?raw=true">
		</a>
		<br />
		<a href="https://github.com/JKorf/Bitfinex.Net">Bitfinex</a>
	</td>
	<td>
		<a href="https://github.com/JKorf/CoinEx.Net">
			<img src="https://github.com/JKorf/CoinEx.Net/blob/master/Resources/icon.png?raw=true">
		</a>
		<br />
		<a href="https://github.com/JKorf/CoinEx.Net">CoinEx</a>
	</td>
	</tr>
</table>

And other API wrappers based on CryptoExchange.Net:
<table>
	<tr>
		<td>
			<a href="https://github.com/Zaliro/Switcheo.Net">
				<img src="https://github.com/Zaliro/Switcheo.Net/blob/master/Resources/switcheo-coin.png?raw=true">
			</a>
			<br />
			<a href="https://github.com/Zaliro/Switcheo.Net">Switcheo</a>
		</td>
	</tr>
</table>


## Installation
![Nuget version](https://img.shields.io/nuget/v/bittrex.net.svg) ![Nuget downloads](https://img.shields.io/nuget/dt/Bittrex.Net.svg)

Available on [NuGet](https://www.nuget.org/packages/Bittrex.Net/):
```
PM> Install-Package Bittrex.Net
```
To get started with Bittrex.Net first you will need to get the library itself. The easiest way to do this is to install the package into your project using  [NuGet](https://www.nuget.org/packages/Bittrex.Net/). Using Visual Studio this can be done in two ways.

### Using the package manager
In Visual Studio right click on your solution and select 'Manage NuGet Packages for solution...'. A screen will appear which initially shows the currently installed packages. In the top bit select 'Browse'. This will let you download net package from the NuGet server. In the search box type 'Bittrex.Net' and hit enter. The Bittrex.Net package should come up in the results. After selecting the package you can then on the right hand side select in which projects in your solution the package should install. After you've selected all project you wish to install and use Bittrex.Net in hit 'Install' and the package will be downloaded and added to you projects.

### Using the package manager console
In Visual Studio in the top menu select 'Tools' -> 'NuGet Package Manager' -> 'Package Manager Console'. This should open up a command line interface. On top of the interface there is a dropdown menu where you can select the Default Project. This is the project that Bittrex.Net will be installed in. After selecting the correct project type  `Install-Package Bittrex.Net`  in the command line interface. This should install the latest version of the package in your project.

After doing either of above steps you should now be ready to actually start using Bittrex.Net.

## Getting started
To get started we have to add the Bittrex.Net namespace:  `using Bittrex.Net;`.

Bittrex.Net provides two clients to interact with the Bittrex API. The  `BittrexClient`  provides all rest API calls. The  `BittrexSocketClient`  provides functions to interact with the SignalR websocket provided by the Bittrex API. Both clients are disposable and as such can be used in a  `using`statement.

Most API methods are available in two flavors, sync and async:
````C#
public void NonAsyncMethod()
{
    using(var client = new BittrexClient())
    {
        var result = client.GetTicker("BTC-ETH");
    }
}

public async Task AsyncMethod()
{
    using(var client = new BittrexClient())
    {
        var result2 = await client.GetTickerAsync("BTC-ETH");
    }
}
````

## Response handling
All API requests will respond with a CallResult object. This object contains whether the call was successful, the data returned from the call and an error if the call wasn't successful. As such, one should always check the Success flag when processing a response.
For example:
````C#
using(var client = new BittrexClient())
{
	var priceResult = client.GetTicker("BTC-ETH");
	if (priceResult.Success)
		Console.WriteLine($"BTC-ETH price: {priceResult.Data.Last}");
	else
		Console.WriteLine($"Error: {priceResult.Error.Message}");
}
````

## Options & Authentication
The default behavior of the clients can be changed by providing options to the constructor, or using the `SetDefaultOptions` before creating a new client. Api credentials can be provided in these options.

## Websockets
The Bittrex API exposes a SignalR Websocket connection for realtime updates. The websocket provides updates regarding the latest prices and trades of all markets, as well as updates for orders and balances for users. Listening to the websocket is easier to implement and less demanding of the Bittrex server than polling using the Rest API.

#### Subscribing
To subscribe to a socket the `SubscribeXXX` methods on the `BittrexSocketClient` can be used:
````C#
    var socketClient = new BittrexSocketClient();
	var subcribtionSuccess = socketClient.SubscribeToMarketSummariesUpdate("BTC-ETH", data =>
	{
		// Handle data
	});
````

#### Unsubscribing
To unsubscribe from the socket the `UnsubscribeFromStream` method in combination with the stream ID received from subscribing can be used. Alternatively, all subscriptions can be unsubscribed on a client using the `UnsubscribeAllStreams` method:
````C#
    var socketClient = new BittrexSocketClient();
	var subcribtionSuccess = socketClient.SubscribeToMarketSummariesUpdate("BTC-ETH", data =>
	{
		// Handle data
	});
	
	socketClient.UnsubscribeFromStream(subcribtionSuccess.Data); // Unsubscribes a single sub
    socketClient.UnsubscribeAllStreams(); // Unsubscribes all subs on this client 
}
````

#### Connection events
If the connection gets lost when it was connected Bittrex.Net will automatically try to reconnect to the websocket. So when the computer that runs the code loses the internet connection or the Bittrex service fails it will keep retrying to connect to the service as long as there are still subscriptions on any client. To be notified of when this happens there are 2 events to which can be listened, the ConnectionLost and ConnectionRestored events. Note that these are on class events, not instance events. This is because internally there is only a single websocket shared over all clients.
````C#
    var socketClient = new BittrexSocketClient();
    socketClient.ConnectionLost += () => 
    {
        Console.WriteLine("Connection lost!");
    };

    socketClient.ConnectionRestored += () => 
    {
        Console.WriteLine("Connection restored after being lost!");
    };
````
