---
name: Quote and place an Onboard exchange trade
description: Find the best offer and price for a trading pair, then create and track an on/off-ramp order.
api: openapi/nestcoin-external-gateway-openapi-original.json
operations: [getTradeOptions, extFindOffers, getBestOfferExchange, getPrice, createOrder, getExchangeOrderByIdExternal]
---

# Quote and place an Onboard exchange trade

Use this skill to price a crypto/fiat on- or off-ramp trade through Onboard Connect and create the
order, then track it to a terminal state.

## Prerequisites
- Authenticated user session (`x-auth-token`) — see `nestcoin-user-authentication.md`.
- Application `x-api-key`.

## Steps
1. **Discover trade options** — call `getTradeOptions` to see supported pairs, networks, and payment
   channels for the user.
2. **Find offers** — call `extFindOffers` for the trading pair, or `getBestOfferExchange` to get the
   single best offer directly.
3. **Get a price** — call `getPrice` with the chosen offer/amount to obtain the executable rate.
4. **Create the order** — call `createOrder` with the offer, amount, and beneficiary/wallet details.
5. **Track it** — poll `getExchangeOrderByIdExternal`, or (preferred) subscribe to the
   `order.state_transition` webhook (see `asyncapi/nestcoin-webhooks.yml`) to follow the order
   through `INITIATED -> DEPOSITED -> CONFIRMED -> COMPLETED` (or `CANCELLED`/`IN_DISPUTE`).

## Rules
- Prices are time-sensitive; re-quote if the user stalls.
- Verify webhook authenticity with the `x-signature` / `x-timestamp` HMAC-SHA256 headers.
- Handle 400/404/409 per `errors/nestcoin-problem-types.yml`; there is no idempotency key, so
  reconcile by order id rather than re-issuing `createOrder`.
