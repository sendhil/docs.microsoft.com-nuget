---
# required metadata 

title: Overview | NuGet V3 API | Microsoft Docs
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
ms.assetid: 8c81f1ac-18c7-44d1-b2e3-584fe85dee6f

# optional metadata

description: The NuGet V3 API is a set of HTTP endpoints that can be used to download packages, fetch metadata, publish new packages, etc.
keywords: NuGet, API, V3, V2, JSON, registration, flat container, nupkg, metadata, search, push, publish, delete, unlist, protocol
ms.reviewer:
- karann
- unniravindranathan

---

# NuGet V3 API

The NuGet V3 API is a set of HTTP endpoints that can be used to download packages, fetch metadata, publish new packages,
and perform most other operations available in the official NuGet clients.

This API is used by the NuGet client in Visual Studio, nuget.exe, and the .NET CLI to perform NuGet operations such as
[`dotnet restore`](https://docs.microsoft.com/dotnet/articles/core/preview3/tools/dotnet-restore), search in the Visual
Studio UI, and [`nuget.exe push`](../../tools/cli-ref-push.md).

Note in some rare cases, nuget.org has additional requirements that are not enforced by other package sources. These
differences are documented by the [nuget.org Protocols](../nuget-protocols.md).

## Service index

The entry point for the V3 API is a JSON document in a well known location. This document is called the **service index**.
The location of the service index for nuget.org is `https://api.nuget.org/v3/index.json`.

This JSON document contains a list of *resources* which provide different functionality and fulfill different
use cases.

Clients that support the V3 API should accept one or more of these service index URL as the means of connecting to the
respective V3-enabled package sources.

For more information about the service index, see [its API reference](service-index.md).

## Versioning

The API is version 3 of NuGet's HTTP protocol. This protocol is generally referred to as "the V3 API".

The service index schema version is indicated by the `version` property in the service index. The V3 API mandates that
the version string has a major version number of `3`. As non-breaking changes are made to the service index schema, the
version string's minor version will be increased.

Older clients (such as nuget.exe 2.x) do not support the V3 API and only support the older V2 API, which is not
documented here.

## Resources and schema

The service index describes a variety of resources. The current set of supported resources are as follows:

Resource name                                                          | Description
---------------------------------------------------------------------- | -----------
[`PackagePublish`](package-publish-resource.md)                        | Push and delete (or unlist) packages.
[`SearchQueryService`](search-query-service-resource.md)               | Filter and search for packages by keyword.
[`SearchAutocompleteService`](search-autocomplete-service-resource.md) | Discover package IDs and versions by substring.
[`RegistrationsBaseUrl`](registration-base-url-resource.md)            | Get package metadata.
[`PackageBaseAddress`](package-base-address-resource.md)               | Get package content (.nupkg).
[`ReportAbuseUriTemplate`](report-abuse-resource.md)                   | Construct a URL to access a "report abuse" web page.

In general, all non-binary data returned by a V3 API resource will be serialized using JSON. The response schema
returned by each resource in the service index is defined individually for that resource. For more information about
each resource, see the resource topics noted above.

## Timestamps

All timestamps returned by the V3 API are UTC or are otherwise specified using ISO 8601 representation. 

## HTTP methods

Verb   | Use
------ | -----------
GET    | Performs a read-only operation, typically retrieving data.
PUT    | Creates a resource that doesn't exist or, if it does exist, updates it. Some resources may not support update.
DELETE | Deletes or unlists a resource.

## HTTP status codes

Code | Description
---- | -----
200  | Success, and there is a response body.
201  | Success, and the resource was created.
202  | Success, the request has been accepted but some work may still be incomplete and completed asynchronously.
204  | Success, but there is no response body.
400  | The parameters in the URL or in the request body aren't valid.
401  | The provided credentials are invalid.
403  | The action is not allowed given the provided credentials.
404  | The requested resource doesn't exist.
409  | The request conflicts with an existing resource.

## Authentication

Authentication is left up to the package source implementation to define. For nuget.org, only the `PackagePublish`
resource requires authentication via a special API key header. See
[PackagePublish resource](package-publish-resource.md) for details.
