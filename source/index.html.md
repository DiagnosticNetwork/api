---
title: Diagnostic Network API

language_tabs: # must be one of https://git.io/vQNgJ
  - csharp
  - javascript

toc_footers:
  - <a href='https://diag.net/contact'>Contact Diagnostic Network</a>

includes:
  - authentication
  - partners
  - errors

search: true
---

# Diagnostic Network API

### API Documentation 

```csharp
using Newtonsoft.Json;
using System.Net;
using System.Net.Http;
using System.Security.Cryptography;
using System.Text;
using System.Threading.Tasks;

public class DNAPI
{
  // Set this to diag.net for production, or test.diag.net 
  // for testing. Please see our "Testing" section for more 
  // details on test.diag.net.
  public static string BaseHref = "https://diag.net";

  // Cookie handling is needed when getting User Auth cookies
  public static CookieContainer CookieContainer = new CookieContainer();

  // Use a singleton HttpClient for all API requests.
  public static readonly HttpClient Client = new HttpClient(
    new SocketsHttpHandler 
    {
        // Help with DNS handling: https://bit.ly/client-pooling
        PooledConnectionLifetime = TimeSpan.FromMinutes(10),
        PooledConnectionIdleTimeout = TimeSpan.FromMinutes(5),
        // Contact us if you need more than 5 at a time
        MaxConnectionsPerServer = 5,    
        // Avoid redirecting on 201, 303, etc.
        AllowAutoRedirect = false,
        UseCookies = true,
        CookieContainer = CookieContainer
    }
  );
}
```

This page documents the parts of our API which are currently public. Some of these API endpoints are only available to our partners. You can view code examples in the area to the right. We will be exposing more of our API over time. If you have any comments, questions, or requests, please open a [new issue](https://github.com/DiagnosticNetwork/api/issues/new/choose).

### Overview of Diagnostic Network

[Diagnostic Network](https://diag.net) is a community platform to support the vehicle service industry. The primary interface is a continuous message "stream," similar to Facebook, Twitter, and similar social networks, with the difference being each message in the stream is a link to a complete, threaded discussion. 

Each message in the stream is primarily categorized by the message type (e.g. Discussion, Question, Event) and the topics and/or user groups to which the message was posted (e.g. Chassis, Driveability, Joe's User Group). Beyond that, every discussion has a title, an author, and a message body. Optionally, metadata can be attached to messages, usually varying depending on what kind of message it is, and where it's being posted. For example, an _Announcement_ might have no metadata, an _Event_ might have date/time metadata, and a _Question_ might have a vehicle-specific metadata attached.

Most of the discussions take place within the [core topics](https://diag.net/topics), though members of the community can create their own [user groups](https://diag.net/groups), which can be open, closed, or private (hidden). 

The homepage of Diagnostic Network currently features a stream of all non-private discussions, regardless of which topic or group they're in. However, every message type, topic, and user group has its own landing page, featuring a message stream that's filtered down to just the relevant discussions. Further, members are able to subscribe to their preferred topics and user groups, and have the option of showing just their subscription stream on the homepage, rather than the unfiltered view. 

Typically message streams on DN display just the "root message" of each discussion, and to see the replies, you need to click through to view the entire discussion. The two exceptions where replies are also shown in message streams: 

* Search results
* Member profiles

If you're a professional working in or retired from the vehicle service industry, please [join us](https://diag.net/account/register).

