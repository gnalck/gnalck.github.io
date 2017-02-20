---
layout:     post
title:      JWT audience validation - a URI by any other string
date:       2016-10-03 15:31:19
---

I recently ran into a bug at work related to URI comparison, which has gotten me to thinking about URLs, URNs, and URIs (oh my!). There are a couple of good resources going over the difference [between the three](http://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn/1984225), so I won't bore you with that. Instead, let's talk about the nature of URL comparison, and its relation to [OWIN token flows](https://en.wikipedia.org/wiki/Open_Web_Interface_for_.NET).

## The bug
So what precipitated all of this? In [Azure Active Directory](), it is the responsibility of the application that receives a token to validate that it is indeed intended for them. This is done by looking at the [audience claim](https://tools.ietf.org/html/rfc7519#section-4.1.3) within the [JWT](https://jwt.io/introduction/) token we receive from whatever client is trying to authenticate with us. That audience, which the client specifies when requesting a token for you (the resource), is none other than the application's App ID URI:

![App ID URI](https://acom.azurecomcdn.net/80C57D/cdn/mediahandler/docarticles/dpsmedia-prod/azure.microsoft.com/en-us/documentation/articles/api-management-howto-protect-backend-with-aad/20160816064407/api-management-aad-sso-uri.png)

So far so good, but things get a bit worse off when you realize it is [standard](https://msdn.microsoft.com/en-us/library/dn451147(v=vs.114).aspx) to validate the URI in the token with the one we have stored locally via a strict string comparison. This is unfortunate, and where the bug emerges. My [allowedAudienceUris](https://www.google.com/search?q=AllowedAudienceUris&ie=utf-8&oe=utf-8) had e.g., `http://my-app`, but the actual App ID in Azure was `http://my-app/` (note the trailing slash)!

## A URI by any other string...
Of course, all of this could have been avoided by looking at the configs more carefully (luckily, I was able to find this issue pretty easily), but it doesn't quite end there. You see, the 'mismatched' App ID vs Audience values were actually [equivalent URIs](https://tools.ietf.org/html/rfc3986#section-3.3)! From the spec:

```path-abempty  = *( "/" segment )```

If you still have faith in the good old [.NET Standard Library](https://docs.microsoft.com/en-us/dotnet/articles/standard/library), then you can also verify it with the following simple test:

```csharp 
var a = new Uri("http://example.com");
var b = new Uri("http://example.com/")
Console.WriteLine(a == b); // true
```

Unfortunately, the dogma of strict string comparison here is tearing two perfectly equivalent strings apart! I think it was a great man who said:

> What's in a URI? That which we identify 
> with a trailing slash after the authority
> would be just as equal without it

## Closing thoughts
In fact, the [JWT standard](https://tools.ietf.org/html/rfc7519#section-2) even calls out the possibility of a given claim (in this case, the `aud` one), to be a `StringOrURI`. Unfortunately, it is left to the application to determine how to interpret this, but I would argue that 'the application' in this case is the broader OWIN context in which AAD token flows operate.

The lesson here being that one annoying [code analysis warning](https://msdn.microsoft.com/en-us/library/ms182360.aspx) is definitely worth considering. Any time you deal with URIs, try to use a [library](https://msdn.microsoft.com/en-us/library/system.uri(v=vs.110).aspx) to do the comparison. When using another library, hope that it does the same.
