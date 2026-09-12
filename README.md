# salvo-endpoints

Endpoint configuration for [salvo](https://github.com/TheBitGlitch/salvo).

This repository contains the endpoint configuration used by `salvo`. The configuration is downloaded and updated with:

```bash
salvo manage sync-endpoints
```

## Configuration

The configuration is a JSON array of endpoint objects.

Phone numbers must be Iranian. `salvo` accepts supported Iranian phone number formats and normalizes them to `9123456789` before injecting the number into the endpoint configuration.

Use `{phone}` wherever the normalized phone number should be inserted.

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
| `json`     | No       | Request payload.                               |
| `data`     | No       | Form-data request payload.                     |


### Ticket Calculation

$$
success\_rate = \min\left(\frac{sent}{requested}, 1\right)
$$

$$
capacity\_weight =
\log_{max\_capacity + 1}(capacity + 1)
$$

$$
ticket =
\operatorname{clamp}
\left(
\operatorname{round}
\left(
100 \times success\_rate^{0.6}
\times capacity\_weight^{0.4}
\right),
0,
100
\right)
$$

If `requested < 1` or `max_capacity < 1`, the ticket is `0`.

## Repository

* Main project: [salvo](https://github.com/TheBitGlitch/salvo)
