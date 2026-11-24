#### SQLModel Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** Unified Python ORM and data validation layer combining **SQLAlchemy** (SQL database engine/ORM) and **Pydantic** (validation/serialization) to eliminate duplicate schemas in FastAPI applications.
* **Runtime & Environment Requirements:** Supported Python 3.x version; install via `uv add sqlmodel` (auto-installs `sqlalchemy` and `pydantic`). Database-specific drivers installed separately (e.g., `psycopg[binary]`, `pymysql`, `aiosqlite`). Official AI coding skill available via `uvx library-skills`.

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Separate table models (`table=True`) from API schema models using inheritance: `EntityBase(SQLModel)` -> `Entity(EntityBase, table=True)`, `EntityCreate(EntityBase)`, `EntityPublic(EntityBase)`, `EntityUpdate(SQLModel)`.
  * Declare auto-increment primary keys as `id: int | None = Field(default=None, primary_key=True)` so instances can be constructed prior to database insertion.
  * Scope every database interaction inside a context-managed `with Session(engine) as session:` and inject one session per HTTP request in FastAPI via a `yield` dependency (`SessionDep = Annotated[Session, Depends(get_session)]`).
  * Query rows with `session.exec(select(Model).where(...))` rather than raw SQLAlchemy `session.execute(...)` to preserve SQLModel return types and IDE autocomplete.
  * Call `session.refresh(obj)` after `session.commit()` whenever database-generated fields (e.g., `id`, server defaults) must be read immediately.
  * Use `db_obj.sqlmodel_update(update_data)` with `patch_model.model_dump(exclude_unset=True)` for partial updates.
* **DON'T:**
  * Do not expose `table=True` models directly as FastAPI request/response payloads when they contain internal columns (e.g., `hashed_password`) or mutable primary keys.
  * Do not share a single `Session` instance across concurrent requests or threads; share the `Engine`, create one `Session` per unit of work/request.
  * Do not call `SQLModel.metadata.create_all(engine)` before all `table=True` model modules have been imported; unimported models will not have tables created.
  * Do not assume `table=True` models run full Pydantic validation automatically on instantiation in all contexts; validate untrusted external payloads through a non-table schema model (`*Create`/`*Update`) first, then convert via `Model.model_validate(payload)`.

##### 3. Core API Signatures & Cheatsheet
* `create_engine(url: str, *, echo: bool = False, connect_args: dict[str, Any] = {}) -> Engine`
* `SQLModel.metadata.create_all(bind: Engine) -> None`
* `Field(default: Any = Undefined, *, primary_key: bool = False, foreign_key: str | None = None, index: bool = False, unique: bool = False, nullable: bool = Undefined)`
* `Relationship(*, back_populates: str | None = None, link_model: Any = None, cascade_delete: bool = False)`
* `Session(bind: Engine)`:
  * `session.add(instance: object) -> None`
  * `session.commit() -> None`
  * `session.refresh(instance: object) -> None`
  * `session.delete(instance: object) -> None`
  * `session.get(entity: type[T], ident: Any) -> T | None`
  * `session.exec(statement: SelectOfScalar[T]) -> ScalarResult[T]` (`.all()`, `.first()`, `.one()`, `.one_or_none()`)

```python
from sqlmodel import Field, Relationship, Session, SQLModel, create_engine, select

class Team(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    heroes: list["Hero"] = Relationship(back_populates="team")

class HeroBase(SQLModel):
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)
    team_id: int | None = Field(default=None, foreign_key="team.id")

class Hero(HeroBase, table=True):
    id: int | None = Field(default=None, primary_key=True)
    team: Team | None = Relationship(back_populates="heroes")

class HeroCreate(HeroBase):
    pass

class HeroPublic(HeroBase):
    id: int

class HeroUpdate(SQLModel):
    name: str | None = None
    secret_name: str | None = None
    age: int | None = None
    team_id: int | None = None

engine = create_engine("sqlite:///database.db", connect_args={"check_same_thread": False})
SQLModel.metadata.create_all(engine)
```

```python
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException, Query
from sqlmodel import Session, select

def get_session():
    with Session(engine) as session:
        yield session

SessionDep = Annotated[Session, Depends(get_session)]
app = FastAPI()

@app.post("/heroes/", response_model=HeroPublic)
def create_hero(hero: HeroCreate, session: SessionDep) -> Hero:
    db_hero = Hero.model_validate(hero)
    session.add(db_hero)
    session.commit()
    session.refresh(db_hero)
    return db_hero

@app.patch("/heroes/{hero_id}", response_model=HeroPublic)
def update_hero(hero_id: int, hero: HeroUpdate, session: SessionDep) -> Hero:
    db_hero = session.get(Hero, hero_id)
    if not db_hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    hero_data = hero.model_dump(exclude_unset=True)
    db_hero.sqlmodel_update(hero_data)
    session.add(db_hero)
    session.commit()
    session.refresh(db_hero)
    return db_hero

@app.get("/heroes/", response_model=list[HeroPublic])
def read_heroes(
    session: SessionDep,
    offset: int = 0,
    limit: Annotated[int, Query(le=100)] = 100,
) -> list[Hero]:
    return list(session.exec(select(Hero).offset(offset).limit(limit)).all())
```

##### 4. Error Handling & Edge Cases
* **Official Exception Pattern:**
  * Catch `sqlalchemy.exc.IntegrityError` (unique/FK constraint violations) and call `session.rollback()` before returning HTTP `409 Conflict` or `400 Bad Request`.
  * Catch `sqlalchemy.exc.NoResultFound` / `MultipleResultsFound` when calling `.one()`; prefer `.first()` or `session.get(Model, pk)` followed by explicit `None` checks.
* **Edge Cases & Gotchas:**
  * **SQLite Threading:** SQLite requires `connect_args={"check_same_thread": False}` when used with FastAPI's threadpool.
  * **Expired Attributes:** After `session.commit()`, ORM attributes expire; accessing them after the `Session` closes raises `DetachedInstanceError` unless `session.refresh(obj)` was called inside the session context.
  * **Circular Type Annotations:** Use string forward references (e.g., `list["Hero"]`) and `TYPE_CHECKING` blocks for cross-file `Relationship` models.
  * **Table Validation Bypass:** Direct `Hero(name=123)` on `table=True` classes does not always enforce Pydantic coercion/validation; always parse incoming data through a `table=False` model first.

---
