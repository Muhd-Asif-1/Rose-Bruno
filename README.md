# Rose Bruno Collection

Set the `Local` environment before sending requests. Its `baseUrl` already includes `/api`, so request paths begin with `/app` or `/admin`.

## Collection variables

| Variable | Purpose | Default |
| --- | --- | --- |
| `locale` | Localized CMS response language | `en` |
| `legalPageKey` | Legal page requested by the public CMS endpoints | `1` |
| `gender` | Customer registration gender | `1` |

## Enums

### Locale

| Value | Language |
| --- | --- |
| `en` | English |
| `ar` | Arabic |

### Legal page key

Used by `GET /app/customer/legal-pages` and `GET /app/provider/legal-pages`.

| Value | Page | Customer app | Provider app |
| --- | --- | --- | --- |
| `1` | Terms of Service | Yes | Yes |
| `2` | Privacy Policy | Yes | Yes |
| `3` | Cookie Policy | Yes | Yes |
| `4` | Provider Agreement | No | Yes |

The requested page is returned only when it is active, not deleted, and published for the selected locale.

### Gender

Used by customer/provider registration and provider services.

| Value | Meaning |
| --- | --- |
| `1` | Male |
| `2` | Female |
| `3` | Other |

### Provider verification status

Used by the dashboard provider verification-status requests.

| Value | Meaning |
| --- | --- |
| `0` | Pending KYC |
| `1` | Accepted / active |
| `2` | Rejected |
| `3` | Suspended |

### Provider explore sort order

Used by `GET /app/customer/explore/providers`.

| Value | Meaning |
| --- | --- |
| `topRated` | Highest-rated providers first |
| `nearest` | Nearest providers first |
| `lowestPrice` | Lowest starting price first |
| `mostReviewed` | Most-reviewed providers first |

### FAQ audience

FAQ audiences are configured in the dashboard CMS and are filtered automatically by the public endpoints.

| Value | Audience | Returned by |
| --- | --- | --- |
| `1` | Customer app and website | Customer FAQ endpoint |
| `2` | Website only | Neither app endpoint |
| `3` | Customer app | Customer FAQ endpoint |
| `4` | Provider app | Provider FAQ endpoint |
