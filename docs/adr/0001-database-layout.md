# ADR 1: One database for everything, or one per service?

Status: proposed

Context:
ADR 0 picks a modular monolith. A transfer has to commit in one transaction: all or nothing.

Options:
- one Postgres database, a schema per module
- one Postgres database, one shared schema
- a database per module or service

Decision:
One Postgres database, a schema per module.
- a database per service brings distributed transactions and operational cost before there are services
- one shared schema leaves no boundary between modules
- migrations run under a role separate from the runtime role (step 3)

Consequences:
- easier: one database to run, back up and migrate. Transactions across modules are possible
- harder: cross-schema joins, foreign keys or writes would make a later split harder
- move a module to its own database when it's split out of the monolith (ADR 0)

Open:
- which writes from different modules must commit together? → steps 4–6, stage 3 (outbox)
- rules between schemas: joins, foreign keys, writes to another module's tables → step 3
- a connection or role per module vs one pool (step 3 plans one pool) → step 3
- pool sizing → step 6
- entries append-only? If yes, the runtime role gets no UPDATE → step 4
- does a read model need its own database, or a table in the same one? → when it's needed
- does the hot account's write load force a different layout? → step 7
- does the signer's data share this database? → stage 3

Confirmation:
— (planned: Testcontainers test against real Postgres with the ledger schema, step 3)
