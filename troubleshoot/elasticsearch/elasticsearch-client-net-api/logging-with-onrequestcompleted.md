---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/logging-with-on-request-completed.html
---

# Logging with OnRequestCompleted [logging-with-on-request-completed]

When constructing the connection settings to pass to the client, you can pass a callback of type `Action<IApiCallDetails>` to the `OnRequestCompleted` method that can eavesdrop every time a response(good or bad) is received.

If you have complex logging needs this is a good place to add that in since you have access to both the request and response details.

In this example, we’ll use `OnRequestCompleted` on connection settings to increment a counter each time it’s called.

```csharp
var counter = 0;
var client = new ElasticClient(new AlwaysInMemoryConnectionSettings().OnRequestCompleted(r => counter++)); <1>

client.RootNodeInfo(); <2>
counter.Should().Be(1);

await client.RootNodeInfoAsync(); <3>
counter.Should().Be(2);
```

1. Construct a client
2. Make a synchronous call and assert the counter is incremented
3. Make an asynchronous call and assert the counter is incremented


`OnRequestCompleted` is called even when an exception is thrown, so it can be used even if the client is configured to throw exceptions

```csharp
var counter = 0;
var client = FixedResponseClient.Create( <1>
    new { },
    500,
    connectionSettings => connectionSettings
        .ThrowExceptions() <2>
        .OnRequestCompleted(r => counter++)
);

Assert.Throws<TransportException>(() => client.RootNodeInfo()); <3>
counter.Should().Be(1);

await Assert.ThrowsAsync<TransportException>(async () => await client.RootNodeInfoAsync());
counter.Should().Be(2);
```

1. Configure a client with a connection that **always returns a HTTP 500 response
2. Always throw exceptions when a call results in an exception
3. Assert an exception is thrown and the counter is incremented


Here’s an example using `OnRequestCompleted()` for more complex logging

::::{note} 
By default, the client writes directly to the request stream and deserializes directly from the response stream.

If you would also like to capture the request and/or response bytes, you also need to set `.DisableDirectStreaming()` to `true`.

::::


```csharp
var list = new List<string>();
var connectionPool = new SingleNodeConnectionPool(new Uri("http://localhost:9200"));

var settings = new ConnectionSettings(connectionPool, new InMemoryConnection()) <1>
    .DefaultIndex("default-index")
    .DisableDirectStreaming() <2>
    .OnRequestCompleted(apiCallDetails => <3>
    {
        // log out the request and the request body, if one exists for the type of request
        if (apiCallDetails.RequestBodyInBytes != null)
        {
            list.Add(
                $"{apiCallDetails.HttpMethod} {apiCallDetails.Uri} " +
                $"{Encoding.UTF8.GetString(apiCallDetails.RequestBodyInBytes)}");
        }
        else
        {
            list.Add($"{apiCallDetails.HttpMethod} {apiCallDetails.Uri}");
        }

        // log out the response and the response body, if one exists for the type of response
        if (apiCallDetails.ResponseBodyInBytes != null)
        {
            list.Add($"Status: {apiCallDetails.HttpStatusCode}" +
                     $"{Encoding.UTF8.GetString(apiCallDetails.ResponseBodyInBytes)}");
        }
        else
        {
            list.Add($"Status: {apiCallDetails.HttpStatusCode}");
        }
    });

var client = new ElasticClient(settings);

var syncResponse = client.Search<object>(s => s <4>
    .AllIndices()
    .Scroll("2m")
    .Sort(ss => ss
        .Ascending(SortSpecialField.DocumentIndexOrder)
    )
);

list.Count.Should().Be(2);

var asyncResponse = await client.SearchAsync<object>(s => s <5>
    .AllIndices()
    .Scroll("10m")
    .Sort(ss => ss
        .Ascending(SortSpecialField.DocumentIndexOrder)
    )
);

list.Count.Should().Be(4);
list.Should().BeEquivalentTo(new[] <6>
{
    @"POST http://localhost:9200/_all/_search?typed_keys=true&scroll=2m {""sort"":[{""_doc"":{""order"":""asc""}}]}",
    @"Status: 200",
    @"POST http://localhost:9200/_all/_search?typed_keys=true&scroll=10m {""sort"":[{""_doc"":{""order"":""asc""}}]}",
    @"Status: 200"
});
```

1. Here we use `InMemoryConnection` but in a real application, you’d use an `IConnection` that *actually* sends the request, such as `HttpConnection`
2. Disable direct streaming so we can capture the request and response bytes
3. Perform some action when a request completes. Here, we’re just adding to a list, but in your application you may be logging to a file.
4. Make a synchronous call
5. Make an asynchronous call
6. Assert the list contains the contents written in the delegate passed to `OnRequestCompleted`


When running an application in production, you probably don’t want to disable direct streaming for *all* requests, since doing so will incur a performance overhead, due to buffering request and response bytes in memory. It can however be useful to capture requests and responses in an ad-hoc fashion, perhaps to troubleshoot an issue in production.

`DisableDirectStreaming` can be enabled on a *per-request* basis for this purpose. In using this feature, it is possible to configure a general logging mechanism in `OnRequestCompleted` and log out request and responses only when necessary

```csharp
var list = new List<string>();
var connectionPool = new SingleNodeConnectionPool(new Uri("http://localhost:9200"));

var settings = new ConnectionSettings(connectionPool, new InMemoryConnection())
    .DefaultIndex("default-index")
    .OnRequestCompleted(apiCallDetails =>
    {
        // log out the request and the request body, if one exists for the type of request
        if (apiCallDetails.RequestBodyInBytes != null)
        {
            list.Add(
                $"{apiCallDetails.HttpMethod} {apiCallDetails.Uri} " +
                $"{Encoding.UTF8.GetString(apiCallDetails.RequestBodyInBytes)}");
        }
        else
        {
            list.Add($"{apiCallDetails.HttpMethod} {apiCallDetails.Uri}");
        }

        // log out the response and the response body, if one exists for the type of response
        if (apiCallDetails.ResponseBodyInBytes != null)
        {
            list.Add($"Status: {apiCallDetails.HttpStatusCode}" +
                     $"{Encoding.UTF8.GetString(apiCallDetails.ResponseBodyInBytes)}");
        }
        else
        {
            list.Add($"Status: {apiCallDetails.HttpStatusCode}");
        }
    });

var client = new ElasticClient(settings);

var syncResponse = client.Search<object>(s => s <1>
    .AllIndices()
    .Scroll("2m")
    .Sort(ss => ss
        .Ascending(SortSpecialField.DocumentIndexOrder)
    )
);

list.Count.Should().Be(2);

var asyncResponse = await client.SearchAsync<object>(s => s <2>
    .RequestConfiguration(r => r
        .DisableDirectStreaming()
    )
    .AllIndices()
    .Scroll("10m")
    .Sort(ss => ss
        .Ascending(SortSpecialField.DocumentIndexOrder)
    )
);

list.Count.Should().Be(4);
list.Should().BeEquivalentTo(new[]
{
    @"POST http://localhost:9200/_all/_search?typed_keys=true&scroll=2m", <3>
    @"Status: 200",
    @"POST http://localhost:9200/_all/_search?typed_keys=true&scroll=10m {""sort"":[{""_doc"":{""order"":""asc""}}]}", <4>
    @"Status: 200"
});
```

1. Make a synchronous call where the request and response bytes will not be buffered
2. Make an asynchronous call where `DisableDirectStreaming()` is enabled
3. Only the method and url for the first request is captured
4. the body of the second request is captured


