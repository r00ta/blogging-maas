+++
date = '2025-03-18T14:27:20+01:00'
type = "chapter"
title = 'Initialization'
weight = 1
+++

## Common issues when you are initializing MAAS

### Snap
#### Special chars in the postgres password

If you have special characters in the postgres password, you have to properly [encode](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1) them in order to be able to initialize MAAS. For example, if you have a password like `G@oodPassword!`, then you have to encode it as `G%40oodPassword%21`.

So the command to run is:

```bash
sudo maas init region+rack --database-uri "postgres://maas:G%40odPassword%21@localhost/maasdb"
```

