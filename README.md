# Rose Bruno Collection

Set the `Local` environment before sending application requests. Its `baseUrl` already includes `/api`, so request paths begin with `/app`.

## Collection variables

| Variable | Purpose | Default |
| --- | --- | --- |
| `locale` | Localized CMS response language | `en` |
| `legalPageKey` | Legal page requested by the public CMS endpoints | `1` |
| `customerAccessToken` / `providerAccessToken` | Captured bearer tokens for their respective apps | Empty until login/registration completes |
| `customerSessionToken` / `providerSessionToken` | Captured OTP-flow sessions; never interchangeable | Empty until an OTP is requested |
| `specialtyId`, `serviceId`, `serviceAreaId`, `reviewId` | Editable seeded fallbacks; captured dynamically where a response exposes them | Local seed values |

## Required sequences

Customer registration: **Register Request OTP** → **Register Verify OTP** → **Register Update Location**. The final request captures `customerAccessToken`. Existing customers use the login request/verify pair. Provider registration: **Get Categories** → **Register Request OTP** → **Register Verify OTP** → **Register Upload Documents**. This creates a pending provider; only an active provider login returns `providerAccessToken`.

The local development API accepts OTP `123456`. Resend requests apply only to active register/login OTP sessions. Do not reuse customer variables in provider requests or vice versa.

Create Review requires an authenticated customer and an owned, completed booking that has no review. Create Review Reply requires the authenticated owner provider and a review with no existing reply.

## Enums

### Locale

| Value | Language |
| --- | --- |
| `en` | English |
| `ar` | Arabic |

### Legal page key

| Value | Page | Customer app | Provider app |
| --- | --- | --- | --- |
| `1` | Terms of Service | Yes | Yes |
| `2` | Privacy Policy | Yes | Yes |
| `3` | Cookie Policy | Yes | Yes |
| `4` | Provider Agreement | No | Yes |

### Customer and provider gender

| Value | Meaning |
| --- | --- |
| `1` | Male |
| `2` | Female |
| `3` | Other |

### Service gender

| Value | Meaning |
| --- | --- |
| `0` | Both |
| `1` | Men |
| `2` | Women |

### Provider explore sort order

| Value | Meaning |
| --- | --- |
| `1` (`topRated`) | Highest-rated providers first |
| `2` (`nearest`) | Nearest providers first |
| `3` (`lowestPrice`) | Lowest active-service price first |
| `4` (`mostReviewed`) | Most-reviewed providers first |

### Availability

`PUT /app/provider/availability` requires all seven day names exactly once: `monday`, `tuesday`, `wednesday`, `thursday`, `friday`, `saturday`, and `sunday`. Times must be 24-hour `HH:mm`; enabled days need a start and an end time, with end after start. Valid buffer values are `15`, `30`, `45`, and `60` minutes.
