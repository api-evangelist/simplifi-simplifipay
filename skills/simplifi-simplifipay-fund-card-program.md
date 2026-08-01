---
name: Create and fund a SimpliFi card program
description: Stand up a card program, link a funding source, raise funding, and load a card — the money-movement path.
api: openapi/simplifi-simplifipay-openapi.yml
operations: [loginToGenerateJwtToken, createCardProgram, linkFundingSourceToCardProgram, raiseFundingDocumentUpload, raiseFunding, getBalanceOfFundingSource, loadACard]
---

# Create and fund a SimpliFi card program

## Steps

1. **Authenticate** — `loginToGenerateJwtToken` (`POST /v1/auth/login/{companyName}`); Bearer JWT + `requestUuid` on all calls.
2. **Create the program** — `createCardProgram` (`POST /v1/card-program`). Wait for the `CARD_PROGRAM_CREATION` webhook; capture the program `uuid`.
3. **Link a funding source** — `linkFundingSourceToCardProgram` (`POST /v2/card-program/{uuid}/funding-source`).
4. **Attach proof of funds** — `raiseFundingDocumentUpload` (`POST /v1/document/upload`) before raising funds, then `raiseFunding` (`POST /v1/card-program/raise-funding`). Wait for the `RAISE_FUNDING_SOURCE` webhook.
5. **Check balance** — `getBalanceOfFundingSource` (`GET /v1/card-program/{uuid}/funding-source-balance`).
6. **Load a card** — `loadACard` (`POST /v1/card/{uuid}/load`). Wait for the `CARD_LOAD` webhook.

## Rules
- All money movement is async — reconcile on webhooks (`FUNDS_TRANSFER`, `RAISE_FUNDING_SOURCE`, `CARD_LOAD`), correlating by `requestUuid`.
- Amounts cannot be negative (error 10035); required fields missing → 10003.
- Verify webhook signatures (HmacSHA256, see conventions/simplifi-simplifipay-conventions.yml).
