lila-push
=========

Web push microservice for [lichess.org](https://lichess.org).

Setup
-----

1. Generate a private key.

   ```
   openssl ecparam -name prime256v1 -genkey -noout -out private.pem
   ```

2. Print public key and set `push.web.vapid_public_key` in lila configuration:

   ```
   openssl ec -in private.pem -pubout -outform DER | tail -c 65 | base64 | tr -d '\n' && echo
   ```

3. Run microservice:

   ```
   cargo run --release -- --vapid private.pem --subject mailto:contact@lichess.org
   ```

Usage
-----

```
lila-push 0.1.0
Niklas Fiekas <niklas.fiekas@backscattering.de>
Web push microservice for lichess.org

USAGE:
    lila-push [OPTIONS] --subject <subject> --vapid <vapid>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
        --bind <address:port>  Listen on this address and port [default: 127.0.0.1:9054]
        --subject <subject>    VAPID subject (example: mailto:contact@lichess.org)
        --vapid <vapid>        PEM file with private VAPID key
```

HTTP API
--------

### `POST /` send a push message

```javascript
{
  "subs": [{
    "endpoint": "https://fcm.googleapis.com/fcm/send/dpH5lCsTSSM:APA91bHqjZxM0VImWWqDRN7U0a3AycjUf4O-byuxb_wJsKRaKvV_iKw56s16ekq6FUqoCF7k2nICUpd8fHPxVTgqLunFeVeB9lLCQZyohyAztTH8ZQL9WCxKpA6dvTG_TUIhQUFq_n",
    "keys": {
      "p256dh": "BLQELIDm-6b9Bl07YrEuXJ4BL_YBVQ0dvt9NQGGJxIQidJWHPNa9YrouvcQ9d7_MqzvGS9Alz60SZNCG3qfpk=",
      "auth": "4vQK-SvRAN5eo-8ASlrwA=="
    }
  }],
  "payload": "lorem ipsum", // could be json encoded as a string
  "ttl": 43200 // 12 hour limit to deliver
}
```

```javascript
{
  "https://fcm.googleapis.com/fcm/send/dpH5lCsTSSM:AP...": "ok"
}
```

response | description
--- | ---
ok | **success**
invalid_uri |
invalid_ttl |
missing_crypto_keys |
invalid_crypto_keys |
unauthorized |
endpoint_not_found |
endpoint_not_valid | Subscription is invalid and will never be valid
payload_too_large |
not_implemented | Endpoint does not support encryption algorithm
tls_error | Endpoint misbehaving
ssl_error | ...
io_error | ...
invalid_response | ...
other | ...
unspecified | ...

License
-------

lila-push is licensed under the GNU Affero General Public License 3.0 (or any
later version at your option). See COPYING for the full license text.
