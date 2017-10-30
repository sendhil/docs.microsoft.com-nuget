---
# required metadata 

title: Report Abuse URL Template, NuGet V3 API | Microsoft Docs
author:
- joelverhagen
- kraigb
ms.author:
- joelverhagen
- kraigb
manager: skofman
ms.date: 10/26/2017
ms.topic: reference
ms.prod: nuget
ms.technology: null
ms.assetid: 148d743a-09e5-4539-8454-675be11902db

# optional metadata

description: The report abuse URL template allows clients to display a report abuse link in their UI.
keywords: NuGet V3 API report abuse, NuGet V3 API file complaint, NuGet.org report URL template
ms.reviewer:
- karann
- unniravindranathan

---

# Report Abuse URL Template

It is possbile for a client to build a URL that can be used by the user to report abuse about a specific package. This
is useful when a V3 package source wants to enable all client experiences (even 3rd party) to delegate abuse reports to
the package source.

The resource used for fetching package content is the `ReportAbuseUriTemplate` resource found in the
[service index](service-index.md).

## Versioning

There are two resource `@type` values for building report abuse URLs. Both values represent the exact same API surface
area.

The `@type` values are:

 - `ReportAbuseUriTemplate/3.0.0-beta`
 - `ReportAbuseUriTemplate/3.0.0-rc`

## URL template

The URL for the following API is the value of the `@id` property associated with one of the aforementioned
resource `@type` values. In the following document, this placeholder URL will be used:

```
https://example/abuse/{id}/{version}
```

## HTTP methods

Although the client is not intended to make requests to the report abuse URL on behalf of the user, the web page
should support the `GET` method to allow a clicked URL to be easily opened in a web browser.

## Construct the URL

Given a known package ID and version, the client implementation can construct a URL used to access a web interface. The
client implementation should display this constructed URL (or clickable link) to the user allowing them to open a web
browser to the URL and make any necessary abuse report. The implementation of the abuse report form is determined by
the server implementation.

The value of the `@id` is a URL string containing any of the following placeholder tokens:

### URL placeholders

Name        | Type    | Notes
----------- | ------- | -----
`{id}`      | string  | Optional: the package ID to report abuse for
`{version}` | string  | Optional: the package version to report abuse for

The `{id}` and `{version}` values interpreted by the server implementation must be case insenstive and not sensitive to
whether the version is normalized.

For example, nuget.org's report abuse template looks like this:

```
https://www.nuget.org/packages/{id}/{version}/ReportAbuse
```

If the client implementation needs to display a link to the report abuse form for NuGet.Versioning 4.3.0, it would
produce the following URL and provide it to the user:

```
https://www.nuget.org/packages/NuGet.Versioning/4.3.0/ReportAbuse
```