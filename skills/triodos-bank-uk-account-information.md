---
name: Access Triodos account information (AIS)
description: Register an AIS consent, drive the PSU through SCA, then read accounts, balances, and transactions under the Berlin Group NextGenPSD2 standard.
api: openapi/triodos-bank-uk-xs2a-openapi.json
operations: [registerConsentRequest, createAisAuthorisation, getAisConsentStatus, getAccounts, getAccount, getBalances, getTransactions]
---

# Access Triodos account information (AIS)

Read a Triodos PSU's accounts, balances, and transactions with a consented XS2A Account Information Service flow.

## Prerequisites
- Registered TPP with a valid eIDAS QWAC (mutual-TLS) and QSEALC (message signing) certificate. In the sandbox (`https://xs2a-sandbox.triodos.com`) neither mTLS nor eIDAS signing is required.
- Every XS2A request must carry `X-Request-ID` (UUID), a `Digest` (`SHA-256=...`), a `Signature` over `digest x-request-id` (`rsa-sha256`), and `TPP-Signature-Certificate`. See `conventions/triodos-bank-uk-conventions.yml`.

## Steps
1. **Register consent** — `registerConsentRequest` (`POST /{tenant}/v1/consents`) with the account access requested. Use Detailed Consent (supply the IBAN) or Bank Offered Consent (let the PSU pick). Capture the returned `consentId`.
2. **Start SCA** — `createAisAuthorisation` (`POST /{tenant}/v1/consents/{resource-id}/authorisations`). Follow the `scaRedirect` link: OIDC Authorization Code Flow with PKCE (`code_challenge_method=S256`), scope `openid AIS:{consentId}` (+ `offline_access` for recurring access).
3. **Exchange the code** for an access token at `POST /auth/{tenant}/v1/token` (HTTP Basic client auth; access token TTL 600s; refresh via `offline_access`).
4. **Confirm consent** — `getAisConsentStatus` (`GET /{tenant}/v1/consents/{resource-id}/status`) should read `valid`.
5. **Read data** — `getAccounts`, then `getAccount`, `getBalances`, `getTransactions` for each account, passing the OAuth2 access token.

## Rules
- Recurring consent: max 4 accesses/day in unattended mode (omit `PSU-IP-Address`); unlimited attended. Exceeding it returns `429` / `ACCESS_EXCEEDED`.
- Errors follow the Berlin Group `tppMessages` envelope — see `errors/triodos-bank-uk-problem-types.yml`.
- Paginate transactions via the response `_links.next` edge token; do not use the deprecated `getTransactionsPage`.
