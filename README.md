# Rose Bruno Collection

Set the `Local` environment before sending application requests. Its `baseUrl` already includes `/api`, so request paths begin with `/app`.

## Oursms provider verification

The **Oursms SMS** folder is a provider-level readiness check, separate from the local Rose API. Select the `Oursms` environment, then replace `oursmsApiKey`, `oursmsSenderId`, and `oursmsTestRecipient` with live values. The first four requests validate the provider without sending an SMS. **Send OTP Test SMS** sends a real transactional message and can use one credit; it captures the returned message/job identifiers when supplied. Run **Get Message Status** afterward to verify delivery; **Get Job Message Status** additionally requires a job ID from the send response.

Use the source address exactly as returned by **List Source Addresses**. Indian test numbers must be entered in international format without `+`, for example `919876543210`. An Indian delivery test is only meaningful after Oursms confirms a supported India route and the required Indian DLT registration/template settings are configured.

## Collection variables

| Variable | Purpose | Default |
| --- | --- | --- |
| `locale` | Localized CMS response language | `en` |
| `legalPageKey` | Legal page requested by the public CMS endpoints | `1` |
| `customerAccessToken` / `providerAccessToken` | Captured bearer tokens for their respective apps | Empty until login/registration completes |
| `customerSessionToken` / `providerSessionToken` | Captured OTP-flow sessions; never interchangeable | Empty until an OTP is requested |
| `specialtyId`, `providerId`, `serviceId`, `serviceAreaId`, `bookingId`, `reviewId` | IDs captured by requests or editable seeded fallbacks | Mixed |
| `bookingDate` | Future Riyadh date used to generate available booking slots | `2026-09-01` |
| `bookingScheduledAt`, `mapBookingScheduledAt` | First two available timestamps captured by Get Available Slots | Empty until slots are fetched |
| `bookingMapAddress`, `bookingMapLatitude`, `bookingMapLongitude` | Direct map-pin location used by the map booking example | Riyadh example |

## Required sequences

Customer registration: **Register Request OTP** → **Register Verify OTP** → **Register Update Location**. The final request captures `customerAccessToken`. Existing customers use the login request/verify pair. Provider registration: **Get Categories** → **Register Request OTP** → **Register Verify OTP** → **Register Upload Documents**. This creates a pending provider; only an active provider login returns `providerAccessToken`.

The local development API accepts OTP `123456`. Resend requests apply only to active register/login OTP sessions. Do not reuse customer variables in provider requests or vice versa.

Create Review requires an authenticated customer and an owned, completed booking that has no review. Create Review Reply requires the authenticated owner provider and a review with no existing reply.

## Customer booking sequence

1. Log in as a customer so `customerAccessToken` is populated.
2. Run **List Providers** to capture `providerId`, then **Get Provider** to capture an active `serviceId`.
3. Set `bookingDate` to a future date on which that provider is enabled, then run **Get Available Slots**. It captures the first slot in `bookingScheduledAt` and the second in `mapBookingScheduledAt`.
4. For a saved address, run **List Addresses** or **Create Address**, followed by **Create Booking - Saved Address**. Alternatively, run **Create Booking - Map Pin** with the map variables.
5. Run **Get Booking Details** with the captured `bookingId` to inspect the pending countdown and, after the configured timeout, the automatic expired/cancelled transition.

The two create requests intentionally use different captured slots. Booking bodies contain no payment method or transaction data.

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
| `1` (`topRated`) | Top rated (placeholder; sorting is not active yet) |
| `2` (`nearest`) | Nearest (placeholder; sorting is not active yet) |
| `3` (`lowestPrice`) | Lowest price (placeholder; sorting is not active yet) |
| `4` (`highestPrice`) | Highest price (placeholder; sorting is not active yet) |

### Availability

`PUT /app/provider/availability` requires all seven day names exactly once: `monday`, `tuesday`, `wednesday`, `thursday`, `friday`, `saturday`, and `sunday`. Times must be 24-hour `HH:mm`; enabled days need a start and an end time, with end after start. Valid buffer values are `15`, `30`, `45`, and `60` minutes.

### Booking lifecycle

| Field | Values |
| --- | --- |
| `status` | `pending`, `upcoming`, `in_progress`, `completed`, `cancelled` |
| `requestStatus` | `pending`, `accepted`, `rejected`, `expired` |
