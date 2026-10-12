---
layout: post
title: "Why your brokerage penny-drop API fails at 2 AM"
date: 2026-10-12
canonical: https://www.finovo.tech/blog/penny-drop
excerpt: Penny-drop (bank account verification) looks simple until the NPCI gateway blips at 2 AM and your customer onboarding stops cold. Here is the architecture that handles it.
tags: [penny-drop, bank-verification, broker, api, npci]
---

> **Full article: [finovo.tech/blog/penny-drop](https://www.finovo.tech/blog/penny-drop)**
> This is a technical mirror. The canonical version lives at [finovo.tech](https://www.finovo.tech/).

Penny-drop is the standard way to verify an Indian bank account during KYC. Your KYC engine credits ₹1 to the customer's account via IMPS, reads back the account-holder name from the IMPS response, and matches it against the PAN/Aadhaar name. If it matches, bank account is verified.

Simple on paper. Falls apart at 2 AM.

## The failure modes

- **NPCI switch maintenance window**: typically Fri/Sat 2-4 AM IST
- **Bank-side IMPS batch delays**: some PSBs batch-process IMPS between 1-5 AM
- **ENACH SOS flag**: triggers transaction failure even on live rails
- **Name-match false negatives**: "RAMESH K." vs "Ramesh Kumar" vs "Mr. Ramesh Kumar"
- **Reversal timing**: ₹1 is credited and must be reversed; if the reversal fails the customer's balance is permanently wrong by ₹1

## What a resilient penny-drop API does

1. **Multi-provider fallback**: don't rely on one gateway — route to IMPS primary, UPI auto-collect fallback
2. **Smart retry with jitter**: back off exponentially on provider errors; no retry on hard failures
3. **Fuzzy name matching with Indian name conventions**: handle initials, honorifics, middle-name reorderings
4. **Reversal reconciliation**: nightly cron matches each ₹1 debit to a reversal credit; alerts on mismatches
5. **Status pass-through to customer**: don't silently retry — show "verifying your bank" and tell them if it's going to take 10 min at 2 AM

[Finovo's penny-drop API](https://www.finovo.tech/services#ekyc-vanilla) handles all of the above as default behaviour. We've shipped it for 55+ Indian brokers; we know what the 2 AM failure modes look like.

## Read the full architecture

- [Full technical article](https://www.finovo.tech/blog/penny-drop)
- [eKYC Vanilla](https://www.finovo.tech/services#ekyc-vanilla)
- [eKYC Enterprise with full compliance pipeline](https://www.finovo.tech/services#ekyc-enterprise)
- [Contact us for integration spec](https://www.finovo.tech/contact)
