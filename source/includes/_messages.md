# Messages

Currently you can access message feeds from Diagnostic Network as RSS/Atom, embeddable HTML (suitable for dropping into an `iframe` on your website), or on social media like Twitter and Facebook.

## Embeddable HTML

```html
<iframe src="https://diag.net/case-studies?e=1"/>
<iframe src="https://diag.net/t/driveability?e=1"/>
<iframe src="https://diag.net/case-studies/t/driveability?e=1"/>
```

> The above HTML code embeds three streams into iframes: recent Case Studies, recent Driveability discussions, and recent Driveability Case Studies, respectively.

It's easy to embed any message stream from Diagnostic Network on your website, using embeddable feeds. The easiest way is to navigate around DN until you are on the view that you would like to embed, and then add `?e=1` to the URL. If the user is not logged in to Diagnostic Network, this embed will show a "About DN" opening box. If you'd like to hide that, just add `&hd=1` to the querystring. 

### HTTP Request

`GET /?e=1` (all recent discussions)

`GET /{TypeKey}?e=1` (filter by type)

`GET /t/{TopicKey}?e=1` (filter by topic)

`GET /t/{TopicKey}?e=1` (filter by user group)

`GET /{TypeKey}/t/{TopicKey}?e=1` (filter by type and topic)

`GET /{TypeKey}/g/{TopicKey}?e=1` (filter by type and user group)

### URL Parameters

