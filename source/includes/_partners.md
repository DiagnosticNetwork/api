# Partner Customers

> Partner Customer objects look like this:

```json
{
  "PartnerCustomerKey": "p0123456789abcdefghijklmno",
  "PartnerRemoteId": "customer-unique-id-from-partner",
  "Email": "jane-work@example.com",
  "EmailAlt": "jane-personal@example.com",
  "Name": "Jane Doe",
  "BusinessName": "Jane's Auto Repair",
  "BusinessStreet": "123 Main Street",
  "BusinessCity": "Los Angeles",
  "BusinessStateCode": "CA",
  "BusinessCountryCode": "US",
  "BusinessPostalCode": "90210",
  "BusinessCounty": "Los Angeles County",
  "BusinessLatitude": null,
  "BusinessLongitude": null,
  "BusinessPhone": "+18005551234",
  "MobilePhone": "+13105555678",
  "DateCreated": "2018-05-03T08:35:55.6492475+00:00",
  "DateLinked": "2018-05-03T08:35:55.6492475+00:00",
  "GroupInvitations": [
    "group-one",
    "group-two",
    "group-three"
  ],
  "Metadata": {
    "Anything": {
      "ThePartner": "wants",
      "SuchAs": 12345,
      "OrEven": true
    }
  }
}
```

Some partnerships allow for creating user accounts on Diagnostic Network, on behalf of their customers, which we refer to as “partner customers.” You may also use this API to link your existing customers to their existing account on Diagnostic Network, linked using their email address. One reason you may want to do this is to make it easier for your customers to gain access to user groups you maintain on Diagnostic Network, which can be done as part of the Partner Customer API.

Once created, a member of DN that is linked to a partner with a Partner Customer record may receive additional benefits from the partner, such as having access to closed and private user groups, and we may allow users to display these partner relationships on their user profile on DN, creating additional awareness of these partners. 

### Email Notifications

If you link or unlink an _existing_ DN member to you (via the Partner Customer `POST`, `PATCH`, or `DELETE` endpoints), we will notify them of the change to their account status via email. When you create a Partner Customer as a _real_ user (setting `CreateUser` to `true`, and without setting `NotifyUser` to `false`), we'll send the user an email that lets them know about their new account. This is recommended, as it is the easiest way for the user to create the password for their new account. You can even specify a custom message we'll show above our messaging. Example:

![Email Notification Example](/images/partnercustomer-email.png)

You can tell whether or not a user has been sent a notification email due to your API call by referring to the `UserNotified` property in the responses.

