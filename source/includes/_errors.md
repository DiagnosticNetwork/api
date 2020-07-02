# Errors

> Below is an example error object for a `400 Bad Request`. Its keys represent the API request parameters, each pointing to an array of one or more error messages. If there are any generic errors that are not specific to a single request parameter, the key will be an empty string.

```json
{
    "": 
    [
        "Error message #1, not specific to any particular request parameter."
    ],
    "ParameterA": 
    [
        "Error message #1 about request ParameterA",
        "Error message #2 about ParameterA"
    ],
    "ParameterB": 
    [
        "Error message #1 about request ParameterB"
    ]
}
```

> The benefit of this format is it enables you to place error messages near the parts of your UI that are relevant to the specific error. E.g. the `ParameterA` errors could be placed in red near the UI's input for `ParameterA`. 

The API uses the following HTTP status codes for errors or other issues. Typically a `400 Bad Request` is the most common error, indicating a problem with your request, and in this scenario the response will contain the error(s). 

Code       | Meaning
---------- | -------
303        | See Other -- Likely the resource you're trying to create already exists.
400        | Bad Request -- Your request is invalid.
401        | Unauthorized -- For Partners, your API key is wrong or your signature on the request is wrong.
403        | Forbidden -- The endpoint you tried to access is not authorized for the current authentication method.
404        | Not Found -- The resource you're trying to access could not be found with the identifier you provided.
405        | Method Not Allowed -- You tried to access with an invalid HTTP method.
406        | Not Acceptable -- You requested a format that isn't supported. (Please use JSON.)
500        | Internal Server Error -- We had a problem with our server. Try again later.
503        | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.
