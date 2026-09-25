# salvo-endpoints

Endpoint configuration for [salvo](https://github.com/TheBitGlitch/salvo).

This repository contains the endpoint configuration used by `salvo`. The
configuration is downloaded and updated with:

```bash
salvo manage sync-endpoints
```

---

## What This Repository Contains

This repository provides the endpoint configuration consumed by `salvo`.

Each endpoint entry defines the request URL, HTTP method, capacity, selection
weight, and optional request payload. The configuration allows `salvo` to
select and manage multiple endpoints at runtime.

Phone numbers must be Iranian. `salvo` accepts supported Iranian phone number
formats and normalizes them to `9123456789` before injecting the number into
the endpoint configuration.

Use `{phone}` wherever the normalized phone number should be inserted.

Sources whose value starts with `#` are ignored by `salvo`. This allows an
endpoint to be temporarily disabled without removing its configuration.

---

## Configuration

The configuration is a JSON array of endpoint objects.

### Template

```json
[
    {
        "source": "example.com",
        "url": "https://example.com/send-sms/",
        "method": "POST",
        "capacity": 50,
        "ticket": 100,
        "json": {
            "phone": "+98{phone}"
        }
    },
    {
        "source": "test.ir",
        "url": "https://test.ir/otp/",
        "method": "POST",
        "capacity": 22,
        "ticket": 64,
        "data": {
            "phone": "98-{phone}",
            "id": "0000001000000001"
        }
    },
    {
        "source": "sample.org",
        "url": "https://sample.org/send-sms/phone=?{phone}",
        "method": "GET",
        "capacity": 20,
        "ticket": 56
    },
    {
        "source": "#disabled.example",
        "url": "https://disabled.example/send-sms/",
        "method": "POST",
        "capacity": 10,
        "ticket": 50
    }
]
```

### Fields

| Field      | Required | Description                                    |
| ---------- | -------- | ---------------------------------------------- |
| `source`   | Yes      | API source identifier.                         |
| `url`      | Yes      | Target endpoint URL.                           |
| `method`   | Yes      | HTTP method: `GET` or `POST`.                  |
| `capacity` | Yes      | Maximum executions per phone number per cycle. |
| `ticket`   | Yes      | API selection weight.                          |
| `json`     | No       | JSON request payload.                          |
| `data`     | No       | Form-data request payload.                     |

The `json` and `data` fields are optional and are used according to the
specified HTTP method and endpoint requirements.

---

## Ticket Calculation

The `ticket` value represents the endpoint's selection weight. It is calculated
from the endpoint's observed success rate and relative capacity.

```text
success_rate = min(sent / requested, 1)

capacity_weight = log(capacity + 1, max_capacity + 1)

ticket = min(
    max(
        round(100 * success_rate^0.6 * capacity_weight^0.4),
        0
    ),
    100
)
```

If `requested < 1` or `max_capacity < 1`, the ticket is `0`.

The resulting value is bounded between `0` and `100`.

---

## Usage

To download or update the endpoint configuration:

```bash
salvo manage sync-endpoints
```

To force a fresh download without using the stored ETag:

```bash
salvo manage sync-endpoints --force
```

To use a different remote configuration source:

```bash
salvo manage sync-endpoints --source <URL>
```

The configuration is downloaded to the platform-specific user data directory.
`salvo` uses ETag-based caching to avoid unnecessary downloads and replaces the
configuration atomically after a successful validation.

---

## Repositories

* Source Code: [salvo](https://github.com/TheBitGlitch/salvo)
* Endpoints: [salvo-endpoints](https://github.com/TheBitGlitch/salvo-endpoints)

---

## Disclaimer

This software is provided as-is. The author assumes no liability for misuse,
damage, or legal consequences arising from its use.

Users are responsible for complying with all applicable laws, regulations,
terms of service, and obtaining appropriate authorization before sending
requests to any service.

---

## License

This repository is released under the [MIT License](LICENSE).

---

## Author

Maintained by [@TheBitGlitch](https://github.com/TheBitGlitch).
