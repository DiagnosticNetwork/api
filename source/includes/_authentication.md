# Authentication

```csharp
using Newtonsoft.Json;
using System.Net;
using System.Net.Http;
using System.Security.Cryptography;
using System.Text;
using System.Threading.Tasks;

public class DNAPI
{
  static DNAPI()
  {
    // Set this to diag.net for production, or test.diag.net for testing
    // Please see our "Test Access" section for more details on test.diag.net.
    Client.BaseAddress = new Uri("https://test.diag.net");
  }

  // Cookie handling is needed when getting User Auth cookies
  public static CookieContainer CookieContainer = new CookieContainer();

  // Use a singleton HttpClient for all API requests.
  public static readonly HttpClient Client = new HttpClient(
    new SocketsHttpHandler 
    {
      PooledConnectionLifetime = TimeSpan.FromMinutes(10),   // Help with DNS handling:
      PooledConnectionIdleTimeout = TimeSpan.FromMinutes(5), // https://bit.ly/client-pooling
      MaxConnectionsPerServer = 5,  // Contact us if you need more than 5 at a time  
      AllowAutoRedirect = false,    // Avoid redirecting on 201, 303, etc.
      UseCookies = true,
      CookieContainer = CookieContainer
    }
  );
}
```

Some API endpoints are only available for our partners, using [_Partner Authentication_](#partner-authentication). Other API endpoints require [_User Authentication_](#user-authentication), the same kind of authentication users experience directly on our website. And finally, some API endpoints, like our "Get Messages" endpoint, require no authentication, but may work differently when accessed by an authenticated client (e.g. anonymized messages when not authenticated, versus complete messages with full details for authenticated users).

## Test Access

```csharp
// Set the test platform cookie to allow access
await DNAPI.Client.GetAsync(MAGIC_LINK);
```

```javascript
// Set the test platform cookie to allow access
$.get(MAGIC_LINK);
```

We _highly_ recommend testing your API access against our test environment at `test.diag.net`. In order to access this site, you must pass a special cookie with each request. The easiest way to set this cookie is to hit a special URL that can be provided to you. Please contact us for the `MAGIC_LINK` URL used in the code sample here.


## Partner Authentication

```csharp
private static ASCIIEncoding AsciiEncoding = new ASCIIEncoding();

// These next two values are provided by DN to partners
private static int PartnerId = PARTNER_ID;
private static byte[] PartnerApiKey = AsciiEncoding.GetBytes("PARTNER_API_KEY");

/// Sends requests to the partner API that are authenticated using the
/// `PartnerId` and `PartnerApiKey` specified above.
public async static Task<HttpResponseMessage> 
PartnerSendAsync(HttpMethod method, string path, dynamic json)
{
  using (var hmacsha256 = new HMACSHA256(PartnerApiKey)) 
  {
    var requestMessage = new HttpRequestMessage {
      RequestUri = new Uri(DNAPI.Client.BaseAddress, path),
      Method = method,
      Content = new StringContent(
        JsonConvert.SerializeObject(json),
        Encoding.UTF8, 
        "application/json"
      )
    };
    requestMessage.Headers.Add("Authentication", $"hmac " + 
      PartnerId + ":" + 
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

For Partner API functionality, we provide a `PartnerId` and a `PartnerApiKey` that are used to sign requests. Partners can [contact us](https://diag.net/contact) to obtain these, and learn more about what features are authorized for your partnership level.



## User Authentication

```csharp
var response = await DNAPI.Client.PostAsync
(
  "/api/v1/user/login",
  new StringContent(
    JsonConvert.SerializeObject(new {
      Email = "jane-work@example.com",
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
        DNAPI.Client.BaseAddress, 
        "/api/v1/user/login"
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
$.ajax({
  url: "https://diag.net/api/v1/user/login",
  type: "POST",
  contentType: "application/json",
  data: JSON.stringify({
    "Email": "jane-work@example.com",
    "Password": "Jane'sPassword",
    "RememberMe": true
  })
})
.done(function(data) {
  // The User Authentication cookie is now set for future
  // ajax calls. You can access the user's unique identifier 
  // with `data.UserProfile.UserKey`.
});
```

Other API endpoints require _User Authentication_, i.e. the same `dnid` cookie that we set when users login to our website directly. This cookie can be obtained by hitting the `POST /api/v1/user/login` endpoint. The response will also include a `UserProfile` object which contains some additional information about the user as well as the `UserKey` that uniquely identifies the user, and a `dnid` cookie set in the `Cookies` header. That same `dnid` cookie must be passed with any API request that requires _User Authentication_.

<aside class="notice">
If you don't yet have a membership in Diagnostic Network, you can <a href="https://diag.net/account/register">create one here</a>.
</aside>
