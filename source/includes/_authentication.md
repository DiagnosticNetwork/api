# Authentication

Some API endpoints are only available for our partners, using [_Partner Authentication_](#partner-authentication). Other API endpoints require [_User Authentication_](#user-authentication), the same kind of authentication users experience directly on our website. And finally, some API endpoints, like our "Get Messages" endpoint, require no authentication, but may work differently when accessed by an authenticated client (e.g. anonymized messages when not authenticated, versus complete messages with full details for authenticated users).


## Partner Authentication

```csharp
/// Sends requests to the partner API that are authenticated using the
/// `PartnerApiId` and `PartnerApiKey` specified above.
public async static Task<HttpResponseMessage> 
PartnerSendAsync(HttpMethod method, string path, dynamic json)
{
  using (var hmacsha256 = new HMACSHA256(PartnerApiKey)) 
  {
    var requestMessage = new HttpRequestMessage {
      RequestUri = new Uri($"{DNAPI.BaseHref}{path}"),
      Method = method,
      Content = new StringContent(
        JsonConvert.SerializeObject(json),
        Encoding.UTF8, 
        "application/json"
      )
    };
    requestMessage.Headers.Add("Authentication", $"hmac " + 
      PartnerApiId + ":" + 
      Convert.ToBase64String(
        hmacsha256.ComputeHash(
          AsciiEncoding.GetBytes($"{method}+{path}")
        )
      )
    );
    return await DNAPI.Client.SendAsync(requestMessage);
  }
}
```

For Partner API functionality, we provide a `PartnerApiId` and a `PartnerApiKey` that are used to sign requests. Partners can [contact us](https://diag.net/contact) to obtain these, and learn more about what features are authorized for your partnership level.



## User Authentication

```csharp
var response = await DNAPI.Client.PostAsync
(
  $"{DNAPI.BaseHref}/api/v1/user/login",
  new StringContent(
    JsonConvert.SerializeObject(new {
        Email = "janedoe@example.com",
        Password = "Jane'sPassword",
        RememberMe = true
    }),
    Encoding.UTF8, 
    "application/json"
  )
);

if (response.StatusCode == HttpStatusCode.OK)
{
    // Access the user's unique identifier with data["UserKey"]
    var data = Newtonsoft.Json.Linq.JObject.Parse(
      await response.Content.ReadAsStringAsync()
    );

    // If using our cookie handling code in DNAPI above, the cookie is
    // now set and will be passed on subsequent API calls automatically.
    // But if you need the value to manually pass the auth cookie later, 
    // you can retrieve it from the cookie handler like this:
    var authCookie = DNAPI.CookieContainer.GetCookies(
      new Uri(
        $"{DNAPI.BaseHref}/api/v1/user/login"
      )
    )["dnid"];
}
else if (response.StatusCode == HttpStatusCode.Redirect)
{
  // Redirects normally happen for bad passwords or incomplete accounts.
  // Best option is to redirect the user to the given Location: header URL
  // so they can resolve the problem directly.
}
```

```javascript
jQuery.ajax({
    url: "https://diag.net/api/v1/user/login",
    type: "POST",
    contentType: "application/json",
    data: JSON.stringify({
        "Email": "janedoe@example.com",
        "Password": "Jane'sPassword",
        "RememberMe": true
    })
})
.done(function(data) {
    // The User Authentication cookie is now set for future
    // ajax calls. You can access the user's unique identifier 
    // with `data.UserKey`.
});
```

Other API endpoints require _User Authentication_, i.e. the same `dnid` cookie that we set when users login to our website directly. This cookie can be obtained by hitting the `POST /api/v1/user/login` endpoint. The response will also include the `UserKey` that uniquely identifies the user, as well as the `dnid` cookie set in the `Cookies` header. That same `dnid` cookie must be passed with any API request that requires _User Authentication_.

<aside class="notice">
If you don't yet have a membership in Diagnostic Network, you can <a href="https://diag.net/account/register">create one here</a>.
</aside>
