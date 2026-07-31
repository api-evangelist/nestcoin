---
name: Authenticate an Onboard Connect user
description: Log a user in with phone/email OTP and obtain the session token used for authenticated Onboard Connect calls.
api: openapi/nestcoin-external-gateway-openapi-original.json
operations: [requestAccessToken, initiateUserLogin, verifyAuthOtp, resendAuthOtp, refreshAccessToken]
---

# Authenticate an Onboard Connect user

Use this skill to authenticate an end user against the Onboard External API Gateway and get the
`x-auth-token` session token that authenticated endpoints require.

## Prerequisites
- An Onboard `x-api-key` (generate it in the Business Dashboard after approval).
- The API secret, used to compute the HMAC `x-signature` for sensitive operations.

## Steps
1. **Get an application access token** — call `requestAccessToken` (`POST /auth/oauth/access-token`)
   with your `x-api-key`. Refresh it later with `refreshAccessToken`.
2. **Initiate user login** — call `initiateUserLogin` with the user's phone/email. This is a
   sensitive operation: sign it with HMAC-SHA256 and send `x-api-key`, `x-signature`, and
   `x-timestamp` (timestamp must be within 30 seconds of server time or you get 403).
3. **Verify the OTP** — call `verifyAuthOtp` with the code the user received. On success you receive
   the user's `x-auth-token`.
4. **Resend if needed** — if the code expired, call `resendAuthOtp` before retrying step 3.
5. **Use the session** — send `x-auth-token` (as `Authorization: Bearer <token>`) on all
   authenticated user-session endpoints.

## Rules
- HMAC signature = HMAC-SHA256 over `x-timestamp` + request body, keyed with your API secret.
- Handle 401 (bad/missing credentials) and 403 (not permitted / stale timestamp) per
  `errors/nestcoin-problem-types.yml`.
- No idempotency key is supported; do not blindly retry writes.
