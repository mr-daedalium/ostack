# Provisioning: Stripe (Conditional — Paid Projects Only)

**Skip this step if:** Project is free (no Stripe).

Uses Stripe MCP tools (mcp__plugin_stripe_stripe__*). Always in **test mode**.

---

## Step 1: Verify Stripe account

Call `mcp__plugin_stripe_stripe__get_stripe_account_info` to verify the Stripe connection is active.

If not connected, inform the user:
> Stripe MCP is not connected. Connect it in your Claude Code settings, then re-run /startup.

## Step 2: Create product

Call `mcp__plugin_stripe_stripe__create_product` with:
- `name`: the project name
- `description`: the project description

Store the `product.id` (format: `prod_xxx`).

## Step 3: Create price

Based on the billing model from the architecture conversation:

Call `mcp__plugin_stripe_stripe__create_price` with:
- `product`: the product ID from Step 2
- `unit_amount`: price in cents (ask user during architecture if not decided)
- `currency`: `eur` (or based on geography)
- `recurring.interval`: `month` or `year` (based on architecture decision)

For seat-based pricing, set `billing_scheme: "per_unit"`.

Store the `price.id` (format: `price_xxx`).

## Step 4: Create webhook endpoint

Call `mcp__plugin_stripe_stripe__create_payment_link` — actually, for webhooks, use the Stripe API directly via curl since MCP may not have a webhook creation tool:

The webhook endpoint URL is: `https://{domain}/api/webhooks/stripe`
(or Cloud Run URL if no domain)

Events to listen for:
- `checkout.session.completed`
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.payment_succeeded`
- `invoice.payment_failed`

**Note:** If the Stripe MCP doesn't support webhook endpoint creation, log this as a manual step:
> Stripe webhook: Create manually at https://dashboard.stripe.com/test/webhooks
> Endpoint URL: https://{domain}/api/webhooks/stripe
> Events: checkout.session.completed, customer.subscription.*, invoice.payment_*

## Step 5: Store values

- Store in `.env`:
  - `STRIPE_SECRET_KEY` — from Stripe dashboard (test mode key: `sk_test_xxx`)
  - `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` — `pk_test_xxx`
  - `STRIPE_WEBHOOK_SECRET` — `whsec_xxx` (from webhook creation)
- Store in `project.json` under `services.stripe`:
  - `mode`: "test"
  - `productId`: from Step 2
  - `priceId`: from Step 3
  - `webhookEndpointId`: from Step 4 (if created)
  - `customerPortalEnabled`: true

**Important:** The Stripe MCP tools work in the context of the connected Stripe account. The API keys themselves may need to be copied from the Stripe dashboard. Inform the user:
> Stripe product and price created in test mode. You'll need to copy your test API keys from https://dashboard.stripe.com/test/apikeys into your .env file.

## Output

Log:
- Stripe product: created (ID: {product-id})
- Stripe price: created (ID: {price-id})
- Stripe webhook: created / manual setup needed
- API keys: user must copy from Stripe dashboard
