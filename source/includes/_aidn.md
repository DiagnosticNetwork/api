# <img src="https://d3tl0hgcgcslkv.cloudfront.net/img/robot-avatars/aidn.jpg" width="35" height="35" alt="Image" style="float: left; margin-right: 0.5rem; border-radius: 5px; vertical-align: top; position: relative; top: -5px;">AiDN&reg; — AI-powered assistant

```csharp
var response = await DNAPI.PartnerSendAsync
(
  HttpMethod.Post,
  "/api/v1/robot",
  new 
  {
    Vin = "JF2SKAWCXKH400000",                     // Optional, but helpful
    Vehicle = "2019 Subaru Forester 2.5L",         // Highly recommended, in case VIN decoding fails
    Classification = "{ClassificationIdentifier}", // we'll coordinate with you on this value
    UserIdentifier = "{UserIdentifier}",           // uniquely identify the customer/employee/user
    Message = "{YourMessageToAiDN}",               // this is typically a user-supplied question
    DiscussionKey = "mxxxxxxxxxxxxxxxxxxxxxxxxx",  // Only provide this if replying to an existing conversation
    ResponseType = {ResponseType},                 // An integer representing the feature flags (see docs)
    // Metadata is an optional field that can be useful for passing DTCs and Symptoms, e.g.:
    Metadata = new
    {
      Symptoms = 
      [
        new 
        {
          Name = "Crank / No Start"
        },
        new 
        {
          Name = "When Cold"
        }
      ],
      TroubleCodes = 
      [
        new 
        {
          Code = "P2837",
          Definition = "Your internal DTC definition, if available" // optional, but helpful
        }
      ]
    },
    // `Data` is a placeholder for situations where you need to pass arbitrary data.
    // In that scenario, we would work with you to coordinate the format of this data.
    Data = (object) null
);

var data = Newtonsoft.Json.Linq.JObject.Parse(
  await response.Content.ReadAsStringAsync()
);

if (response.StatusCode == HttpStatusCode.OK)
{
  Console.WriteLine("AiDN: " + data["Message"]);
}
else if (!string.IsNullOrEmpty(data["Error"])) 
{
  if (data["ShouldRetry"]?.ToObject<bool>() ?? false)
  {
    // A temporary error occurred, retry the same request.
    Console.WriteLine("Retry your request later: " + data["Error"]);
  }
  else
  {
    // A permanent error occurred, do NOT retry the same request.
    Console.WriteLine(data["Error"]);
  }
}
else
{
  // An unknown error occurred. If it's a 4xx response (other than 
  // 429 Too Many Requests, recommend changing the request before retrying).
}
  if ()
  {
    // Since `ShouldRetry` is true, it's telling us the error is temporary, and
    // you can retry your request exactly as you first submitted it. 
    //
    // data["Error"] will be provided here and will explain the underlying problem.
  }
  else
  {
    // If the HTTP status code of the response is 4xx, then there's either a problem
    // with your request (400 Bad Request), and you likely need to fix something before
    // retrying. (The exception is a 429 Too Many Requests error, however in that case
    // ShouldRetry would be `true`.)
    //
    // If the error is a 5xx error, it indicates a server error, which should be 
    // temporary, and you can retry your request (preferably with an exponential back-off).
  }
}
```

```javascript
$.ajax({
  url: '/api/v1/robot',
  type: 'POST',
  contentType: 'application/json',
  data: JSON.stringify
  (
    {
      Vin: "JF2SKAWCXKH400000",                     // Optional, but helpful
      Vehicle: "2019 Subaru Forester 2.5L",         // Highly recommended, in case VIN decoding fails
      Classification: "{ClassificationIdentifier}", // We'll coordinate with you on this value
      UserIdentifier: "{UserIdentifier}",           // Uniquely identify the customer/employee/user
      Message: "{YourMessageToAiDN}",               // This is typically a user-supplied question
      DiscussionKey: "mxxxxxxxxxxxxxxxxxxxxxxxxx",  // Only provide this if replying to an existing conversation
      Metadata:
      {
        Symptoms: 
        [
          {
            Name: "Crank / No Start"
          },
          {
            Name: "When Cold"
          }
        ],
        TroubleCodes: 
        [
          {
            Code: "P2837",
            Definition: "Your internal DTC definition, if available" // optional, but helpful
          }
        ]
      },
      // `Data` is a placeholder for situations where you need to pass arbitrary data.
      // In that scenario, we would work with you to coordinate the format of this data.
      Data = (object) null
    }
  ),
  success: function (data, textStatus, jqXHR) {
    if (jqXHR.status === 200) {
      console.log("AiDN: " + data.Message);
    }
  },
  error: function (jqXHR, textStatus, errorThrown) {
    var data = JSON.parse(jqXHR.responseText);
    if (data.Error && data.Error !== '') {
      if (data.ShouldRetry) {
        console.log("Retry your request later: " + data.Error);
      } else {
        console.log(data.Error);
      }
    } else {
      if (jqXHR.status >= 400 && jqXHR.status < 500 && jqXHR.status !== 429) {
        console.log("There's a problem with your request, please check before retrying.");
      } else if (jqXHR.status >= 500) {
        console.log("Server error, try again later.");
      }
    }
  }
});
```

