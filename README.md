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
| `specialtyId`, `providerId`, `serviceId`, `secondServiceId`, `serviceAreaId`, `bookingId`, `reviewId` | IDs captured by requests or editable seeded fallbacks | Mixed |
| `bookingDate` | Future Riyadh date used to generate available booking slots | `2026-09-01` |
| `bookingScheduledAt`, `mapBookingScheduledAt` | First two available timestamps captured by Get Available Slots | Empty until slots are fetched |
| `bookingMapAddress`, `bookingMapLatitude`, `bookingMapLongitude` | Direct map-pin location used by the map booking example | Riyadh example |
| `bookingListStatus`, `bookingListPage`, `bookingListPageSize` | Customer booking-list filter and pagination values | `upcoming`, `1`, `15` |
| `providerBookingListStatus`, `providerBookingListPage`, `providerBookingListPageSize` | Provider booking-list filter and pagination values | `upcoming`, `1`, `15` |
| `providerBookingRequestStatus` | Provider action for a pending request | `accepted` or `rejected` |
| `customerCancellationReason` / `customerCancellationComments` | Customer cancellation reason and optional details | `customer_request` / Example text |
| `providerCancellationReason` / `providerCancellationComments` | Provider cancellation reason and optional details | `personal_emergency` / Example text |
| `chatMessageId`, `chatImageMessageId`, `chatPageSize` | Booking-chat pagination/read/image identifiers | Captured / `30` |
| `chatText`, `chatCaption`, `chatImagePath` | Text and private image-message example content | Example values |
| `customerChatClientMessageId`, `providerChatClientMessageId` and image variants | Idempotency UUIDs for chat sends; change them to create another message | Example UUIDs |

## Required sequences

Customer registration: **Register Request OTP** → **Register Verify OTP** → **Register Update Location**. The final request captures `customerAccessToken`. Existing customers use the login request/verify pair. Provider registration: **Get Categories** → **Register Request OTP** → **Register Verify OTP** → **Register Upload Documents**. This creates a pending provider; only an active provider login returns `providerAccessToken`.

The local development API accepts OTP `123456`. Resend requests apply only to active register/login OTP sessions. Do not reuse customer variables in provider requests or vice versa.

Provider registration requires a display image, National ID front and back images, and a selfie-verification image. Submit the National ID images as `nationalId[]` in front-then-back order. Provider auth responses return only session/access tokens and provider ID/verification status where applicable.

Create Review requires an authenticated customer and an owned, completed booking that has no review. Create Review Reply requires the authenticated owner provider and a review with no existing reply.

## Customer booking sequence

1. Log in as a customer so `customerAccessToken` is populated.
2. Run **List Providers**, copy a marketplace-ready provider ID into the editable `providerId` Local-environment variable, then run **Get Provider** to capture two active services from the same category in `serviceId` and `secondServiceId`. If no same-category pair exists, create or activate another service before continuing.
3. Set `bookingDate` to a future date on which that provider is enabled, then run **Get Available Slots**. It calculates availability from the summed service duration, applies one provider buffer after the bundle, and captures the first slot in `bookingScheduledAt` and the second in `mapBookingScheduledAt`.
4. For a saved address, run **List Addresses** or **Create Address**, followed by **Create Booking - Saved Address**. Alternatively, run **Create Booking - Map Pin** with the map variables.
5. Run **Get Booking Details** with the captured `bookingId` to inspect the pending countdown and, after the configured timeout, the automatic expired/cancelled transition.
6. Run **Get My Bookings** to fetch the authenticated customer's paginated booking cards. The `upcoming` filter includes both pending provider requests and accepted upcoming bookings while preserving each booking's stored `status` and `requestStatus`.
7. Run **Cancel Booking** to withdraw a pending request at any time, or cancel an accepted upcoming booking before the configured customer cancellation window. The body accepts `customer_request` or `other` and optional comments.

The two create requests intentionally use different captured slots. Both send a `serviceIds` array; every selected service must be unique, active, owned by the selected provider, and belong to the same category. Booking bodies contain no payment method or transaction data.

## Provider booking sequence

1. Log in as an active, KYC-approved provider so `providerAccessToken` is populated.
2. Run **Get All Booking Requests** to fetch pending requests and capture its first `bookingId`.
3. Run **Get Booking Details** to inspect the selected request, then set `providerBookingRequestStatus` to `accepted` or `rejected` and run **Update Booking Request Status**.
4. Run **Get All Bookings** with `providerBookingListStatus` set to `upcoming`, `in_progress`, or `completed`.
5. For an accepted upcoming booking, run **Start Upcoming Booking** to change its status to `in_progress`.
6. For an upcoming booking, run **Cancel Booking**. It requires one of `personal_emergency`, `health_issue`, `equipment_or_supply_issue`, `scheduling_conflict`, `unable_to_reach_location`, or `other`; comments are optional.

Only pending, non-expired requests can be accepted or rejected. An accepted request becomes an upcoming booking; a rejected request becomes cancelled.

## Booking chat sequence

Chat is available only after a request is accepted and while its booking is `upcoming` or `in_progress`. From either app, run **Get Booking Messages**, **Send Text Message** or **Send Image Message**, and **Mark Booking Messages Read**. **Download Message Image** verifies that image bytes require the authenticated booking participant. Change the relevant client-message UUID before sending another message; repeating one UUID returns the original message without creating a duplicate.

Connect Socket.IO to `{{baseUrl}}` with `/api` removed, using namespace `/chat` and handshake auth `{ token, role }`, where `role` is `customer` or `provider`. Listen for `chat:message`, `chat:read`, and `chat:presence`. REST remains the source of truth for sending, history, read updates, and reconnect recovery. Completed or cancelled accepted bookings retain read-only history but reject new messages.

Provider details returned by **Get Provider** include a paginated `reviews` section containing active customer reviews. Add `reviewsPage` and `reviewsPageSize` to the request URL when more results are needed; their defaults are `1` and `10`.

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

Get My Bookings accepts `upcoming`, `in_progress`, `completed`, or `cancelled` as `bookingListStatus`. Remove the `status` query parameter from the request URL to fetch every customer-facing booking group.
