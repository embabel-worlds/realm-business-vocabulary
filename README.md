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
| `CustomerAccount` | `TRACKED_AS` | `CrmAccount` |
| `CrmAccount` | `HAS_ACCOUNT_CONTACT` | `AccountContact` |
| `CrmAccount` | `HAS_OPPORTUNITY` | `Opportunity` |
| `CrmAccount` | `HAS_NOTE` | `AccountNote` |
| `AccountContact` | `HAS_NOTE` | `AccountNote` |
| `Opportunity` | `HAS_NOTE` | `AccountNote` |
| `FollowUp` | `FOLLOW_UP_ON` | `CrmAccount` |
| `Meeting` | `MEETING_WITH` | `CrmAccount` |
| `CrmAccount` | `SUBSIDIARY_OF` | `CrmAccount` |
| `CrmAccount` | `HAS_SUBSIDIARY` | `CrmAccount` |
| `CustomerAccount` | `HAS_CASE` | `SupportCase` |
| `SupportCase` | `HAS_MESSAGE` | `SupportMessage` |
| `CustomerAccount` | `BILLED_AS` | `BillingCustomer` |
| `BillingCustomer` | `HAS_SUBSCRIPTION` | `BillingSubscription` |
| `BillingCustomer` | `HAS_INVOICE` | `BillingInvoice` |
| `BillingInvoice` | `HAS_ATTEMPT` | `PaymentAttempt` |

`HAS_ACCOUNT_CONTACT` and not `HAS_CONTACT`: the host already writes `HAS_CONTACT` when it
resolves a record onto its Person spine, and a vocabulary that reuses a host relationship
name for something else has made every traversal of it ambiguous.

`SUBSIDIARY_OF` and `HAS_SUBSIDIARY` are the company hierarchy as the CRM records it: the
parent company a company belongs to, and the companies filed under it. Every CRM keeps one —
Odoo's parent company on a company record, Salesforce's Parent Account, HubSpot's parent
company association — and a realm maps its own onto these two names, so "this company and its
subsidiaries" is `(k:CrmAccount)-[:HAS_SUBSIDIARY*1..4]->(s:CrmAccount)` whichever CRM answers.
Two names rather than one edge walked both ways, as FIBO does for control: a walk UP is one
parent per hop and never asks, at each company it passes, for all of that company's siblings.
They join CRM records, not `CustomerAccount`s: a group's companies are separate accounts with
domains of their own, and each subsidiary reaches its own account the usual way, by its website.
The edge joins companies only. A person's link to the company they work at is
`HAS_ACCOUNT_CONTACT`, even where the CRM stores both in one field.

## Reading a whole system

A question about the book — every open case, everything owed — has to start somewhere, and a
consumer that names no product cannot write a product's door. So a child realm also agrees:

- **A bare parent label reads the whole system.** `MATCH (c:SupportCase)` must reach every
  case. A child gets this from a book door whose default identity is its join key
  (`status: { metadata: { identity: "true", default: "all" } }` on the door, `keyField: status`
  on the join). Where a child has more than one such door onto the same type, the engine will
  not choose between them; mark the one that means what the parent means with
  `implicitDoor: true` (for `CrmAccount`, every company the CRM knows — prospects included).
- **A spine edge rides that sweep.** `TRACKED_AS`, `HAS_CASE` and `BILLED_AS` ask the product
  about one account at a time, which is refused above a few hundred accounts. Declared
  `ridesHub: true`, they are answered instead from records the query has already swept, matched
  on the spine's key. That is sound only if the sweep is COMPLETE for the edge: the bare-label
  door above must reach every record the edge would. A narrower door (customers who have bought,
  open cases only) ridden by a wider edge loses the rest without a word.

So the shape a consumer writes for an account-level question over the book is: sweep by the
bare label, then join through the spine.

```cypher
MATCH (k:CrmAccount)                                     // the whole CRM, one door
MATCH (a:CustomerAccount)-[:TRACKED_AS]->(k)             // rides the sweep, no call per account
```

## Depending on it

A realm cannot yet declare that it needs another realm (embabel/me#495, #1168). Until it
can: install this realm first. A child whose parent is not installed is a type with a parent
the world has never heard of.
