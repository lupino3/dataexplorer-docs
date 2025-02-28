---
title: Request rate limits enforcement policy - Azure Data Explorer
description: This article describes the request rate limits enforcement policy in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: yonil
ms.service: data-explorer
ms.topic: reference
ms.date: 05/09/2021
---
# Request rate limits enforcement policy

A workload group's request rate limits enforcement policy controls how request rate limits are enforced.

## The policy object

A request rate limit policy has the following properties:

| Name                     | Supported values                            | Default value | Description                                   |
|--------------------------|---------------------------------------------|----------------|-----------------------------------------------|
| QueriesEnforcementLevel  | `Cluster`, `QueryHead`                      | `QueryHead`   | Indicates the enforcement level for queries.  |
| CommandsEnforcementLevel | `Cluster`, `Database`                       | `Database`    | Indicates the enforcement level for commands. |

### Request rate limits enforcement level

Request rate limits can be enforced at one of the following levels:

* `Cluster`:
  * Rate limits are enforced by the single cluster admin node.
* `Database`:
  * Rate limits are enforced by the database admin node that manages the database the request was sent to.
  * If there are multiple database admin nodes, the configured rate limit is effectively multiplied by the number of database admin nodes.
* `QueryHead`:
  * Rate limits for *queries* are enforced by the query head node that the query was routed to.
  * This option affects queries that are sent with either strong or weak [query consistency](../concepts/queryconsistency.md).
    * Strongly consistent queries run on the database admin node, and the configured rate limit is effectively multiplied by the number of database admin nodes.
    * For weakly consistent queries, the configured rate limit is effectively multiplied by the number of query head nodes.
  * This option doesn't apply to *control commands*.

> [!NOTE]
> * If the policy is undefined (`null`), the default enforcement level applies to both commands and queries.
> * Rate limits for cluster-scoped commands are always enforced at the cluster level, regardless of the value configured in the policy. For example: control commands that manage cluster-level policies.

## Examples

### Setup:

* The cluster has 10 nodes as follows:
    * one cluster admin node.
    * two database admin nodes (each manages 50% of the cluster's databases).
    * 50% of the tail nodes (5 out of 10) can serve as query heads for weakly consistent queries.
* The `default` workload group is defined with the following policies:

    ```json
    "RequestRateLimitPolicies": [
        {
            "IsEnabled": true,
            "Scope": "WorkloadGroup",
            "LimitKind": "ConcurrentRequests",
            "Properties": {
                "MaxConcurrentRequests": 200
            }
        }
    ],
    "RequestRateLimitsEnforcementPolicy": {
        "QueriesEnforcementLevel": "QueryHead",
        "CommandsEnforcementLevel": "Database"
    }
    ```

### Effective rate limits

The effective rate limits for the `default` workload group are:
   * The maximum number of concurrent *cluster-scoped control commands* is `200`.
   * The maximum number of concurrent *database-scoped control commands* is <br>`2` (database admin nodes) x `200` (max per admin node) = `400`.
   * The maximum number of concurrent *strongly consistent queries* is <br>`2` (database admin nodes) x `200` (max per admin node) = `400`.
   * The maximum number of concurrent *weakly consistent queries* is <br>`5` (query heads) x `200` (max per query head) = `1000`.

## Control commands

Manage the workload group's request concurrency policy with [Workload groups control commands](./show-workload-group-command.md).