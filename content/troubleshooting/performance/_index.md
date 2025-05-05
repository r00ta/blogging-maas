+++
date = '2025-05-05T23:27:20+01:00'
type = "chapter"
title = 'Performances'
weight = 4
+++

## Some tips to speedup your MAAS

The `maasserver_events` table is a table that stores all the events that happen in MAAS. It's a good idea to clean it regularly as it can grow quite large and it does not have any retention policy. 

My suggestion would be to keep only the latest 20% of events for every machine (see [here](https://bugs.launchpad.net/maas/+bug/2044895/comments/5) for the reference). You can do this with the following query:

```sql
WITH node_records AS (
  SELECT
    id,
    created,
    updated,
    action,
    description,
    node_id,
    ROW_NUMBER() OVER (PARTITION BY node_id ORDER BY updated DESC) AS row_num,
    COUNT(*) OVER (PARTITION BY node_id) AS total_count
  FROM maasserver_event
)
DELETE FROM maasserver_event
WHERE id IN (
  SELECT id
  FROM node_records
  WHERE row_num > CEIL(total_count * 0.2)
);
```

Expecially for large and old environments, you should see a significant performance improvement and a significant reduction in the size of the `maasserver_events` table.

## Cleanup the discoveries!

The "discoveries" entries can slow down the performance of the MAAS server when it tries to find the next IP address to assign to a machine. I'd suggest to disable and clean them regularly.

## DNS and IP Addresses cleanup

There are known bugs for which MAAS does not cleanup the DNS entries and IP addresses. There is an automatic cleanup that runs at every restart (at least up to 3.6 - the release that is currently in the stable channel). I'd suggest to restart MAAS regularly to avoid this issue. 

## Memory leaks

It's well known that MAAS can leak memory, but we haven't found the root cause yet. The only solution that we found is to restart MAAS regularly.