---
name: Confirm availability of funds (CoF / CBPII)
description: Register a confirmation-of-funds consent and query whether funds are available on a Triodos account, for card-based payment instrument issuers under Berlin Group NextGenPSD2.
api: openapi/triodos-bank-uk-xs2a-openapi.json
operations: [registerConsentRequest_1, createPiisAuthorisation, getPiisConsentStatus, confirmFundsAvailable]
---

# Confirm availability of funds (CoF / CBPII)

Check whether sufficient funds are available on a Triodos account, as a card-based payment instrument issuer (CBPII/PIIS).

## Prerequisites
A PIISP-role eIDAS certificate and the standard signed headers (`X-Request-ID`, `Digest`, `Signature`, `TPP-Signature-Certificate`).

## Steps
1. **Register CoF consent** — `registerConsentRequest_1` (`POST /{tenant}/v2/consents/confirmation-of-funds`) naming the account to be checked. Capture the `consentId`.
2. **Authorise** — `createPiisAuthorisation` (`POST /{tenant}/v2/consents/confirmation-of-funds/{resource-id}/authorisations`); PSU completes SCA (OIDC Authorization Code Flow, scope `openid PIIS:{consentId}`); check readiness with `getPiisConsentStatus`.
3. **Confirm funds** — `confirmFundsAvailable` (`POST /{tenant}/v1/funds-confirmations`) with the account reference and amount. The response returns `fundsAvailable: true|false`.

## Rules
- One CoF consent authorises repeated funds checks on the covered account until revoked/expired.
- `NO_PIIS_ACTIVATION` / `CARD_INVALID` in the `tppMessages` envelope indicate the account is not enabled for CoF or the instrument is invalid — see `errors/triodos-bank-uk-problem-types.yml`.
