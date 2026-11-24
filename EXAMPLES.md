# Worked examples

Each reads one account, by its domain, so each product is asked about that domain alone:

```cypher
// One account's open support cases
MATCH (a:CustomerAccount {accountKey: 'acme.com'})-[:HAS_CASE]->(c:SupportCase)
WHERE c.status = 'open'
RETURN c.subject, c.priority, c.openedOn

// What one account owes, per currency
MATCH (a:CustomerAccount {accountKey: 'acme.com'})-[:BILLED_AS]->(:BillingCustomer)-[:HAS_INVOICE]->(i:BillingInvoice)
WHERE i.paymentStatus <> 'succeeded'
RETURN i.currency AS currency, sum(i.amount - i.amountPaid) AS owed, count(i) AS unpaidInvoices

// One account's open deals, per currency
MATCH (a:CustomerAccount {accountKey: 'acme.com'})-[:TRACKED_AS]->(:CrmAccount)-[:HAS_OPPORTUNITY]->(o:Opportunity)
WHERE o.status = 'open'
RETURN o.currency AS currency, sum(o.amount) AS openPipeline, collect(o.title) AS deals

// A company by name: exactly, because near names are other companies
MATCH (a:CustomerAccount) WHERE a.name = 'Acme AB'
RETURN a.accountKey, a.name, a.website
```

A question about the whole book sweeps by the bare label first; see the README's "Reading a whole
system".
