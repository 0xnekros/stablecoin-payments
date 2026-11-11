# ADR 0: Modular monolith first, or services from day one?

Status: proposed

Context:
I don't know the boundaries of my business logic and domain model yet. They'll show up while I build.
The ledger is the core subdomain: built in-house, and money correctness depends on it. Other modules ask it about the state of money.

Options:
- modular monolith
- services from day one

Decision:
Modular monolith. Services from day one would mean guessing service boundaries before I know them. In a modular monolith the module boundaries show up as I build, and a module with clear boundaries is easier to split out later.

Ledger module:
- in: ledger accounts (who has what, not merchant accounts), entries, balances, bookkeeping
- out: merchants, payouts, fee policy (what to charge)
- asks: other modules ask the ledger about balances and entries

Hexagonal structure:
- domain: model and money invariants, no Spring
- application: use cases, declares the ports
- inbound: a port that other modules and the merchant API call (POST /transfers, GET /balance, GET /entries in step 4)
- outbound: a persistence port, with a Postgres adapter

Known exceptions to the monolith, already in `Project.md`:
- signer: a separate service in stage 3, for key isolation
- webhook dispatcher: Go, stage 4, a different language
- chain-watcher: published as a separate module in stage 2

Consequences:
- easier: one build, one deploy, transactions across modules are possible
- harder: module boundaries are only enforced by discipline and ArchUnit, so they can erode
- split a module out when its requirements diverge and it needs to be maintained or deployed separately

Open:
- money invariants beyond "sum of entries = 0" → step 4
- debts / negative balances: does the ledger know about them? → step 4
- fee recording (entries to a fee account) vs fee policy → when fees are built (after 80%)
- does the ledger expose writes to other modules, and which ones? → deposits (stage 2), payouts (stage 3)
- HTTP controller in `ledger` or in `app` → step 3
- core / supporting / generic for the other subdomains → as each one is built
- published modules and license → stage 2 (chain-watcher)

Confirmation:
— (planned: ArchUnit rule "ledger domain doesn't import Spring", step 3)