Name     | Description
-------- | -----------
TopicKey | (Optional) Filter messages by topic or user group. If user group, make sure to use `/g/` in the URL instead of the `/t/`. See [Message Topics](#message-topics) for a list of topic TopicKeys. You can also ascertain any User Group's TopicKey by looking at the URL for any [user group](https://diag.net/groups).
TypeKey  | (Optional) Filter messages by message type. See [Message Types](#message-types) for a list of TypeKeys.

## Get Messages

This API endpoint does _not_ require any authentication. But if the request uses [User Authentication](#user-authentication), the results will return more information, e.g. the messages will no longer be anonymized, and full names of members will be displayed. In this scenario, please do **not** publish any of the returned data publicly, to preserve the privacy of our members. Instead, please use the anonymous API results.

Our Messages API will normally return the root message of each discussion happening on Diagnostic Network, not the replies. 

You can use this endpoint in an RSS feed reader (or in Slack using `/feed subscribe {url}`) by specifying `o=atom` or `o=rss` for Atom or RSS output format. We recommend Atom format as it returns the author with the results. (Soon we will publish details on obtaining messages in JSON format, keyword searching, vehicle make filtering, etc.)

### HTTP Request

`GET /api/v1/messages?o=atom`

### URL Querystring Parameters

> NOTE: For any parameters that contain something other than a letter or a number, the value must be URL encoded first.

Name | Description
---- | -----------
o    | **(Required)** Output Format: Currently we support `atom` or `rss` output formats. Atom is recommended if you want to see the author's name.
me    | Returns only discussions created by the current user, requires [User Authentication](#user-authentication).
s    | Sort: `0` (latest messages, the default), `1` (latest bookmarked messages first, requires [User Authentication](#user-authentication) and will only return messages bookmarked by the current user), `2` (highest scoring first, only useful if passing keywords), `3` (for searches filtering down to bounties only, this returns not only the latest bounties, but also the most recently increased bounties), or `4` (oldest messages first).
to   | Topic (or User Group): E.g. `1` for `Autonomous Vehicles`, `2` for `Braking`, and so on. You can specify multiple `to` parameters to filter down to messages matching _any_ of those given topics/groups. For a full breakdown of available types, see [Message Topics](#message-topics). **NOTE:** Soon this parameter will accept topic keys as well, e.g. `braking` instead of `2`.
ty   | Type: E.g. 1 for `Case Studies`, 2 for `Demonstrations`, and so on. You can specify multiple `ty` parameters to filter down to messages matching _any_ of those given types. For a full breakdown of available types, see [Message Types](#message-types).
uk   | UserKey: Filter the stream down to messages written by a specific user. The UserKey is the unique identifier found in a user's profile URL. E.g. Diagnostic Network's brand account `UserKey` is `u3udukosklafng6pp4647hrq1u`.
v    | Vehicle Type Group: `0` (Unknown), `2` (Light Duty), `7` (Powersports), or `8` (Medium/Heavy Duty). E.g. `v=8` would filter down to just medium/heavy duty discussions. By default, we don't constrain the vehicle type group. Currently, Light Duty (`v=2`) are the most commonly discussed vehicles on Diagnostic Network.

[//]: # (k    | Keywords: Filter the results down to matching keywords. This may change over time, but currently the keywords search the following message properties: title, author name, message body, metadata (e.g. symptoms, DTCs, vehicles). Multiple keywords should be separated by spaces (encoded as `%20`). The more keywords supplied, the more constrained your search will be, because every keyword must match. To search for a phrase, wrap the keywords in quotation marks (`%22`). Example: `k=toyota corolla` will match any post containing "toyota" and "corolla", anywhere in our searched message properties, wheras `k="Toyota Corolla"` would only match messages with "toyota" followed directly by "corolla". (Keywords are _not_ case sensitive.))
[//]: # (mk   | Make: Any make name, such as Ford, Toyota, and so on. You can specify multiple `mk` parameters to filter down to messages matching _any_ of those given makes. This field is case sensitive: the make you supply must match the way we spell it in our list of makes on the site. See [Makes](#makes))
[//]: # (nr    | No Replies: If you're searching with the `k` or `uk` parameters, the results will begin to include replies as well. You can prevent this by setting `nr=1` in your request.)

## Message Topics

> NOTE: Eventually we plan to offer an API endpoint to retrieve the current mapping of all message topics and user groups. If you need a User Group TopicId in the meantime, please contact us.

The available topics may change over time. In addition to topics, every user group can also be specified if you know the group's TopicId. 

TopicId | TopicKey           | Name
------- | ------------------ | ------------------
1       | autonomousvehicles | Autonomous Vehicles
2       | braking            | Braking
3       | chassis            | Chassis
4       | collision          | Collision
5       | diagnosticnetwork  | Diagnostic Network
6       | driveability       | Driveability
7       | drivetrain         | Drivetrain
8       | education          | Education
9       | electrical         | Electrical
10      | electrification    | Electrification
11      | emissions          | Emissions
12      | employment         | Employment
14      | fabrication        | Fabrication
46      | heavyduty          | Heavy Duty
15      | hvac               | HVAC
16      | industry           | Industry
102     | infotainment       | Infotainment
17      | management         | Management
18      | marketplace        | Marketplace
19      | motorsports        | Motorsports
45      | networkcomms       | Network Communications
20      | other              | Other
21      | parts              | Parts
22      | powersports        | Powersports
23      | programming        | Programming
24      | propulsion         | Propulsion
25      | restoration        | Restoration
26      | safety             | Safety
27      | security           | Security
28      | tooling            | Tooling


## Message Types

This mapping from `TypeId` to a type name shouldn't change, but the available types will change over time. 

TypeId  | TypeKey       | Name
------- | ------------- | ----------
10      | announcements | Announcements
9       | bounties      | Bounties
1       | case-studies  | Case Studies
2       | demos         | Demonstrations
3       | discussions   | Discussions
7       | events        | Events
4       | questions     | Questions
8       | resources     | Resources (special type that doesn't normally appear in message feeds, except when the user is searching with keywords)
5       | surveys       | Surveys
6       | tech-tips     | Tech Tips


## Social Media Feeds

Every new discussion started on Diagnostic Network that is not posted to a _private_ user group is posted to our Twitter and Facebook feeds:

* [twitter.com/OnDiagNetwork](https://twitter.com/OnDiagNetwork)
* [facebook.com/OnDiagNetwork](https://facebook.com/OnDiagNetwork)
