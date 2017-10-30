---
# required metadata 

title: Service Index | NuGet V3 API | Microsoft Docs
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
ms.assetid: 2f6d6cf2-53fb-417a-b1d8-e0ac591c1699

# optional metadata

description: The V3 service index is the entry point of the NuGet V3 and enumerates the capabilities of the V3 server.
keywords: API, V3, NuGet, entry point, versioning, resources, discovery
ms.reviewer:
- karann
- unnir

---

# Service Index

The V3 service index is a JSON document that is the entry point for a V3 NuGet package source and allows a client
implementation to discover the package source's capabilities. The service index is a JSON object with two required
properties: `version` (the schema version of the service index) and `resources`  (the endpoints or capabilities of the
package source).

nuget.org's service index is located here:
```
https://api.nuget.org/v3/index.json
```

The service index for nuget.org looks something like is this:

[!code-JSON [service-index.json](./_data/service-index.json)]

## Versioning

The `version` value is a SemVer 2.0.0 parseable version string which indicates the schema version of the service index.
The V3 API mandates that the version string has a major version number of `3`. As non-breaking changes are made to the
service index schema, the version string's minor version will be increased.

Each resource in the service index is versioned independently from the service index schema version.

The current schema version is `3.0.0-beta.1`.

## Resources

The `resources` property contains an array of resources supported by this package source.

### Resource

A resource is an object in the `resources` array. It represents a versioned capability of a V3 package source. A resource
has two required properties: `@id` (the URL of the resource) and `@type` (an identifier of what the resource is used
for).

The `@id` property is a string that is the URL to the resource. This URL must be absolute and must either have the HTTP
or HTTPS schema.

The `@type` property is a string used to identify the specific protocol to use when interacting with resource. The type
of the resource is an opaque string but generally has the format:

```
{RESOURCE_NAME}/{RESOURCE_VERSION}
```

Clients are expected to hard code the `@type` values that they understand and look them up in a V3 package source's
service index.

For the sake of this V3 documentation, the documentation about different resources will essentially be grouped by the
`{RESOURCE_NAME}` found in the service index which is analogous to grouping by scenario.

There is no requirement that each resource has a unique `@id` or `@type`. It is up to the client implementation to
determine which resource to prefer over another. One possible implementation is that resources of the same or
compatible `@type` can be used in a round-robin fashion in case of connection failure or server error.

Optionally, the resource can have a `comment` property which is a human readable string describing the resource.

### Client version

A resource `@type` can optionally have the following form:

```
{RESOURCE_NAME}/Versioned
```

When the `@type` is suffixed with the `/Versioned` string, an additional property is required on the resource:
`clientVersion`.

This value is the minimum client version compatible with this resource. The client version is SemVer 2.0.0 compliant
version string. A client implementation knows its own version (also a SemVer 2.0.0 version) and compares it to the
resource `clientVersion` value using standard SemVer 2.0.0 comparison rules. If the client's own version is greater
than or equal to the resource `clientVersion` and if the `{RESOURCE_NAME}` is known, this resource should be considered
consumable by the client implementation.

For this `clientVersion` model to work, there needs to be some agreement between the server and client implementors
about what protocol each `{RESOURCE_NAME}` and `clientVersion` combination implies.

This feature allows server implementations to backport resources to clients that have already shipped. When no
`clientVersion` is specified on the resource, there is no way for a client that is already shipped to discover the
resource since the `@type` values that client knows about are hard coded.

> [!Note]
> For nuget.org, the client versions in the service index align with the official NuGet client.
