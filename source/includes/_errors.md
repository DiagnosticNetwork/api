# Errors

The API uses the following HTTP status codes for errors or other issues.

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
