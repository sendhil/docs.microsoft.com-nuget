---
# required metadata 

title: NuGet API Introduction | Microsoft Docs
author:
- jver
- kraigb
ms.author:
- jver
- kraigb
manager: skofman
ms.date: 10/26/2017
ms.topic: article
ms.prod: nuget
ms.technology: null
ms.assetid: 100606fd-0a03-4274-8ebf-7d778f73869c

# optional metadata

description: Describes how to interact with NuGet programmatically: fetching package information and extending authentication with package sources
keyworkds: NuGet API, NuGet REST API, NuGet HTTP protocol, NuGet credentials, NuGet SDK
ms.reviewer:
- karann
- unniravindranathan

---

# NuGet API Introduction

There are a variety of ways to interact with NuGet programmatically.

## Consuming and publishing packages

The following methods allow you to interact with packages available on nuget.org and other V3-enabled NuGet package sources such as MyGet and Visual Studio Team Services package management:

1. Use the [HTTP-based V3 API](v3/overview.md) and write your own client code.
1. Use the [NuGet client SDK](nuget-client-sdk.md), a set of .NET libraries used by the official NuGet clients.

When interacting with [nuget.org's HTTP protocol](nuget-protocols.md), there are additional requirements and endpoints
available on top of the documented V3 API.

## Extending authentication

NuGet supports extensions on how users authenticate with packages sources via **credential providers**. There are two
forms of credential providers each used by a different client experience.

1. [Credential providers for Visual Studio](NuGet-Credential-Providers-for-Visual-Studio.md).
1. [Credential providers for nuget.exe](nuget-exe-Credential-Providers.md).
