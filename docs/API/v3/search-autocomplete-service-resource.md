---
# required metadata 

title: Autocomplete | NuGet V3 API | Microsoft Docs
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
ms.assetid: ead5cf7a-e51e-4cbb-8798-58226f4c853f

# optional metadata

description: The search autocomplete service supports interactive discovery of package IDs and versions.
keywords: API, autocomplete, package, ID, partial, substring, ngram, search, version, identifier
ms.reviewer:
- karann
- unniravindranathan

---

# Autocomplete

It is possible to build a package ID and version autocomplete experience using the V3 API. The resource used for making
autocomplete queries is the `SearchAutocompleteService` resource found in the [service index](service-index.md).

## Versioning

There are three resource `@type` values for searching. All three values represent the exact same API surface area.

The `@type` values are:

 - `SearchAutocompleteService`
 - `SearchAutocompleteService/3.0.0-beta`
 - `SearchAutocompleteService/3.0.0-rc`

### API Changes

 - March 2017: Added a `semVerLevel` query parameter which defaults to filtering out SemVer 2.0.0 packages.

## Base URL

The base URL for the following APIs is the value of the `@id` property associated with one of the aforementioned
resource `@type` values. In the following document, this placeholder URL will be used:

```
https://example/autocomplete
```

## HTTP Methods

All URLs found in the autocomplete resource only support the HTTP method `GET`.

## Search for package IDs

The first autocomplete API supports searching for part of a package ID string. This is great when you want to provide
a package typeahead feature in a user interface integrated with a V3 NuGet package source.

An package with only unlisted versions will not appear in the results.

```
GET https://example/autocomplete?q={QUERY}&skip={SKIP}&take={TAKE}&prerelease={PRERELEASE}&semVerLevel={SEMVERLEVEL}
```

### Request parameters

Name        | In     | Type    | Notes
----------- | ------ | ------- | -----
q           | URL    | string  | Optional: the string to compare against package IDs
skip        | URL    | integer | Optional: the number of results to skip, for pagination
take        | URL    | integer | Optional: the number of results to return, for pagination
prerelease  | URL    | boolean | Optional: `true` or `false` determining whether to include [pre-release packages](../../create-packages/prerelease-packages.md)
semVerLevel | URL    | string  | Optional: a SemVer 2.0.0 version string 

The autocomplete query `q` is parsed in a manner that is defined by the server implementation. NuGet.org supports
querying for the prefix of package ID tokens, which are pieces of the ID produced by spliting the original by camel
case and symbol characters.

The `skip` parameter defaults to 0.

The `take` parameter should be an integer greater than zero. The server implementation may impose a maximum value.

If `prerelease` is not provided, pre-release packages are excluded.

The `semVerLevel` query parameter is used to opt-in to SemVer 2.0.0 packages. If this query parameter is excluded, only
package IDs with SemVer 1.0.0 compatible versions will be returned. If `semVerLevel=2.0.0` is provided, both SemVer
1.0.0 and SemVer 2.0.0 compatible packages will be returned. See the
[SemVer 2.0.0 protocol](https://github.com/NuGet/Home/wiki/Semver-2.0.0-Protocol) for more information.

### Response

The response is JSON document containing up to `take` autocomplete results.

Autocomplete results returned by nuget.org looks something like this, when using `q=storage&prerelease=true`:

[!code-JSON [autocomplete-id-result.json](./_data/autocomplete-id-result.json)]

The root JSON object has the following properties:

Name      | Type             | Notes
--------- | ---------------- | -----
totalHits | integer          | Required: the total number of matches, disregarding `skip` and `take`
data      | array of strings | Required: the package IDs matched by the request

## Enumerate package versions

Once a package ID is discovered using the previous API, a client can use the autocomplete API to enumerate package
versions for a provided package ID.

An package version that is unlisted will not appear in the results.

```
GET https://example/autocomplete?id={ID}&prerelease={PRERELEASE}&semVerLevel={SEMVERLEVEL}
```

### Request parameters

Name        | In     | Type    | Notes
----------- | ------ | ------- | -----
q           | URL    | string  | Optional: the package ID to fetch versions for
prerelease  | URL    | boolean | Optional: `true` or `false` determining whether to include [pre-release packages](../../create-packages/prerelease-packages.md)
semVerLevel | URL    | string  | Optional: a SemVer 2.0.0 version string 

If `prerelease` is not provided, pre-release packages are excluded.

The `semVerLevel` query parameter is used to opt-in to SemVer 2.0.0 packages. If this query parameter is excluded, only
SemVer 1.0.0 versions will be returned. If `semVerLevel=2.0.0` is provided, both SemVer 1.0.0 and SemVer 2.0.0 versions
will be returned. See the [SemVer 2.0.0 protocol](https://github.com/NuGet/Home/wiki/Semver-2.0.0-Protocol) for more
information.

### Response

The response is JSON document containing all package versions of the provided package ID, filtering by the given query
parameters.

The version list result returned by nuget.org looks something like this, when using
`id=nuget.protocol&prerelease=true`:

[!code-JSON [autocomplete-version-result.json](./_data/autocomplete-version-result.json)]

The root JSON object has the following property:

Name      | Type             | Notes
--------- | ---------------- | -----
data      | array of strings | Required: the package versions matched by the request

The package versions in the `data` array could contain SemVer 2.0.0 build metadata (e.g. `1.0.0+metadata`) if the
`semVerLevel=2.0.0` was provided in the query string.
