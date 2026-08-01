---
name: Issue and activate a SimpliFi card
description: Authenticate, create a cardholder, issue a virtual or physical card under a card program, and activate it — handling SimpliFi's async webhook model.
api: openapi/simplifi-simplifipay-openapi.yml
operations: [loginToGenerateJwtToken, createAUser, createACard, activateACard, getCardDetails]
---

# Issue and activate a SimpliFi card

SimpliFi is asynchronous: write calls return `202 Accepted` on receipt and the
final result arrives on your configured webhook endpoint. Do not treat a 2xx as
completion — wait for the matching webhook (correlate on `requestUuid`).

## Steps

1. **Authenticate** — `loginToGenerateJwtToken` (`POST /v1/auth/login/{companyName}`).
   Send `client_id`, `client_secret`, `grant_type=client_credentials` as
   `application/x-www-form-urlencoded`. Use the returned JWT as
   `Authorization: Bearer <token>` on every later call. Generate a unique
   `requestUuid` (20-40 chars) header for every request.
2. **Create the cardholder** — `createAUser` (`POST /v1/user`). Capture the
   returned user `uuid`. Wait for the `CARD_HOLDER_CREATION` webhook.
3. **Issue the card** — `createACard` (`POST /v1/card`) with `userUuid`,
   `cardProgramUuid`, and `instrument` (`VIRTUAL` | `PHYSICAL` | `VIRTUAL_AND_PHYSICAL`).
   Wait for the `CARD_ISSUANCE` webhook; capture the card `uuid`.
4. **Activate** — `activateACard` (`POST /v1/card/{uuid}/activate`). Wait for the
   `CARD_ACTIVATION` webhook.
5. **Confirm** — `getCardDetails` (`GET /v1/card/{uuid}`) to read `cardStatus`
   (expect `ISSUED` then `ACTIVATED`) and the `maskedPan`.

## Rules
- Verify every webhook: `X-SimpliFi-Webhook-Signature` = Base64(HmacSHA256(`X-SimpliFi-Webhook-Timestamp` + raw-body, pre-shared-key)). The body is `application/text`; never reformat it before verifying.
- On `requestStatus: FAILURE`, read `errorCode`/`errorMessage` (see errors/simplifi-simplifipay-error-codes.yml) and `sourceErrorCode` for the processor reason.
- Never render PAN/CVV yourself — use the Virtual Card SDK (components/simplifi-simplifipay-components.yml) with an `SDK_ADMIN` token.