> The Robot API call returns JSON like below when successful. The ResponseKey will be a 
> unique identifier representing this response, currently useful for debugging issues with us.

```json
{
  "Error": null,
  "ShouldRetry": false,
  "Message": "This is where the response from AiDN will be. It will typically be formatted using Markdown (e.g. **bold**, _italics_, etc.). However, we can work with you to return the response in a different format if desired.",
  "DiscussionKey": "mxxxxxxxxxxxxxxxxxxxxxxxxx",
  "DiscussionLink": "https://...",
  "ResponseKey": "rxxxxxxxxxxxxxxxxxxxxxxxxx",
  "ResponseLink": "https://..."
}
```

> When there's an error, it may be like the one shown below, indicating you _should_ retry
> your request without any changes. We recommend using expontential back-off, or (if present) 
> the delay specified in the [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) header in our response.

```json
{
  "Error": "Request timed out.",
  "ShouldRetry": true,
  "Message": null,
  "ResponseKey": null
}
```

> When `ShouldRetry` is false, it indicates there is a problem with your request, which should
> be fixed prior to retrying the request. 

```json
{
  "Error": "The request was too large, consider abbreviating your request.",
  "ShouldRetry": false,
  "Message": null,
  "ResponseKey": null
}
```

AiDN is Diagnostic Network's [award-winning](https://www.prnewswire.com/news-releases/diagnostic-network-receives-prestigious-motor-top-20-tool-award-for-2023-301931391.html) AI-powered virtual robot, designed specifically for assisting with vehicle diagnostics and repair. [First released](https://diag.net/msg/m3h0hjpu4jbpmh197p5f6jv3rj) in June 2023, AiDN responds to most questions posted to [Diagnostic Network](https://diag.net), and is now available for use outside of Diagnostic Network via this API.

NOTE: You will need to use [Partner Authentication](#partner-authentication) when accessing the AiDN API. Please [contact us](https://diag.net/contact) to inquire about pricing for AiDN and to obtain API credentials. We can also work with you to customize AiDN's capabilities and response formats to suit your needs.

### ResponseType Flags

When calling the AiDN API, there are a few different types of responses. You control which one you want using the `ResponseType` parameter, which is an integer of bitwise feature flags. (Not all feature flags are enabled for all partners. Please discuss with us first which you anticipate needing so that we may activate them.)

  - **(`0`) Default**: AiDN responds directly, including its response in the API response back to the caller once the response is completely ready. This is the simplest method and is useful for a one-off response. 

  - **(`1`) Reserved**: This flag is reserved for future use.

  - **(`2`) Chat**: AiDN starts (or continues) a chat conversation, responding to the user's initial message. In addition to the `Message` response that you receive, you'll also receive a `DiscussionKey`, `ResponseKey`, and `DiscussionLink`. To continue the conversation further, post additional `Chat` requests to the API, being sure to pass the `DiscussionKey` in subsequent requests so that AiDN will understand the full context and can continue the conversation where it left off. `ResponseKey` is a unique identifier for AiDN's response, and can be useful for debugging purposes or your own local storage.

  - **(`4`) Reserved**: This flag is reserved for future use.

  - **(`8`) ReturnImmediately**: Normally the API will wait to return a response until AiDN has completely responded. If you set this flag on (e.g. call `ResponseType = 10` for a chat with an immediate response), then AiDN returns as soon as possible with a `DiscussionLink` to the newly created discussion, and no `Message`. 

### Chat UI

In addition to providing you the ability to chat with AiDN via our API, we also have the capability for you to leave the entire chat UI to us. You would first need to work with us to enable this feature, but once enabled, you can specify ResponseType: 10 to start a chat with an immediate response, and then load the `DiscussionLink` in a web view for your end user. They can then chat directly with AiDN through our embeddable chat interface. 

[Contact us](https://diag.net/contact) to learn more.
