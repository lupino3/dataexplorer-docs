---
title: Materialized views data purge - Azure Data Explorer
description: This article describes materialized views data purge in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
ms.date: 02/08/2021
---

# Materialized views data purge

[!INCLUDE [gdpr-intro-sentence](../../../includes/gdpr-intro-sentence.md)]

[Data purge](../../concepts/data-purge.md) commands can be used to purge records from materialized views. The same guidelines for purging records from a table apply to materialized views purge.

The purge command only deletes records from the *materialized part* of the view ([what is the materialized part](materialized-view-overview.md#how-materialized-views-work)?). Therefore, if the source table of the materialized view includes records to purge, these *may* be returned from the materialized view query, even after purge completed successfully.

The recommended process for purging records from a materialized view is:

1. Purge the source table of the materialized view.
1. After the source table purge is completed successfully, purge the materialized view.

> [!WARNING]
> Data deletion through the `.purge` command is designed to be used to protect personal data and should not be used in other scenarios. It is not designed to support frequent delete requests, or deletion of massive quantities of data, and may have a significant performance impact on the service.

**Limitations**

The purge predicate of a materialized view purge can only reference the group by keys of the aggregation, or any column in a [arg_max()](../../query/arg-max-aggfunction.md)/[arg_min() ](../../query/arg-min-aggfunction.md)/[any()](../../query/take-any-aggfunction.md) view. It **cannot** reference other aggregation functions result columns.

For example, for a materialized view `MV` which is defined with the following aggregation function:

```kusto
T | summarize count(), avg(Duration) by UserId
```

The following purge predicate is not valid, since it references the result of the avg() aggregation:

```kusto
MV | where avg_Duration > 1h
```