Aside from those situations, we will _not_ send any communication to Partner Customers that have no prior relationship with us, nor will we share their email address with any third parties just as we protect our own member information according to our [privacy policy](https://diag.net/privacy).

## Create a Partner Customer

```csharp
var response = await DNAPI.PartnerSendAsync
(
  HttpMethod.Post,
  "/api/v1/partner/customer",
  new 
  {
    PartnerRemoteId = "customer-unique-id-from-partner",
    Email = "jane-work@example.com",
    EmailAlt = "jane-personal@example.com",
    Name = "Jane Doe",
    BusinessName = "Jane's Auto Repair",
    BusinessStreet = "123 Main Street",
    BusinessCity = "Los Angeles",
    BusinessStateCode = "CA",
    BusinessCountryCode = "US",
    BusinessPostalCode = "90210",
    BusinessCounty = "Los Angeles County",
    BusinessLatitude = (string) null,
    BusinessLongitude = (string) null,
    BusinessPhone = "+18005551234",
    MobilePhone = "+13105555678",
    // if you're inviting the user to your group(s)
    GroupInvitations = new List<int> {
      "group-one",
      "group-two",
      "group-three"
    },
    Metadata = new {
      Anything = new {
        ThePartner = "wants",
        SuchAs = 12345,
        OrEven = true
      }
    },
    CreateUser = true, // or `false` for stub user
    NotifyUser = true, // (recommended) notify the user
    Introduction = "We look forward to seeing you on DN!"
  }
);

if (response.StatusCode == HttpStatusCode.Created ||
    response.StatusCode == HttpStatusCode.SeeOther) // Previously Created
{
    // Access things like data["PartnerCustomerKey"]
    var data = Newtonsoft.Json.Linq.JObject.Parse(
      await response.Content.ReadAsStringAsync()
    );
}
```

> The create customer API call returns JSON like this.
> (Note the `PartnerCustomerKey`, which you will want to store.)

```json
{
  "PartnerCustomer": {
    "PartnerCustomerKey": "p0123456789abcdefghijklmno",
    "PartnerRemoteId": "customer-unique-id-from-partner",
    "Email": "jane-work@example.com",
    "EmailAlt": "jane-personal@example.com",
    "Name": "Jane Doe",
    "BusinessName": "Jane's Auto Repair",
    "BusinessStreet": "123 Main Street",
    "BusinessCity": "Los Angeles",
    "BusinessStateCode": "CA",
    "BusinessCountryCode": "US",
    "BusinessPostalCode": "90210",
    "BusinessCounty": "Los Angeles County",
    "BusinessLatitude": null,
    "BusinessLongitude": null,
    "BusinessPhone": "+18005551234",
    "MobilePhone": "+13105555678",
    "DateCreated": "2018-05-03T08:35:55.6492475+00:00",
    "DateLinked": "2018-05-03T08:35:55.6492475+00:00",
    "GroupInvitations": [
      "group-one",
      "group-two",
      "group-three"
    ],
    "Metadata": {
      "Anything": {
        "ThePartner": "wants",
        "SuchAs": 12345,
        "OrEven": true
      }
    }
  },
  "UserNotified": true,
  "GroupInvitationErrors: []
}
```

As a partner, with this API you can create either **stub** or **real** user accounts on Diagnostic Network. In either scenario, you must have **permission from your customer** to share their data with Diagnostic Network. Our [privacy policy](https://diag.net/privacy) explains how we operate, but to put it bluntly, we don't spam our users and we don't allow others to spam our users, and this includes the user data provided by partners. 

> Example PartnerCustomerKey: `p0123456789abcdefghijklmno`

These user accounts, once created, are explicitly linked to the partner, and have an associated unique `PartnerCustomerKey` associated with them, which begins with `p` and is 26 characters long (lowercase letters and numbers). This `PartnerCustomerKey` can be used later to update the Partner Customer, sever the relationship with the partner, or to update the relationship (e.g. to enable/disable certain partner-specific features). In addition, you can (and should) specify your own unique identifier for this customer, especially if you anticipate that Diagnostic Network may be making requests against _your_ API in the future.

A **stub user** acts as a reservation. The user visits a special link to create their password and complete their registration, but most of the information should be filled in for them based on what the partner shared with us, making registration quick and simple. In this scenario, we **will not** contact your customer, unless they are already a member of DN.

Conversely, a **real user** can be created by a partner, on behalf of their customer, in the following limited circumstance:

 1. The partner has confirmed that their customer agrees to abide by the Diagnostic Network [Terms of Service](https://diag.net/terms) and makes them aware of our [Privacy & Cookie Policies](https://diag.net/privacy), both of which must be linked for the customer.
Create Account. 
 2. The partner must have confirmed that their customer is a professional in the vehicle service industry, or has a reasonable expectation that they are.

If an API request is made to create a user, and we indicate the user was successfully created, the partner should inform their customer that they need to click a link in an email we've sent them to complete their registration on DN and set their password with us.

For both types of user creation, an implicit link is created between the partner and the user, which can later be severed using the `DELETE partner/customer` endpoint. If the Partner Customer already existed, a `303 See Other` will be returned, otherwise `201 Created`.


### HTTP Request

`POST /api/v1/partner/customer`

### POST JSON Fields

Fields               | Type            | Description
-------------------- | --------------- | -----------
CreateUser           | `boolean`       | If true, create a real user; otherwise (the default), a stub user.
NotifyUser           | `boolean`       | If true (the default), send an email notification to the user, which is recommended as it allows us to give them a link to complete their DN account. If you've invited them to any groups using `GroupInvitations`, these will also be included in this email.
Introduction         | `string`        | (Optional) If set, we'll display this HTML message from you in the email we send to the user, at the top of the email, with your partner name signed at the end. If the account you're creating is not being done in realtime at your customer's request, we highly recommend using this to explain to your customer why the account was created. 
Email                | `string`        | The email address of the user.
EmailAlt             | `string`        | (Optional) An alternate email address of the user.
Name                 | `string`        | The full name (must contain at least two name parts).
BusinessName         | `string`        | The business name.
BusinessStreet       | `string`        | The business street address.
BusinessCity         | `string`        | The business city.
BusinessStateCode    | `string`        | The business state (ISO standard 2-letter code).
BusinessCountryCode  | `string`        | The business country (ISO standard 2-letter code).
BusinessPhone        | `string`        | The business phone number (ideally E.164 format)
MobilePhone          | `string`        | The user's mobile phone number (ideally E.164 format)
PartnerRemoteId      | `string`        | (Recommended) Unique identifier for this customer, supplied by the partner, which can be used in the future for API requests that originate _from_ Diagnostic Network to you.
BusinessPostalCode   | `string`        | (Optional) The business zip/postal code.
BusinessCounty       | `string`        | (Optional) The business county (not country)
BusinessLatitude     | `decimal`       | (Optional) The business latitude coordinates.
BusinessLongitude    | `decimal`       | (Optional) The business longitude coordinates.
GroupInvitations     | `array<string>` | (Optional) Array of your user groups to which this user will have access. 
Metadata             | `json`          | (Optional) Any additional data DN or the partner needs associated with this customer, perhaps to unlock other features on DN.

## Get a Partner Customer

```csharp
var response = await DNAPI.PartnerSendAsync
(
  HttpMethod.Get,
  $"/api/v1/partner/customer/{Identifier}"
);

if (response.StatusCode == HttpStatusCode.OK)
{
    // Access things like data["PartnerCustomerKey"]
    var data = Newtonsoft.Json.Linq.JObject.Parse(
      await response.Content.ReadAsStringAsync()
    );
}
```

> The above command returns JSON structured like this:

```json
{
  "PartnerCustomerKey": "p0123456789abcdefghijklmno",
  "PartnerRemoteId": "customer-unique-id-from-partner",  
  "Email": "jane-work@example.com",
  "EmailAlt": "jane-personal@example.com",
  "Name": "Jane Doe",
  "BusinessName": "Jane's Auto Repair",
  "BusinessStreet": "123 Main Street",
  "BusinessCity": "Los Angeles",
  "BusinessStateCode": "CA",
  "BusinessCountryCode": "US",
  "BusinessPostalCode": "90210",
  "BusinessCounty": "Los Angeles County",
  "BusinessLatitude": null,
  "BusinessLongitude": null,
  "BusinessPhone": "+18005551234",
  "MobilePhone": "+13105555678",
  "DateCreated": "2018-05-03T08:35:55.6492475+00:00",
  "DateLinked": "2018-05-03T08:35:55.6492475+00:00",
  "GroupInvitations": [
    "group-one",
    "group-two",
    "group-three"
  ],
  "Metadata": {
    "Anything": {
      "ThePartner": "wants",
      "SuchAs": 12345,
      "OrEven": true
    }
  }
}
```

This endpoint retrieves a Partner Customer that you've previously created. You can identify the customer using any of: `PartnerCustomerKey` (our identifier), `PartnerRemoteId` (your identifier), `Email`, or `EmailAlt`.

### HTTP Request

`GET /api/v1/partner/customer/{Identifier}`

### URL Parameters

Parameter          | Description
------------------ | -----------
Identifier         | The `PartnerRemoteId`, `Email`, or `EmailAlt` you previously provided when creating or updating a Partner Customer, or the `PartnerCustomerKey` we generated when a Partner Customer was created.


## Update a Partner Customer

```csharp
var response = await DNAPI.PartnerSendAsync
(
  HttpMethod.Patch,
  $"/api/v1/partner/customer/{Identifier}",
  new 
  {
    Email = "jane-work@example.com",
    // Removing them from other groups you may have invited
    // them to, and only letting them access your group-three.
    GroupInvitations = new List<int> {
      "group-three"
    }
  }
);

if (response.StatusCode == HttpStatusCode.OK)
{
  // Partner Customer updated. Access updated data:
  var data = Newtonsoft.Json.Linq.JObject.Parse(
    await response.Content.ReadAsStringAsync()
  );
}
```

> The update customer API call returns JSON like this.

```json
{
  "PartnerCustomer": {
    "PartnerCustomerKey": "p0123456789abcdefghijklmno",
    "PartnerRemoteId": "customer-unique-id-from-partner",
    "Email": "jane-work@example.com",
    "EmailAlt": "jane-personal@example.com",
    "Name": "Jane Doe",
    "BusinessName": "Jane's Auto Repair",
    "BusinessStreet": "123 Main Street",
    "BusinessCity": "Los Angeles",
    "BusinessStateCode": "CA",
    "BusinessCountryCode": "US",
    "BusinessPostalCode": "90210",
    "BusinessCounty": "Los Angeles County",
    "BusinessLatitude": null,
    "BusinessLongitude": null,
    "BusinessPhone": "+18005551234",
    "MobilePhone": "+13105555678",
    "DateCreated": "2018-05-03T08:35:55.6492475+00:00",
    "DateLinked": "2018-05-03T08:35:55.6492475+00:00",
    "Metadata": {
      "Anything": {
        "ThePartner": "wants",
        "SuchAs": 12345,
        "OrEven": true
      }
    },
    "UserNotified": false
  },
  "GroupInvitationErrors: []
}
```

Updating a Partner Customer is normally useful for adjusting metadata stored with the Partner Customer account that can impact their benefits on Diagnostic Network. For example, adding or removing closed/private user groups on DN that the Partner Customer is authorized to join. You can identify the customer using any of: `PartnerCustomerKey` (our identifier), `PartnerRemoteId` (your identifier), `Email`, or `EmailAlt`.

When updating a **stub user** Partner Customer that has not yet been linked to a real DN user account, any updated information will be used if and when the user registers to join Diagnostic Network, which can provide a better user experience for your customer. For example, if the customer changes their email address on your site, changing it on their Partner Customer record could make it easier for them to join DN later.

For **real user** Partner Customers, or stub users that have already been linked to a real DN user account, updating a Partner Customer's basic information (like name, email, etc) will have no real impact, as these changes **do not** flow through to the user's account information on DN. (Changing that information on a real user account requires _User Authentication_ and accessing the User API endpoints.)

To sever a relationship with a Partner Customer completely, use the [Delete Partner Customer](#delete-a-partner-customer) endpoint.

### HTTP Request

`PATCH /api/v1/partner/customer/{Identifier}`

### URL Parameters

Parameter          | Description
------------------ | -----------
Identifier         | The `PartnerRemoteId`, `Email`, or `EmailAlt` you previously provided when creating or updating a Partner Customer, or the `PartnerCustomerKey` we generated when a Partner Customer was created.

### PATCH JSON Fields

Fields               | Type            | Description
-------------------- | --------------- | -----------
Email                | `string`        | (Optional) The email address of the user.
EmailAlt             | `string`        | (Optional) An alternate email address of the user.
Name                 | `string`        | (Optional) The full name (must contain at least two name parts).
BusinessName         | `string`        | (Optional) The business name.
BusinessStreet       | `string`        | (Optional) The business street address.
BusinessCity         | `string`        | (Optional) The business city.
BusinessStateCode    | `string`        | (Optional) The business state (ISO standard 2-letter code).
BusinessCountryCode  | `string`        | (Optional) The business country (ISO standard 2-letter code).
BusinessPhone        | `string`        | (Optional) The business phone number (ideally E.164 format)
MobilePhone          | `string`        | (Optional) The user's mobile phone number (ideally E.164 format)
PartnerRemoteId      | `string`        | (Optional) Unique identifier for this customer, supplied by the partner, which can be used in the future for API requests that originate _from_ Diagnostic Network to you.
BusinessPostalCode   | `string`        | (Optional) The business zip/postal code.
BusinessCounty       | `string`        | (Optional) The business county (not country)
BusinessLatitude     | `decimal`       | (Optional) The business latitude coordinates.
BusinessLongitude    | `decimal`       | (Optional) The business longitude coordinates.
GroupInvitations     | `array<string>` | (Optional) Array of your user groups to which this user will have access. If this is different from a previous `POST` or `PATCH` request, they'll be removed from any groups not listed here.
Metadata             | `json`          | (Optional) Any additional data DN or the partner needs associated with this customer, perhaps to unlock other features on DN.

## Delete a Partner Customer

```csharp
var response = await DNAPI.PartnerSendAsync
(
  HttpMethod.Delete,
  $"/api/v1/partner/customer/{Identifier}"
);

if (response.StatusCode == HttpStatusCode.OK)
{
    // Partner Customer deleted. Access data["UserNotified"]:
    var data = Newtonsoft.Json.Linq.JObject.Parse(
      await response.Content.ReadAsStringAsync()
    );
}
```

> The above command returns JSON structured like this:

```json
{
  "UserNotified": true 
}
```

This endpoint is used to delete a Partner Customer, and if an existing Diagnostic Network member account is linked to this record, the relationship between the partner and the DN member will be severed: they'll no longer receive any partner benefits they may have previously been receiving. The user will also be removed from any closed/private user groups they previously were subscribed to for the partner. You can identify the customer using any of: `PartnerCustomerKey` (our identifier), `PartnerRemoteId` (your identifier), `Email`, or `EmailAlt`.

The entire partner customer record will be deleted once we have notified the user of the change to their account status. This should allow you to comply with any GDPR requests to erase all traces of a user. Until then, the GET response data includes a `DateLinked` field that will be `null` if the `DELETE` endpoint has been used on the Partner Customer.

### HTTP Request

`DELETE /api/v1/partner/customer/{Identifier}`

### URL Parameters

Parameter          | Description
------------------ | -----------
Identifier         | The `PartnerRemoteId`, `Email`, or `EmailAlt` you previously provided when creating or updating a Partner Customer, or the `PartnerCustomerKey` we generated when a Partner Customer was created.

