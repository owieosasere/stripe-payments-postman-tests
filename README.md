# Stripe Payments Sandbox: Postman Test Collection

## What this is

A Postman collection that tests the main steps of a card payment against Stripe's API: creating a customer, taking a payment, declining a card, refunding, and repeating a request safely. It runs only in Stripe's **test mode**. No real money, cards or customers are involved.

Each request has automatic checks, so the whole collection can be run in one go and reports pass or fail. Last run: **18 of 18 checks passed**.

## What I tested

1. **Create customer**: a customer is created and the response has an id starting with `cus_`.
2. **Create payment intent**: a £20.00 payment with Stripe's test Visa succeeds and the amount is correct.
3. **Retrieve payment intent**: fetching the payment returns the same payment with status `succeeded`.
4. **Decline test**: Stripe's test decline card returns `402` with the code `card_declined`.
5. **Refund payment**: the successful payment is refunded in full and the refund status is `succeeded`.
6. **Idempotency test**: the same payment request is sent twice with the same `Idempotency-Key`, and Stripe returns the same payment instead of charging twice.
7. **Add test balance**: a payment made with Stripe's test card that puts funds straight into the available balance.
8. **Get balance**: the balance can be read and returns a balance object.

## What I learned

- A declined card is not a `200` response. Stripe returns `402` with an error code, so a test for a decline should expect the failure, not treat it as a bug.
- A payment can only be refunded once. A second refund on the same payment is rejected with `charge_already_refunded`, so tests that refund need a fresh payment each run.
- An idempotency key must be used with the same request. Reusing a key with different parameters is rejected, and reusing it with the same parameters returns the original payment. That is how a retry after a timeout avoids a double charge. A fresh key is generated on every run so the tests can be repeated.

## Known limitations

- **Payouts are not covered.** A payout needs a linked bank account. The sandbox rejected Stripe's test bank details for this account, so I documented the error and left the payout request out of the run.
- Test values such as test cards can change. Check Stripe's testing page if a request starts failing.

## How to run it

1. Import the collection into Postman.
2. Create a Postman environment with two variables:
   - `base_url` = `https://api.stripe.com/v1`
   - `secret_key` = your own Stripe **test** secret key (it starts with `sk_test_`)
3. Select that environment at the top right of Postman.
4. Open the collection, choose **Run collection** and click **Run**. Leave `Create payout` unticked.

Never put a live key (`sk_live_`) in this collection, and never share your environment file.

## How this connects to my work

As a product owner I tested payment provider integrations in sandbox before launch: refunds, declines, wallet payments and retries. This collection rebuilds that method on Stripe's public test mode, so it can be shared and reproduced by anyone.
