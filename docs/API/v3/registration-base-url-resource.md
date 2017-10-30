---
# required metadata 

title: Package Metadata | NuGet V3 API | Microsoft Docs
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
ms.assetid: 96b07019-c2e1-4f40-9290-f65ad71af3b1

# optional metadata

description: The package registration base URL allows fetching metadata about packages.
keywords: API, NuGet, package, metadata, registration, details, description, HTTP, unlisted
ms.reviewer:
- karann
- unniravindranathan

---

# Package Metadata

It is possible to fetch metadata about the packages available on a package source using the NuGet V3 API. This
metadata can be fetched using the `RegistrationsBaseUrl` resource found in the [service index](service-index.md).

The collection of the documents found under `RegistrationsBaseUrl` are often called "registrations" or "registration
blobs". The set of documents under a single `RegistrationsBaseUrl` is referred to as a "registration hive". A
registration have contains all metadata about every package available on a package source.

## Versioning

There have been three versions of the `RegistrationsBaseUrl` released. All versions of the `RegistrationsBaseUrl` has
the same general API surface area. However, there have been changes in the `Content-Encoding` and the support of SemVer
2.0.0 packages. For more information about SemVer 2.0.0,
[SemVer 2.0.0 protocol](https://github.com/NuGet/Home/wiki/Semver-2.0.0-Protocol) design.

nuget.org has three registration hives available for various client versions.

### Initial hive

The `@type` values for the initial release of the `RegistrationsBaseUrl` are:

 - `RegistrationsBaseUrl`,
 - `RegistrationsBaseUrl/3.0.0-beta`
 - `RegistrationsBaseUrl/3.0.0-rc`

These registrations are not compressed (meaning they use an implied `Content-Encoding: identity`). SemVer 2.0.0
packages are **excluded** from this hive.

### Gzipped hive - February 2016

The `@type` value for the second release of the `RegistrationsBaseUrl` is:

 - `RegistrationsBaseUrl/3.4.0`

These registrations are compressed using `Content-Encoding: gzip`. SemVer 2.0.0 packages are **excluded** from this
hive.

### SemVer 2.0.0 hive - April 2017

The `@type` values for the third release of the `RegistrationsBaseUrl` is:

 - `RegistrationsBaseUrl/3.6.0`
 - `RegistrationsBaseUrl/Versioned`
   - `clientVersion` of `4.3.0-alpha`

These registrations are compressed using `Content-Encoding: gzip`. SemVer 2.0.0 packages are **included** in this hive.

## Base URL

The base URL for the following APIs is the value of the `@id` property associated with the aforementioned
resource `@type` values. In the following document, this placeholder URL will be used:

```
https://example/registration/
```

## HTTP methods

All URLs found in the registration resource only support the HTTP method `GET`.

## Registration index

The registration resource groups package metadata by package ID. It is not possible to get data about more than one
package ID at a time. This resource provides no way to discover package IDs. Instead the client is assumed to already
know the desired package ID. Available metadata about each package version varies by server implementation. The package
registration blobs have the following hierarchical structure:

- **Index**: the entry point for the package metadata, shared by all packages on a source with the same package ID.
- **Page**: a grouping of package versions. The number of package versions in a page is defined by server implementation.
- **Leaf**: a document specific to a single package version.

The URL of the registration index is predictable and can be determined by the client given a package ID and the
registration resource's `@id` value from the service index. The URLs for the registration pages and leaves are
discovered by inspecting the registration index.

### Registration pages and leaves

Although it is not strictly required for a server implementations to store registration leafs in seperate
registration page documents, it is a recommended practice to conserve client-side memory. Instead of inlining all
registration leaves in the index or immediately storing leaves in page documents, it is recommended that the server
implementation define some heuristic to choose between the two approaches based on the number of package versions
or cumulative size of package leaves.

Storing all package versions (leaves) in the registration index saves on the number of HTTP requests necessary to fetch
package metadata but means that a larger document must be downloaded and more client memory must be allocated. On the
other hand, if the server implementation immediately stores registration leaves in seperate page documents, the client
must perform more HTTP requests to get the information it needs.

The heuristic that nuget.org uses is as follows: if there are 128 or more versions of a package, break the leaves
into pages of size 64. If there are less than 128 versions, inline all leaves into the registration index.

### Location

The location of the registration index is:
```
https://example/registration/{LOWER_ID}/index.json
```

### Request parameters

Name     | In     | Type    | Notes
-------- | ------ | ------- | -----
LOWER_ID | URL    | string  | Required: the package ID, lowercase

The `LOWER_ID` value is the desired package ID lowercased using the rules implemented by .NET's
[`System.String.ToLowerInvariant()`](https://msdn.microsoft.com/en-us/library/system.string.tolowerinvariant.aspx)
method.

### Response

The response is a JSON document which has a root object with the following properties:

Name  | Type             | Notes
----- | ---------------- | -----
count | integer          | Required: the number of registration pages in the index
items | array of objects | Required: the array of registration pages

Each item in the index object's `items` array is a JSON object representing a registration page.

#### Registration page object

The registration page object found in the registration index has the following properties:

Name   | Type             | Notes
------ | ---------------- | -----
@id    | string           | Required: the URL to the registration page
count  | integer          | Required: the number of registration leaves in the page
items  | array of objects | Optional: the array of registration leaves and their associate metadata
lower  | string           | Required: the lowest SemVer 2.0.0 version in the page (inclusive)
parent | string           | Required: the URL to the registration index
upper  | string           | Required: the highest SemVer 2.0.0 version in the page (inclusive)

The `lower` and `upper` bounds of the page object are useful when the metadata for a specific page version is needed.
These bounds can be used to fetch the only the registration page needed. The version strings adhere to
[NuGet's version rules](../../reference/package-versioning.md). The version strings are normalized and do not include
build metadata.

If the `items` property is not present in the registration page object, the URL specified in the `@id` must be used to
fetch metadata about individual package versions. The `items` array is sometimes excluded from the page
object as an optimization. If the number of versions of a single package IDs is very large, then the registration index
document will be massive and wasteful to process for a client that only cares about a specific version or small range
of versions.

Note that is the `items` property is present, the `@id` property need not be used, since all of the page data is
already inlined in the `items` property.

Each item in the page object's `items` array is a JSON object representing a registration leaf and it's associated
metadata.

#### Registration leaf object in a page

The registration leaf object found in a registration page has the following properties:

Name           | Type   | Notes
-------------- | ------ | -----
@id            | string | Required: the URL to the registration leaf
catalogEntry   | object | Required: the catalog entry containing the package metadata
packageContent | string | Required: the URL to the package content (.nupkg)

Each registration leaf object represents data associated with a single package version.

#### Catalog entry

The `catalogEntry` property in the registration leaf object has the following properties:

Name                     | Type                       | Notes
------------------------ | -------------------------- | -----
@id                      | string                     | Required: the URL to document used to produce this object
authors                  | string or array of strings | Optional
dependencyGroups         | array of objects           | Required: the URL to the package content (.nupkg)
description              | string                     | Optional
iconUrl                  | string                     | Optional
id                       | string                     | Required: the ID of the package
licenseUrl               | string                     | Optional
listed                   | boolean                    | Optional: should be considered as listed if absent
minClientVersion         | string                     | Optional
projectUrl               | string                     | Optional
published                | string                     | Optional: a string containing a ISO 8601 timestamp of when the package was published
requireLicenseAcceptance | boolean                    | Optional
summary                  | string                     | Optional
tags                     | string or array of string  | Optional
title                    | string                     | Optional
version                  | string                     | Required: the version of the package

The `dependencyGroups` property is an array of objects representing the dependencies of the package, grouped by target
framework.

#### Package dependency group

Each dependency group object has the following properties:

Name            | Type             | Notes
--------------- | ---------------- | -----
targetFramework | string           | Optional: the target framework that these dependencies are applicable to
dependencies    | array of objects | Optional

The `targetFramework` string uses the format implemented by NuGet's .NET library
[NuGet.Frameworks](https://www.nuget.org/packages/NuGet.Frameworks/). If no `targetFramework` is specified, the
dependency group applies to all target frameworks.

The `dependencies` property is an array of objects, each representing a package dependency of the current package.

#### Package dependency

Each package dependency has the following properties:

Name         | Type   | Notes
------------ | ------ | -----
id           | string | Required: the ID of the package dependency
range        | object | Optional: the allowed [version range](../../reference/package-versioning.md#version-ranges-and-wildcards) of the dependency
registration | string | Optional: the URL to the registration index for this dependency

If the `range` property is excluded or an empty string, the client should default to the version range `(, )`. That is,
any version of the dependency is allowed.

#### Example

The registration index for NuGet.Server.Core on nuget.org looks something like this:

[!code-JSON [package-registration-index.json](./_data/package-registration-index.json)]

In this particular case, the registration index has the registration page inlined so no extra requests are needed to
fetch metadata about individual package versions.

## Registration page

The registration page contains registration leaves. The URL to fetch a registration page is determined by the `@id`
property in the [registration page object](#registration-page-object) mentioned above.

When the `items` array is not provided in the registration index, an HTTP GET request of the `@id` value will return a
JSON document which has an object as its root. The object has the following properties:

Name   | Type             | Notes
------ | ---------------- | -----
@id    | string           | Required: the URL to the registration page
count  | integer          | Required: the number of registration leaves in the page
items  | array of objects | Required: the array of registration leaves and their associate metadata
lower  | string           | Required: the lowest SemVer 2.0.0 version in the page (inclusive)
parent | string           | Required: the URL to the registration index
upper  | string           | Required: the highest SemVer 2.0.0 version in the page (inclusive)

The shape of the registration leaf objects is the same as in the registration index
[above](#registration-leaf-object-in-a-page).

## Registration leaf

The registration leaf contains information about a specific package ID and version. The metadata about the specific
version may not be available in this document. Package metadata should be fetched from the
[registration index](#registration-index) or the [registration page](#registration-page) (which is discovered using
the registration index).

The URL to fetch a registration leaf is obtain from the `@id` property of a registration leaf object in either a
registration index or registration page.

The registration leaf is a JSON document with a root object with the following properties:

Name           | Type    | Notes
-------------- | ------- | -----
@id            | string  | Required: the URL to the registration leaf
catalogEntry   | string  | Optional: the URL to the catalog entry that produced these leaf
listed         | boolean | Optional: should be considered as listed if absent
packageContent | string  | Optional: the URL to the package content (.nupkg)
published      | string  | Optional: a string containing a ISO 8601 timestamp of when the package was published
registration   | string  | Optional: the URL to the registration index

> [!Note]
> On nuget.org, the `published` value is set to year 1900 when the package is unlisted.

### Example

The registration index for NuGet.Versioning 4.3.0 on nuget.org looks something like this:

[!code-JSON [package-registration-leaf.json](./_data/package-registration-leaf.json)]
