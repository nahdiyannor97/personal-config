---
name: sql-model
description: Relational database modeling, table declarations, foreign keys, relationships, session handling, and CRUD operations using SQLModel and SQLAlchemy. Use when defining database tables, migrations, indexes, or executing queries.
---

# SQLModel Engineering Skill

This skill governs the persistence and ORM layer using SQLModel. It bridges Python type annotations directly into relational database tables, managing entities, relationships, and queries.

---

## 1. Scope & Core Responsibilities

- **Entity Modeling:** Relational table declarations using `SQLModel` with `table=True`.
- **Relationship Mapping:** Parent-child mappings, foreign keys, cascade configurations, and many-to-many link tables using `Relationship`.
- **Query & CRUD Operations:** Building composable `select()` statements, filtering, sorting, pagination, and transactional updates.
- **Engine & Session Management:** Synchronous and asynchronous session factories, connection pooling, and Alembic migration compatibility.

---

## 2. Implementation Rules

1. **Table Flag Discipline:** Keep entities meant strictly for database storage (`table=True`) distinct from intermediate transfer structures.
2. **Atomic Transactions:** Wrap multi-step mutations in explicit transaction contexts (`session.commit()` with proper rollback handling).
3. **Index & Constraint Definitions:** Define explicit indexes on frequently filtered fields, unique constraints, and foreign key references at the model level.
