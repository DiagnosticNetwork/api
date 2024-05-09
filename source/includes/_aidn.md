# <img src="https://d3tl0hgcgcslkv.cloudfront.net/img/robot-avatars/aidn.jpg" width="35" height="35" alt="Image" style="float: left; margin-right: 1rem; border-radius: 5px"> AiDN&reg; — AI-powered assistant

AiDN is Diagnostic Network's [award-winning](https://www.prnewswire.com/news-releases/diagnostic-network-receives-prestigious-motor-top-20-tool-award-for-2023-301931391.html) AI-powered virtual robot, designed specifically for assisting with vehicle diagnostics and repair. [First released](https://diag.net/msg/m3h0hjpu4jbpmh197p5f6jv3rj) in June 2023, AiDN responds to most questions posted to [Diagnostic Network](https://diag.net), and is now available for use outside of Diagnostic Network via this API.

> Please [contact us](https://diag.net/contact) to inquire about pricing for AiDN and to obtain API credentials. We can also work with you to customize AiDN's capabilities and response formats to suit your needs.


## Ask AiDN for Help

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
    // `Data` can be an arbitrary piece of data. Typically it is JSON, though we can work with you
    // to accept your specific data formats. It's common to use the `Data` field to pass things 
    // like Symptoms and TroubleCodes (DTCs), as shown below.
    Data = new
    {
      Symptoms = 
      [
        "Crank / No Start",
        "When Cold"
      ],
      TroubleCodes = 
      [
        new 
        {
          Code = "P2837",
          Definition = "Your internal DTC definition, if available" // optional, but helpful
        }
      ]
    }
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
> The Robot API call returns JSON like below when successful. The ResponseKey will be a 
> unique identifier representing this response, currently useful for debugging issues with us.

```json
{
  "Error": null,
  "ShouldRetry": false,
  "Message": "This is where the response from AiDN will be. It will typically be formatted using Markdown (e.g. **bold**, _italics_, etc.). However, we can work with you to return the response in a different format if desired.",
  "ResponseKey": "r6rxd67venhswfmfxiq7ito6p5"
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
