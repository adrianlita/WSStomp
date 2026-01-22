# WSStomp

STOMP over WebSockets in C#

## Intention

WSStomp was originally developed as a library for using STOMP over WebSockets in C#.

The main intent was to be used with the SpringBoot SimpleBroker, but STOMP 1.2 functionality was further enhanced.

## Installation

Install WSStomp via NuGet.

## Usage

```aiignore
static async Task Main(string[] args)
        {
            var uri = new Uri("wss://....");
            Console.WriteLine("Connecting to " + uri);

            var stomp = new WsStomp();
            stomp.StompOnConnected += OnConnected;
            stomp.StompOnConnectFailed += OnDisconnected;
            stomp.StompOnDisconnected += OnDisconnected;
            stomp.StompOnError += OnError;
            stomp.SetStompHeader("Authorization", "...");   // if needed
            await stomp.ConnectAsync(uri);
           
            Console.WriteLine("Connected");
            
            await stomp.SubscribeAsync("/topic/topic1", TestCallback);
            var serviceSubscription = await stomp.SubscribeAsync("/topic/topic2", TestCallback2);
            
            await stomp.SubscribeAsync("/topic/topic3", PhoneCallback);
            Console.WriteLine("Subscribed");
            
            Console.ReadLine();

            Console.WriteLine("Sending...");
            await stomp.SendAsync("/app/path", "");
            
            Console.ReadLine();
            await serviceSubscription.Unsubscribe();

            Console.ReadLine();
        }

        static void OnConnected(object sender, StompConnected e)
        {
            Console.WriteLine($"Connected from OnConnected with session {e.Session}");
        }

        static void OnDisconnected(object sender)
        {
            var stomp = (WsStomp) sender;
            
            Console.WriteLine("Disconnected from OnDisconnected");
            
            while (stomp.Connected == false)
            {
                Console.WriteLine("Reconnecting...");
                Task.Delay(1000).Wait();
                stomp.ReconnectAsync().Wait();
            }

        }
        
        static void OnError(object sender, StompError e)
        {
            Console.WriteLine("Error from OnError");
        }
        
        static Task TestCallback(StompMessage message)
        {
            Console.WriteLine($"Message received on subscription {message.Subscription.GetId()} / {message.Subscription.GetDestination()}:");
            Console.WriteLine(message.Body);
            return Task.CompletedTask;
        }
        
        static Task TestCallback2(StompMessage message)
        {
            Console.WriteLine($"Just another callback function on subscription {message.Subscription.GetId()} / {message.Subscription.GetDestination()}:");
            Console.WriteLine(message.Body);
            return Task.CompletedTask;
        }
```