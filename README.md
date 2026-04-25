# Established Emails API

Email age intelligence for fraud, trust, and risk teams.

Established Emails is an API for checking whether an email address has a minimum observed age. It is designed to be used as a positive anti-fraud signal: if an address appears in the dataset, your system can treat it as having existed no later than the returned year.

It does not claim an exact account creation date, and a missing record should not be treated as proof that an email is fraudulent.

## Why Email Age Helps

Fraud teams often need lightweight signals that are fast, explainable, and easy to combine with an existing model or rules engine. Email age is useful because older observed addresses are often less disposable than brand-new signups.

Use cases include:

- Signup and onboarding risk scoring
- Checkout and payment review
- Account recovery controls
- Trial abuse and multi-account detection
- Trust and safety enrichment

The strongest use is as a positive signal: "this email has evidence of age." Absence of evidence should remain neutral unless your own model says otherwise.

## Demo

Try the demo at [establishedemails.com](https://establishedemails.com).

The demo lets security professionals enter an email address and see the normalized API response, minimum observed age wording, and beta domain classification.

## API

### Lookup An Email

```http
GET /v1/email?email=<encoded-email>
```

Example:

```bash
curl "https://establishedemails.com/v1/email?email=analyst%40example.com"
```

Example response when a record is found:

```json
{
  "found": true,
  "email": "analyst@example.com",
  "year": 2020,
  "classification": "mainstream"
}
```

Example response when no established-age record is found:

```json
{
  "found": false,
  "email": "new@example.com",
  "year": null,
  "classification": "unknown"
}
```

### Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `found` | boolean | Whether the email was found in the age dataset. |
| `email` | string | Normalized lowercase email address used for lookup. |
| `year` | number or null | Earliest observed year when `found` is true. |
| `classification` | string | Beta domain classification. Values include `mainstream`, `privacy_relay`, `disposable`, `suspicious`, and `unknown`. |

### Classification Is Beta

The `classification` field is in development. It provides lightweight domain context and should not be used as a standalone fraud decision.

Current values:

- `mainstream`: common mailbox providers such as Gmail, Outlook, Yahoo, iCloud, Proton, Fastmail, and similar domains.
- `privacy_relay`: relay providers such as Apple Hide My Email, Firefox Relay, SimpleLogin, and DuckDuckGo Email Protection.
- `disposable`: domains present in the built-in disposable email list.
- `suspicious`: lightweight keyword-based hints in the local part or domain.
- `unknown`: no known classification.

### Error Responses

| Status | Error | Meaning |
| --- | --- | --- |
| `400` | `missing_email` | The `email` query parameter was not provided. |
| `400` | `invalid_email` | The provided value could not be normalized into a valid email address. |
| `405` | `method_not_allowed` | The endpoint only supports `GET` and `HEAD`. |
| `404` | `not_found` | The requested path does not exist. |
| `500` | `internal_error` | Unexpected server error. |

Example invalid email response:

```json
{
  "error": "invalid_email",
  "input": "not-an-email"
}
```

## Interpreting The Year

The API returns a year, not a precise creation timestamp.

If the response contains:

```json
{
  "found": true,
  "year": 2020
}
```

the conservative interpretation is:

> This email was observed no later than 2020.

For minimum-age calculations, use the end of the returned year as the latest possible observation date. For example, a `year` of `2020` means "at least as old as December 31, 2020" for conservative age-floor purposes.

## Integration Example

```js
async function getEmailAgeSignal(email) {
  const url = new URL("https://establishedemails.com/v1/email");
  url.searchParams.set("email", email);

  const response = await fetch(url, {
    headers: { Accept: "application/json" },
  });

  const body = await response.json();

  if (!response.ok) {
    throw new Error(body.error || `HTTP ${response.status}`);
  }

  return body;
}
```

Suggested decision logic:

- Treat `found: true` with an older `year` as a positive trust signal.
- Treat `found: false` as "no signal", not as a negative verdict.
- Combine `classification` with your own device, velocity, payment, IP, and behavioral signals.
- Keep a human-readable audit reason such as `email_observed_since_2020`.

## Project

Established Emails is a project by [Joe Rutkowski](https://www.linkedin.com/in/joe12387/).

- Email: [Joe@dreggle.com](mailto:Joe@dreggle.com)
- GitHub: [Joe12387](https://github.com/Joe12387)
- LinkedIn: [joe12387](https://www.linkedin.com/in/joe12387/)
