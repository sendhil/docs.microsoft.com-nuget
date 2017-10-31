---
# required metadata 

title: Catalog, NuGet V3 API | Microsoft Docs
author:
- joelverhagen
- kraigb
ms.author:
- joelverhagen
- kraigb
manager: skofman
ms.date: 10/30/2017
ms.topic: reference
ms.prod: nuget
ms.technology: null
ms.assetid: cfd338b5-6253-48c0-88ba-17c6b98fc935

# optional metadata

description: The catalog is an index of all packages created, edited, and deleted on nuget.org.
keywords: NuGet V3 API catalog, nuget.org transaction log, replicate NuGet.org, clone NuGet.org, append-only record of NuGet.org
ms.reviewer:
- karann
- unniravindranathan

---

# Catalog

It is possible to enumerate all package creates, edits, and deletes on a V3 package source by inspecting a resource
called the "catalog". The catalog resource has the `Catalog` type in the [service index](service-index.md).

> [!Note]
> Since the catalog is not used by the official NuGet client, not all V3 package sources implement the catalog.
> nuget.org, however supports the catalog!

## Versioning

The only `@type` of the catalog resource is: `Catalog/3.0.0`.

## Base URL

The base URL for the following APIs is the value of the `@id` property associated with the aforementioned
resource `@type` values. In the following document, this placeholder URL will be used:

```
https://example/catalog/
```

## HTTP methods

All URLs found in the catalog resource only support the HTTP method `GET`.

## Catalog index

The catalog index is a document in a well-known location that has a list of catalog items, ordered cronologically. It
is the entry point of the catalog resource. The index is comprised of catalog pages. Each catalog page contains catalog
items. Each catalog item represents an event concerning a single package at a point in time. A catalog item can
represent a package that was created, unlisted, edited, or deleted from the package source. By processing the catalog
items in chronological order, the client can build an up-to-date index of every package that exists on the V3 package
source.

Each catalog item has a property called the `commitTimeStamp`. This is the time when the item was added to the catalog.
Catalog items are added to a catalog page in batches called "commits". All catalog items in the same commit have the
same commit timestamp.

In short, catalog blobs have the following hierarchical structure:

- **Index**: the entry point for the catalog.
- **Page**: a grouping of catalog items. The number of catalog items in a page is defined by server implementation.
- **Leaf**: a document representing a catalog item, which is specific to a single package version at a point in time.

In contrast to the [package metadata resource](registration-base-url-resource.md) which is indexed by package ID, the
catalog indexed by time.

Catalog items are always appended to the catalog in chronological order. This means that if a catalog commit is added
at time X then no catalog commit will ever be added with a time less than or equal to X.

The catalog index is at the following location:
```
https://example/catalog/index.json
```

The catalog index is a JSON document that contains an object with the following properties:

Name            | Type             | Required | Notes
--------------- | ---------------- | -------- | -----
commitId        | string           | yes      | A GUID associated with the most commit
commitTimeStamp | string           | yes      | A timestamp of the most recent commit
count           | integer          | yes      | The number of pages in the index
items           | array of objects | yes      | A array of objects, one object per page

As items are added to the catalog, the `commitId` will change and the `commitTimeStamp` will increase.

Each element in the `items` array is an object with some minimal details about each page. These page objects do not
contain the actual catalog items. The order of the items in this array is not defined. Pages should be ordered using
their `commitTimeStamp` property.

As new pages are introduced, the `count` will be incremented and a new object will appear in the `items` array.

### Catalog page object in the index

The catalog page objects found in the catalog index's `items` property have the following properties:

Name            | Type    | Required | Notes
--------------- | ------- | -------- | -----
@id             | string  | yes      | The URL to the catalog page
commitId        | string  | yes      | A GUID associated with the most recent commit in this page
commitTimeStamp | string  | yes      | A timestamp of the most recent commit in this page
count           | integer | yes      | The number of items in the page

## Catalog page

The catalog page is a collection of catalog items. It is a document fetched using the `@id` value found in the catalog
index. The URL to a catalog page is not intended to be predictable and should be discovered using only the catalog
index.

The catalog page document is a JSON object with the following properties:

Name            | Type             | Required | Notes
--------------- | ---------------- | -------- | -----
commitId        | string           | yes      | A GUID associated with the most recent commit in this page
commitTimeStamp | string           | yes      | A timestamp of the most recent commit in this page
count           | integer          | yes      | The number of items in the page
items           | array of objects | yes      | The catalog items in this page
parent          | string           | yes      | A URL to the catalog index

As items are added to the page, the `commitId` will change and the `commitTimeStamp` will increase.

Each element in the `items` array is an object with some minimal details about each item. These item objects do not
contain all of the catalog item's data. The order of the items in this array is not defined. Items should be ordered
using their `commitTimeStamp` property.

As new items are introduced, the `count` will be incremented and a new object will appear in the `items` array.

### Catalog item object in a page

The catalog item objects found in the catalog page's `items` property have the following properties:

Name            | Type    | Required | Notes
--------------- | ------- | -------- | -----
@id             | string  | yes      | The URL to the catalog item
@type           | string  | yes      | The type of the catalog item
commitId        | string  | yes      | A GUID associated with this catalog item
commitTimeStamp | string  | yes      | The commit timestamp of this catalog item
nuget:id        | string  | yes      | The package ID of the catalog item
nuget:version   | string  | yes      | The package version of the catalog item

Details about the possible values for `@type` are available below.

## Cursor

Since the catalog is an append-only data structure indexed by time, the client can store a "cursor" locally. This
cursor is simply a commit timestamp representing up to what point in time the client has processed catalog items. Every
time the client wants to process new events on the V3 package source, it need only query the catalog for all catalog items
with a commit timestamp greater than its stored cursor. After the client successfully processes all new catalog items,
it records the largest commit timestamp of the new items as its new cursor value.

Using this approach, the client can be sure to never miss any package events occuring on the V3 package source.
Additionally, the client never has to reprocess old events prior to the cursor's recorded commit timestamp.

When the catalog client is starting for the very first time (and therefor has no cursor value), it should use a default
cursor value of `DateTimeOffset.MinValue` or some such notion of minimum representable timestamp.