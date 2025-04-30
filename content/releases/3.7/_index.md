+++
title = "3.7 (upcoming)"
type = "chapter"
date = '2025-04-30T00:06:05+01:00'  
+++

MAAS 3.7 is supposed to be released in Q3 2025. There are a bunch of new features, such as

## User facing features

- support for DPU
- CLI startup speedup (2x). This will not impact the time needed to fetch data from a remote server that might still be slow. It impacts the time needed to start the CLI (from 2 seconds to less than 1 second). See https://r00ta.com/posts/speeding-up-maas-cli/
- New boot source selection page in the UI 

## Non-user facing features

- DHCP is now configured with temporal workflows
- DNS is now configured with temporal workflows
- DNS recursive resolver in the racks has been reimplemented in GO (before it was using BIND).
- No more django migrations. We use Alembic.
- Lots of rewrite from django to the servicelayer.
- Lots of V3 endpoint, still in alpha (not advertised!)
- UI moved many websocket handlers to the V3 api. 