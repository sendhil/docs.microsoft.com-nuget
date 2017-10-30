---
# required metadata 

title: Push and Delete, NuGet V3 API | Microsoft Docs
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
ms.assetid: 1eaa403a-5c13-4c05-9352-2f791b98aa7e

# optional metadata

description: The publish service allows clients to publish new packages and unlist or delete existing packages.
keywords: NuGet V3 API push package, NuGet V3 API delete package, NuGet V3 API unlist package, NuGet V3 API upload package, NuGet V3 API create package
ms.reviewer:
- karann
- unniravindranathan

---

# Push and Delete

It is possible to push and delete (or unlist, depending on the server implementation) packages using the NuGet V3 API.
Both operations are based off of the `PackagePublish` resource found in the [service index](service-index.md).

## Versioning

The only `@type` of the resource used for pushing and deleting packages is: `PackagePublish/2.0.0`.

## Base URL

The base URL for the following APIs is the value of the `@id` property of the `PackagePublish/2.0.0` resource in the
package source's [service index](service-index.md). For the documentation below, nuget.org's URL is used. Consider 
`https://www.nuget.org/api/v2/package` as a placeholder for the `@id` value found in the service index.

Note that this URL points to the same location as the legacy V2 push endpoint since the protocol for V2 and V3 push is
the same.

## HTTP methods

The `PUT` and `DELETE` HTTP methods are supported by this resource. For which methods are supported on each endpoint,
see below.

## Push a package

nuget.org supports pushing new packages using the following API. If the package with the provided ID and version
already exists, nuget.org will reject the push. Other package sources may support replacing an existing package.

```
PUT https://www.nuget.org/api/v2/package
```

### Request parameters

Name           | In     | Type   | Notes
-------------- | ------ | ------ | -----
X-NuGet-ApiKey | Header | string | Required: for example, `X-NuGet-ApiKey: {USER_API_KEY}`

### Request body

The request body can come in one of two forms.

#### Multipart form data

The request header `Content-Type` is `multipart/form-data` and the first item in the response body is the raw bytes of
the .nupkg being pushed. Subsequent items in the multipart body are ignored.

#### Raw bytes

If the response body is not `multipart/form-data`, the entire request body is assumed to be the raw bytes of the
.nupkg being pushed.

### Response

Status Code | Meaning
----------- | -------
200, 201    | The package was successfully pushed
400         | The provided package is invalid
409         | A package with the provided ID and version already exists

## Delete a package

nuget.org interprets the package delete request as an "unlist". This means that the package is still available for
existing consumers of the package but the package now longer appears in search results or in the web interface. For
more information about this practice, see the
[Deleted Packages](https://docs.microsoft.com/en-us/nuget/policies/deleting-packages) policy. Other server
implementations are free to interpret this signal as a hard delete, soft delete, or unlist. For example,
[NuGet.Server](https://www.nuget.org/packages/NuGet.Server) (a server implementation only supporting the older V2 API)
supports handling this request as either an unlist or a hard delete based on a configuration option.

```
DELETE https://www.nuget.org/api/v2/package/{ID}/{VERSION}
```

### Request parameters

Name           | In     | Type   | Notes
-------------- | ------ | ------ | -----
ID             | URL    | string | Required: the ID of the package to delete
VERSION        | URL    | string | Required: the version of the package to delete
X-NuGet-ApiKey | Header | string | Required: for example, `X-NuGet-ApiKey: {USER_API_KEY}`

### Response

Status Code | Meaning
----------- | -------
204         | The package was deleted
404         | No package with the provided `ID` and `VERSION` exists
