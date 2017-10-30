---
# required metadata 

title: nuget.org-only Protocols | Microsoft Docs
author: anangaur
ms.author: anangaur
manager: unniravindranathan
ms.date: 10/30/2017
ms.topic: article
ms.prod: nuget
ms.technology: null
ms.assetid: ba1d9742-9f1c-42ff-8c30-8e953e23c501

# optional metadata

description: The evolving nuget.org-only protocols to interact with NuGet clients.
ms.reviewer:
- kraigb
- karann-msft

---
# nuget.org-only Protocols

To interact with nuget.org, clients need to follow certain protocols. Because these protocols keep evolving, clients
must identify the protocol version they use when calling specific nuget.org APIs. This allows nuget.org to introduce
changes in a non-breaking way for the old clients.

> [!Note]
> The APIs documented on this page are specific to nuget.org and there is no expectation for other NuGet server
> implementations to introduce these APIs. 

For information about the NuGet V3 API implemented broadly across the NuGet ecosystem, see the
[V3 API overview](v3/overview.md).

This topic lists various protocols as and when they come to existence.

## NuGet protocol version 4.1.0

The 4.1.0 protocol specifies usage of verify-scope keys to interact with services other than nuget.org, to validate a
package against a nuget.org account. 

Validation ensures that the user-created API keys are used only with nuget.org, and that other verification or
validation from a third-party service is handled through a one-time use verify-scope keys. These verify-scope keys can
be used to validate that the package belongs to a particular user (account) on nuget.org.

### Client requirement

Clients are required to pass the following header when they make API calls to **push** packages to nuget.org:

```
X-NuGet-Protocol-Version: 4.1.0
```

Note that the pre-existing `X-NuGet-Client-Version` header has the same purpose but is now deprecated and should no
longer be used.

The **push** protocol itself is described in the documentation for the
[`PackagePublish` resource](v3/package-publish-resource.md).

If a client interacts with externals services and needs to validate whether a package belongs to a particular user
(account), it should use the following protocol and use the verify-scope keys and not the API keys from nuget.org.

### API to request a verify-scope key

This API is used to get a verify-scope key for a nuget.org author to validate a package owned by him/her.

```
POST api/v2/package/create-verification-key/{ID}/{VERSION}
```

#### Request parameters

Name           | In     | Type   | Required | Notes
-------------- | ------ | ------ | -------- | -----
ID             | URL    | string | yes      | The package identidier for which the verify scope key is requested
VERSION        | URL    | string | no       | The package version
X-NuGet-ApiKey | Header | string | yes      | For example, `X-NuGet-ApiKey: {USER_API_KEY}`

#### Response

```
{
    "Key": "{Verify scope key from nuget.org}",
    "Expires": "{Date}"
}
```

### API to verify the verify scope key

This API is used to validate a verify-scope key for package owned by the nuget.org author.

```
GET api/v2/verifykey/{ID}/{VERSION}
```

#### Request parameters

Name           | In     | Type   | Required | Notes
-------------  | ------ | ------ | -------- | -----
ID             | URL    | string | yes      | The package identifier for which the verify scope key is requested
VERSION        | URL    | string | no       | The package version
X-NuGet-ApiKey | Header | string | yes      | For example, `X-NuGet-ApiKey: {VERIFY_SCOPE_KEY}`

> [!Note]
> This verify scope API key expires in a day's time or on first use, whichever occurs first.

#### Response

Status Code | Meaning
----------- | -------
200         | The API key is valid
403         | The API key is invalid or not authorized to push against the package
404         | The package referred to by `ID` and `VERSION` (optional) does not exist
