# realm-business-vocabulary

The words a business uses for its customers, whatever product it keeps them in.

This realm fetches nothing and answers nothing. It declares ten parent types — a customer
account, a contact, a note, an opportunity, a support case, a message, a billing customer, a
subscription, an invoice, a payment attempt — and the rules a child type agrees to when it
claims one of them as a parent. Realms over real products do the rest.

## Why it exists

A view that says `MATCH (c:ChatwootConversation)` stops working the day the helpdesk is
replaced. A view that says `MATCH (c:SupportCase)` does not, because a realm's labels are
global and a child carries its parents' labels physically:

```yaml
# in realm-chatwoot                     # in realm-zammad
- name: ChatwootConversation            - name: ZammadTicket
  parents: [SupportCase]                  parents: [SupportCase]
```

```cypher
MATCH (c:SupportCase) WHERE c.status = 'open' RETURN c          // whichever is installed
MATCH (c:Chatwoot:SupportCase) RETURN c                         // one product, by its realm label
```

Anything built on the parents — views, DERIVE rules, natural-language questions, apps —
is written once and names no vendor.

## What a child agrees to

Inheriting the property names is the easy half. The contract is that they MEAN the same
thing. It is stated in full at the top of `types/vocabulary.yml`; in short:

| | |
|---|---|
| `accountKey` | the customer's registrable domain, lower case. The only cross-product join key. Empty when unknown; never a name or an id. |
| money | major units as a number, `currency` beside it |
| dates | ISO `YYYY-MM-DD`, and the BUSINESS date, not the row's creation time |
| enumerations | exactly the listed words; a source with more states folds them in |
| `sourceUrl` | opens this record in its own product |

No parent has an identity. Identity is the child's, because only the product knows what
makes its records unique, and two products' ids must never be compared.

## Relationship names

Part of the vocabulary, though nothing here declares them: a product realm contributes its
own joins, and uses these names so that a traversal reads the same over any product.

| From | Relationship | To |
|---|---|---|
| `CustomerAccount` | `HAS_ACCOUNT_CONTACT` | `AccountContact` |
| `CustomerAccount` | `HAS_NOTE` | `AccountNote` |
| `CustomerAccount` | `HAS_OPPORTUNITY` | `Opportunity` |
| `CustomerAccount` | `HAS_CASE` | `SupportCase` |
| `SupportCase` | `HAS_MESSAGE` | `SupportMessage` |
| `CustomerAccount` | `BILLED_AS` | `BillingCustomer` |
| `BillingCustomer` | `HAS_SUBSCRIPTION` | `BillingSubscription` |
| `BillingCustomer` | `HAS_INVOICE` | `BillingInvoice` |
| `BillingInvoice` | `HAS_ATTEMPT` | `PaymentAttempt` |

`HAS_ACCOUNT_CONTACT` and not `HAS_CONTACT`: the host already writes `HAS_CONTACT` when it
resolves a record onto its Person spine, and a vocabulary that reuses a host relationship
name for something else has made every traversal of it ambiguous.

## Depending on it

A realm cannot yet declare that it needs another realm (embabel/me#495, #1168). Until it
can: install this realm first. A child whose parent is not installed is a type with a parent
the world has never heard of.
