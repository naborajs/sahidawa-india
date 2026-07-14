# ADR 0002 - Use Supabase for Primary Database and Auth

* Status: accepted
* Deciders: SahiDawa Core Team
* Date: 2026-07-14

Technical Story: [System Design Documentation](../architecture/system-design.md)

## Context and Problem Statement

SahiDawa requires a robust data persistence layer to store master drug records (scanned barcode IDs, CDSCO approvals, manufacturer information), pharmacy coordinates, counterfeit medicine reports, and citizen alert histories.
We also need:
1. Geospatial queries (via PostGIS) to locate pharmacies near user coordinates.
2. Vector embeddings (via pgvector) for fuzzy drug identification and semantic RAG lookups.
3. User authentication for admins, pharmacists, and reporting citizens.
Hosting, managing, and securing a PostgreSQL database cluster with these extensions along with implementing a secure auth backend introduces significant operational complexity and cost.

## Decision Drivers

* Low operational overhead (managed database infrastructure).
* Support for PostGIS, pgvector, and pg_trgm extensions on PostgreSQL.
* Integrated authentication mechanism (Supabase Auth) with easy-to-use React and Express SDKs.
* Cost-effective (generous free tier suitable for early-stage development).

## Considered Options

* **Option 1: Self-Hosted PostgreSQL with custom Auth Server** (Deploying Postgres on Docker/cloud instances, building custom JWT authentication service).
* **Option 2: Supabase** (Managed Postgres with built-in Auth, instant REST APIs, and native extension support).
* **Option 3: MongoDB / NoSQL Database** (Document-based database, utilizing external Auth provider).

## Decision Outcome

Chosen option: **Option 2: Supabase**, because it eliminates the need to build and maintain user sign-up/login backend pipelines, natively supports required extensions (PostGIS for mapping, pgvector for vector search, pg_trgm for fuzzy matching), and provides structured Postgres capability out-of-the-box.

### Consequences

* **Good:**
  * Zero database server management overhead.
  * Native Postgres capability allows writing complex SQL queries, transactions, and triggers directly.
  * Ready-made auth helpers (`@supabase/ssr` and `@supabase/supabase-js`) speed up client-side and API-side authentication.
  * Simple auto-generated REST/GraphQL endpoints can be leveraged if necessary.
* **Bad:**
  * Moderate vendor lock-in to Supabase-specific client configurations, though the database itself is standard PostgreSQL and can be migrated if necessary.
  * Connection limits and billing scale limits on the free tier when traffic grows.

## Pros and Cons of the Options

### Option 1: Self-Hosted PostgreSQL with custom Auth Server

* **Good:** Complete control over DB tuning, hosting environment, and authentication logic.
* **Bad:** High dev overhead to build, patch, and secure auth routes; manual database backup and extension configurations.

### Option 3: MongoDB / NoSQL Database

* **Good:** Highly flexible schema for medicine reports.
* **Bad:** Poor native support for complex geospatial queries compared to PostGIS; lacks native vector search options matching pgvector's capabilities; requires a separate authentication backend.
