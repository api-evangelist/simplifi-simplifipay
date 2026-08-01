---
name: Apply spend controls to a SimpliFi card
description: Create a rule group (authorization, velocity, merchant controls) and apply it to a card.
api: openapi/simplifi-simplifipay-openapi.yml
operations: [loginToGenerateJwtToken, createRuleGroupOnCard, getRuleGroupsOnCard, applyRuleGroupOnCard, removeRuleGroupOnCard]
---

# Apply spend controls to a SimpliFi card

SimpliFi controls come in three flavors: authorization controls (channel
allow/deny — Online, POS, ATM, cross-border), velocity controls (rolling-window
limits), and merchant controls (MCC/merchant allow/block lists).

## Steps

1. **Authenticate** — `loginToGenerateJwtToken`; Bearer JWT + `requestUuid`.
2. **Create a rule group** — `createRuleGroupOnCard` (`POST /v1/card/{uuid}/rule-group`) with transaction/merchant/load/spend restrictions.
3. **List rule groups** — `getRuleGroupsOnCard` (`GET /v1/card/{uuid}/rule-group`) to find the `ruleGroupUuid`.
4. **Apply** — `applyRuleGroupOnCard` (`POST /v1/card/{uuid}/rule-group/{ruleGroupUuid}`). Wait for the `CARD_RULE_GROUP_CREATION` webhook.
5. **Remove when done** — `removeRuleGroupOnCard` (`DELETE /v1/card/{uuid}/rule-group/{ruleGroupUuid}`).

## Rules
- Rule groups also exist at the funding-account/wallet level (`createRuleGroupOnFundingAccount`) — card-level controls layer on top.
- Reconcile on the `CARD_RULE_GROUP_CREATION` / `CARD_RULE_GROUP_DELETION` webhooks by `requestUuid`.
