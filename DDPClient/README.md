# DDPClient-NET [![Build status](https://ci.appveyor.com/api/projects/status/ykk4v2vd33j4wqc1?svg=true)](https://ci.appveyor.com/project/JohnnyCrazy/ddpclient-net) [![NuGet](https://badge.fury.io/nu/DDPClient-NET.svg)](https://badge.fury.io/nu/DDPClient-NET)
A Client for Meteor's DDP-Protocol written in C#/.NET

##WIP
This library is currently Work-In-Progress. Further Improvements, documentation + Unit-Tests will follow soon.

##Usage
###Connecting
```csharp
static void Main(string [] args)
{
    //URL to your Meteor location
    _client = new DdpConnection();
    _client.Login += OnLogin;
    _client.Connected += OnConnected;
    _client.Connect("localhost:3000");

    Console.ReadKey();
    _client.Close(); //Close when we're done
}

private static void OnConnected(object sender, ConnectResponse connectResponse)
{
    if(connectResponse.DidFail()) //We're using a not supported DDP Version
        Console.WriteLine("Connecting Failed, Server wants Version: " + connectResponse.Failed.Version);

    //The client will save this ID and use it for reconnection attempts automatically
    Console.WriteLine("Connected! Our Session-ID: " + connectResponse.Session);
}

private static void OnLogin(object sender, LoginResponse loginResponse)
{}
```

###Login
```csharp

private static void OnConnected(object sender, ConnectResponse connectResponse)
{
    //Should check if successful


    _client.LoginWithEmail("test@test.de", "password");
    //or
    _client.LoginWithUsername("username", "password");
    //or
    _client.LoginWithToken("token");    
}

private static void Login(object sender, LoginResponse loginResponse)
{
    if(loginResponse.HasError())
        Console.WriteLine(loginResponse.Error.Error);
    Console.WriteLine("Token: " + loginResponse.Token);
    Console.WriteLine("Token expires In:" + loginResponse.TokenExpires.DateTime);

    //You can save the token and use it later with LoginWithToken (until TokenExpires)
}
```

###Methods
####No Return-Type
```csharp
private static void OnConnected(object sender, ConnectResponse connectResponse)
{
    //Should check if successful

     _client.Call("test"); //No parameter
     _client.Call("test", 5); //Single parameter
     _client.Call("test", 5, false, new Task()); //Multiple parameters
}
```
####Dynamic Return-Type
```csharp
private static void OnConnected(object sender, ConnectResponse connectResponse)
{
    //Should check if successful

    _client.Call("test", (response) =>
    {
        if(response.HasError())
            return; //Print Error...
        Console.WriteLine(response.Result); //Result is dynamic
        //or
        Console.WriteLine(response.Get<Task>().Name); //Convert it to some type (Json.NET)
    });
}
```
####Fixed Return-Type
```csharp
private static void OnConnected(object sender, ConnectResponse connectResponse)
{
    //Should check if successful

    _client.Call<Task>("test", (error, result) =>
    {
        if(error != null)
            return; //Print Error...
        Console.WriteLine(result.Name); //strongly typed
    });
}
```

###Collections/Subscribers
**NOTE:** Make sure you have a listening Subscriber **before** you subscribe to the actual Data, else incoming data could get lost.

####Event-Based
```csharp
static void Main(string [] args)
{
    //After connecting...

    DdpSubscriber<Task> subscriber = _client.GetSubscriber<Task>("task");
    subscriber.Added += (o, addedModel) => Console.WriteLine("Added: " + addedModel.Object.Name);
    subscriber.Removed += (o, removedModel) => Console.WriteLine("Removed ID: " + removedModel.Id);

    //Should close here
}
```
####Interface-Based
```csharp
static void Main(string [] args)
{
    //After connecting...

    DdpSubscriber<Task> subscriber = _client.GetSubscriber<Task>("task");
    subscriber.Subscribers.Add(new TaskSubscriber());
    //Should close here
}

internal class TaskSubscriber : IDdpSubscriber<Task>
{
    public void Added(SubAddedModel<Task> added)
    {

    }
    //...
}
```

###Subscriptions
```csharp
private static void OnConnected(object sender, ConnectResponse connectResponse)
{
    //Should check if successful

    DdpSubHandler subHandler = _client.GetSubHandler("testSub");
    subHandler.Ready += (o, args) => Console.WriteLine("Sub Ready");
    subHandler.Sub();

    //If you dont want any data: subHandler.Unsub();
}
```
