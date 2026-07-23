---
name: Initiate and authorise a Triodos payment (PIS)
description: Initiate a SEPA, UK domestic, or cross-border credit transfer, drive the PSU through SCA, and submit the access token to finalise the payment under Berlin Group NextGenPSD2.
api: openapi/triodos-bank-uk-xs2a-openapi.json
operations: [initiateSepaPayment, initiateUkDomesticPayment, initiateCrossBorderPayment, createAuthorisation_2, submitAuthorisation_2, getStatus_2, getAuthorisation_2]
---

# Initiate and authorise a Triodos payment (PIS)

Initiate a payment and complete PSD2 Strong Customer Authentication with the XS2A Payment Initiation Service.

## Prerequisites
Same eIDAS certificates and signed headers (`X-Request-ID`, `Digest`, `Signature`, `TPP-Signature-Certificate`) as the AIS flow. Include `TPP-Redirect-URI` (a registered redirect URI) to start SCA.

## Steps
1. **Initiate** — pick the product operation: `initiateSepaPayment` (`POST /{tenant}/v1/payments/sepa-credit-transfers`), `initiateUkDomesticPayment`, or `initiateCrossBorderPayment`. Body carries `instructedAmount`, `debtorAccount`, `creditorAccount`, `creditorName`. Response returns `paymentId`, an authorisation sub-resource, and `_links` (`scaRedirect`, `confirmation`, `status`). `transactionStatus` starts `RCVD`.
2. **Start SCA** — the response implicitly creates an authorisation sub-resource (or call `createAuthorisation_2`). Send the PSU to `scaRedirect` with your PKCE `code_challenge`; scope is `openid PIS:{paymentId}`.
3. **Exchange the code** for an access token at `POST /auth/{tenant}/v1/token`.
4. **Finalise** — `submitAuthorisation_2` (`PUT .../authorisations/{authorisation-id}`) with `Authorization: Bearer <token>`. A successful response reads `scaStatus: finalised`.
5. **Poll status** — `getStatus_2` (`GET .../{resource-id}/status`). `PATC` means additional authorisations are required (joint/company accounts): create and authorise another sub-resource.

## Rules
- The SCA redirect URL can only be used once; a failed attempt cannot be retried on the same URL.
- Idempotency: reuse the same `X-Request-ID` for a logical retry of the same initiation. See `conventions/triodos-bank-uk-conventions.yml`.
- Errors follow the Berlin Group `tppMessages` envelope (`PAYMENT_FAILED`, `FUNDS_NOT_AVAILABLE`, `CONSENT_EXPIRED`, ...) — see `errors/triodos-bank-uk-problem-types.yml`.
