# Genesis — Plan technique (phases 0 et 1 = sprint 1)

> Auteur : agent Architecte. Source : `spec-genesis.txt` (v1.0). Date : 2026-10-07.
> Portée détaillée : phase 0 (Foundation, §23) + phase 1 (CEO + agents) = sprint 1 (§25.1).
> Sprints 2 et 3 survolés en fin de document.

## 0. Arbitrages à connaître avant de lire

1. **Le scénario §25.4 contient des lignes de sprint 2.** « Skill Manager -> build candidate skill » et
   « Skill tests -> PASS » dépendent du Skill Registry/Builder/sandbox, prévus au sprint 2 (§25.2).
   Critère de fin du sprint 1 retenu : le scénario §25.4 **jusqu'à « CEO -> create venture »**, puis
   « Capital » et « Timeline -> persisted ». Les deux lignes skill s'affichent `skipped (sprint 2)`.
   Le scénario complet devient le critère de fin du sprint 2 (voir la question 4).
2. **V1 = capital virtuel, coûts API réels** (§17). Les 100 EUR sont simulés, mais en mode réel chaque
   appel LLM et chaque recherche coûte de l'argent réel. Il y a donc **deux plafonds indépendants** :
   le budget virtuel du ledger et un plafond de dépense réelle par exécution (`GENESIS_REAL_SPEND_CAP_EUR`).
3. **Le paquet Python s'appelle `genesis`** (pour que `python -m genesis` fonctionne) et vit dans
   `src/genesis/`. Ses sous-paquets reprennent les noms de §22 (`api`, `agents`, `runtime`, `finance`…).

## 1. Stack

| Choix | Décision | Raison (une ligne) |
|---|---|---|
| Langage | **Python 3.12** | Imposé par la spec. 3.12 est stable et toutes les dépendances le supportent. |
| Paquets | **uv** (`pyproject.toml` + `uv.lock`) | Rapide, gère aussi la version de Python, et fonctionne pareil sous Windows et en CI. |
| API | **FastAPI** + uvicorn | Imposé (§22-23). Au sprint 1, l'API se limite à `/health` et à des endpoints de lecture. |
| BDD | **PostgreSQL 16 via Docker Compose** | Imposé. En local sur le Docker de Hugo. Supabase reste possible plus tard : c'est du Postgres standard. |
| Accès BDD | **psycopg 3 + psycopg_pool, sans ORM** | Le contrat repose sur du SQL, des triggers et des verrous : un ORM y ajouterait une couche sans gain. |
| Migrations | **SQL brut versionné** (`migrations/NNNN_nom.sql`) + petit runner (table `schema_migrations`) | Triggers et vues s'écrivent plus lisiblement en SQL. Pas d'autogénération à surveiller. |
| Modèles de données | **pydantic v2** (+ pydantic-settings pour `.env`) | Validation des décisions JSON du LLM et schéma JSON généré pour les structured outputs. |
| LLM | **SDK `anthropic`** derrière `LLMProvider` ; **`FakeProvider`** déterministe | Multi-fournisseurs (§6) et CI à 0 coût. |
| Recherche web | **Tavily** (API conçue pour les agents, offre gratuite) derrière `web_search` ; **`FakeSearchProvider`** à fixtures | Une seule clé, résultats déjà condensés. Alternatives : Brave Search, ou l'outil serveur `web_search` d'Anthropic (sans clé supplémentaire, mais lié au fournisseur LLM). |
| Console | **rich** | Timeline et capital lisibles dans le terminal Windows. |
| Qualité | **pytest**, **ruff** (lint + format) | Standard, rapide. Pas de mypy strict au sprint 1 ; pyright en option. |
| CI | **GitHub Actions** avec service `postgres:16` | Lance migrations + tests + scénario fake. Aucune clé API en CI. |

Modèles Anthropic (tarifs en USD par million de tokens, d'après la référence SDK au 2026-10-06 ;
**à revérifier** avant le mode réel). Ils sont configurés dans `config/models.yaml`, jamais écrits en dur dans le code :

```yaml
# config/models.yaml — propriété de la tâche P0-4
fx: { USD_EUR: 0.92 }            # conversion vers la devise du ledger (EUR), à mettre à jour
tiers:                            # tier 0 = code, sans LLM
  economy:  { provider: anthropic, model: claude-haiku-5-5 }
  standard: { provider: anthropic, model: claude-sonnet-5-5 }
  manager:  { provider: anthropic, model: claude-opus-5-5, effort: low }
  expert:   { provider: anthropic, model: claude-fable-5-1, enabled: false }
prices_usd_per_mtok:              # input / output (les tokens de « thinking » sont facturés en output)
  anthropic/claude-haiku-5-5:  { input: 0.10, output: 0.50 }   # ≤100K tokens de prompt, sinon 0.50/2.50
  anthropic/claude-sonnet-5-5: { input: 2.00, output: 10.00 }
  anthropic/claude-opus-5-5:   { input: 4.00, output: 20.00 }
  anthropic/claude-fable-5-1:  { input: 10.00, output: 50.00 }
  fake/fake-economy:           { input: 0.10, output: 0.50 }   # le mode fake débite comme le réel
  fake/fake-manager:           { input: 4.00, output: 20.00 }
search:
  tavily: { cost_usd_per_call: 0.008 }   # à vérifier selon l'offre ; 0 si l'offre gratuite suffit
  fake:   { cost_usd_per_call: 0.008 }
limits: { max_output_tokens: { economy: 2000, standard: 4000, manager: 3000 } }
```

Notes LLM : sur `claude-opus-5-5`, le « thinking » ne peut pas être désactivé ; on règle donc `effort`
(`low` pour le CEO au sprint 1) et le `max_tokens` borne le coût. Sortie JSON du CEO :
structured outputs (`output_config.format` avec le JSON Schema pydantic), puis re-validation pydantic.
Le choix forcé d'outil (`tool_choice` any/tool) n'est pas utilisé : il est refusé sur ces modèles.
Une réponse avec `stop_reason == "refusal"` est traitée comme un run échoué.

## 2. Structure du dépôt (sprint 1)

```
genesis/
├── pyproject.toml  uv.lock  .python-version  .env.example  .gitignore  .gitattributes
├── docker-compose.yml               # postgres:16 (+ volume), port 5432
├── config/models.yaml               # tiers, prix, fx, limites
├── migrations/0001_core.sql         # schéma §3 ci-dessous
├── src/genesis/
│   ├── __main__.py   cli.py         # python -m genesis …
│   ├── config.py                    # Settings (pydantic-settings) + chargement YAML
│   ├── database/  (db.py, migrate.py)
│   ├── finance/   (ledger.py, models.py)
│   ├── llm/       (provider.py, anthropic_provider.py, fake_provider.py, router.py, pricing.py)
│   ├── runtime/   (events.py, scheduler.py, cost_guard.py, runtime.py)
│   ├── agents/    (service.py, ceo.py, research.py, schemas.py, prompts/ceo.md, prompts/research.md)
│   ├── capabilities/ (registry.py, providers/web_search_tavily.py, providers/web_search_fake.py, policies/permissions.py)
│   ├── ventures/  (service.py)      # création minimale uniquement
│   ├── console/   (timeline.py)     # affichage rich
│   └── api/       (app.py, routes_read.py)
├── tests/ (conftest.py, unit/, db/, e2e/, fixtures/fake_llm/*.json, fixtures/search/*.json)
├── .github/workflows/ci.yml
└── docs/ (README dev Windows, decisions.md)
```
**Plus tard** : `skills/` + `src/genesis/skills/` (sprint 2), `memory/` (phase 3+),
`frontend/` (sprint 3), `infra/` (déploiement), providers GitHub/email/image/hosting.

## 3. Contrat figé (à transmettre tel quel à tous les agents de dev)

### 3.1 Schéma SQL — `migrations/0001_core.sql` (propriété : P0-2)

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE TABLE schema_migrations (version TEXT PRIMARY KEY, applied_at TIMESTAMPTZ NOT NULL DEFAULT now());

CREATE TABLE wallet_accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL UNIQUE,
  kind TEXT NOT NULL CHECK (kind IN ('simulated','real')),
  provider TEXT, currency TEXT NOT NULL DEFAULT 'EUR' CHECK (currency = 'EUR'),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now());

CREATE TABLE agents (                                  -- §18.2, typé
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  parent_id UUID REFERENCES agents(id),
  venture_id UUID,                                     -- FK ajoutée après ventures
  name TEXT NOT NULL, role TEXT NOT NULL, mission TEXT NOT NULL,
  kind TEXT NOT NULL CHECK (kind IN ('ceo','manager','worker','consultant','auditor')),
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active','suspended','terminated')),
  model_tier TEXT NOT NULL DEFAULT 'economy' CHECK (model_tier IN ('economy','standard','manager','expert')),
  capabilities TEXT[] NOT NULL DEFAULT '{}',           -- ex. {web_search}, {spawn_agent,create_venture}
  permissions JSONB NOT NULL DEFAULT '{}'::jsonb,      -- plafonds : max_per_call_eur, max_children…
  depth SMALLINT NOT NULL DEFAULT 0 CHECK (depth BETWEEN 0 AND 3),
  expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(), terminated_at TIMESTAMPTZ);
CREATE UNIQUE INDEX one_ceo ON agents ((kind)) WHERE kind = 'ceo' AND status <> 'terminated';

CREATE TABLE ventures (                                -- minimal sprint 1
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL, hypothesis TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'proposed' CHECK (status IN ('proposed','validating','building','scaling','killed')),
  owner_agent_id UUID NOT NULL REFERENCES agents(id),
  criteria JSONB NOT NULL DEFAULT '{}'::jsonb,         -- kill_if / scale_if (§11)
  created_at TIMESTAMPTZ NOT NULL DEFAULT now());
ALTER TABLE agents ADD FOREIGN KEY (venture_id) REFERENCES ventures(id);

CREATE TABLE agent_budgets (                           -- une « poche » par agent ; AUCUN solde stocké
  agent_id UUID PRIMARY KEY REFERENCES agents(id),
  wallet_account_id UUID NOT NULL REFERENCES wallet_accounts(id),
  max_per_call_eur NUMERIC(18,6),                      -- plafond par opération (blast radius §20.3)
  created_at TIMESTAMPTZ NOT NULL DEFAULT now());

CREATE TABLE ledger_entries (                          -- §18.4 étendu ; IMMUABLE
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  seq BIGSERIAL UNIQUE,                                -- ordre total, rejouable
  txn_id UUID NOT NULL,                                -- regroupe les lignes d'une même opération
  account_id UUID NOT NULL REFERENCES wallet_accounts(id),
  agent_id UUID NOT NULL REFERENCES agent_budgets(agent_id),
  venture_id UUID REFERENCES ventures(id),
  entry_type TEXT NOT NULL CHECK (entry_type IN
    ('funding','allocation','release','hold','hold_release','spend','revenue')),
  amount NUMERIC(18,6) NOT NULL CHECK (amount <> 0),
  currency TEXT NOT NULL DEFAULT 'EUR' CHECK (currency = 'EUR'),
  category TEXT NOT NULL,                              -- llm | search | allocation | funding | …
  provider TEXT, description TEXT,
  external_reference TEXT,                             -- id agent_run / action
  idempotency_key TEXT UNIQUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now());
CREATE INDEX ON ledger_entries (agent_id);  CREATE INDEX ON ledger_entries (txn_id);
```

**Sémantique comptable** (le cash reste dans un seul wallet ; les poches sont virtuelles, §4) :
- `funding` (+100 sur la poche du CEO) ; `allocation`/`release` = **paire** −x parent / +x enfant, même `txn_id`, somme nulle.
- `hold` (−max estimé) **avant** l'exécution ; ensuite `hold_release` (+même montant) + `spend` (−coût réel) dans la même transaction.
- Cash = Σ(funding, spend, revenue). Disponible(agent) = Σ de toutes ses lignes.
  ⇒ Σ disponibles = cash − holds en cours ≤ cash (**somme des allocations ≤ fonds**, par construction).

```sql
-- Immutabilité : ni UPDATE, ni DELETE, ni TRUNCATE (et le rôle applicatif n'a que INSERT/SELECT)
CREATE FUNCTION forbid_mutation() RETURNS trigger LANGUAGE plpgsql AS $$
BEGIN RAISE EXCEPTION 'append-only table %', TG_TABLE_NAME USING ERRCODE = 'GN001'; END $$;
CREATE TRIGGER ledger_no_update BEFORE UPDATE OR DELETE ON ledger_entries FOR EACH ROW EXECUTE FUNCTION forbid_mutation();
CREATE TRIGGER ledger_no_truncate BEFORE TRUNCATE ON ledger_entries FOR EACH STATEMENT EXECUTE FUNCTION forbid_mutation();

-- Sérialisation par poche : verrou transactionnel sur l'agent avant toute insertion
CREATE FUNCTION ledger_lock() RETURNS trigger LANGUAGE plpgsql AS $$
BEGIN PERFORM pg_advisory_xact_lock(hashtextextended(NEW.agent_id::text, 0)); RETURN NEW; END $$;
CREATE TRIGGER ledger_lock BEFORE INSERT ON ledger_entries FOR EACH ROW EXECUTE FUNCTION ledger_lock();

-- Conservation : poche jamais négative ; paires allocation/release équilibrées (vérifié au COMMIT)
CREATE FUNCTION ledger_check() RETURNS trigger LANGUAGE plpgsql AS $$
BEGIN
  IF (SELECT sum(amount) FROM ledger_entries WHERE agent_id = NEW.agent_id) < 0 THEN
    RAISE EXCEPTION 'insufficient_budget agent=%', NEW.agent_id USING ERRCODE = 'GN002'; END IF;
  IF NEW.entry_type IN ('allocation','release') AND
     (SELECT sum(amount) FROM ledger_entries WHERE txn_id = NEW.txn_id) <> 0 THEN
    RAISE EXCEPTION 'unbalanced txn %', NEW.txn_id USING ERRCODE = 'GN003'; END IF;
  RETURN NULL; END $$;
CREATE CONSTRAINT TRIGGER ledger_check AFTER INSERT ON ledger_entries
  DEFERRABLE INITIALLY DEFERRED FOR EACH ROW EXECUTE FUNCTION ledger_check();

CREATE VIEW v_budget_balances AS
  SELECT agent_id,
    sum(amount) AS available,
    -sum(amount) FILTER (WHERE entry_type IN ('hold','hold_release')) AS held,
    -sum(amount) FILTER (WHERE entry_type = 'spend') AS spent
  FROM ledger_entries GROUP BY agent_id;
CREATE VIEW v_company AS
  SELECT sum(amount) FILTER (WHERE entry_type IN ('funding','spend','revenue')) AS cash,
         -sum(amount) FILTER (WHERE entry_type IN ('hold','hold_release')) AS held
  FROM ledger_entries;

CREATE TABLE tasks (                                   -- file de travail du scheduler (mutable)
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  parent_task_id UUID REFERENCES tasks(id),
  agent_id UUID NOT NULL REFERENCES agents(id),        -- propriétaire / exécutant
  created_by_agent_id UUID REFERENCES agents(id),
  venture_id UUID REFERENCES ventures(id),
  kind TEXT NOT NULL,                                  -- ceo_review | research | …
  goal TEXT NOT NULL, input JSONB NOT NULL DEFAULT '{}'::jsonb, result JSONB,
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending','running','done','failed','cancelled')),
  attempts SMALLINT NOT NULL DEFAULT 0, max_attempts SMALLINT NOT NULL DEFAULT 2,
  not_before TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(), started_at TIMESTAMPTZ, finished_at TIMESTAMPTZ);
CREATE INDEX tasks_queue ON tasks (status, not_before);

CREATE TABLE agent_runs (                              -- 1 ligne par appel LLM ; append-only
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agents(id), task_id UUID REFERENCES tasks(id),
  provider TEXT NOT NULL, model TEXT NOT NULL, tier TEXT NOT NULL, task_type TEXT NOT NULL,
  input_tokens INT NOT NULL DEFAULT 0, output_tokens INT NOT NULL DEFAULT 0,
  estimated_cost_eur NUMERIC(18,6) NOT NULL, actual_cost_eur NUMERIC(18,6) NOT NULL DEFAULT 0,
  status TEXT NOT NULL CHECK (status IN ('ok','refused_budget','refused_guard','invalid_output','error','refusal')),
  latency_ms INT, request_hash TEXT NOT NULL,
  output JSONB,                                        -- décision validée, jamais de chaîne de pensée
  ledger_txn_id UUID, error TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now());

CREATE TABLE actions (                                 -- contrat §19.2
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agents(id), task_id UUID REFERENCES tasks(id),
  run_id UUID REFERENCES agent_runs(id), venture_id UUID REFERENCES ventures(id),
  capability TEXT NOT NULL, parameters JSONB NOT NULL,
  risk_class TEXT NOT NULL CHECK (risk_class IN ('internal_read','internal_write','external_reversible',
                                                 'external_write','financial','destructive')),
  estimated_cost NUMERIC(18,6) NOT NULL DEFAULT 0, actual_cost NUMERIC(18,6),
  idempotency_key TEXT NOT NULL UNIQUE, reason TEXT NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('rejected','running','succeeded','failed')),
  result JSONB, error TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(), finished_at TIMESTAMPTZ);

CREATE TABLE events (                                  -- audit / timeline ; append-only
  seq BIGSERIAL PRIMARY KEY,
  id UUID NOT NULL DEFAULT gen_random_uuid() UNIQUE,
  type TEXT NOT NULL,
  agent_id UUID, venture_id UUID, task_id UUID, action_id UUID, run_id UUID,
  payload JSONB NOT NULL DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now());
CREATE TRIGGER events_no_update BEFORE UPDATE OR DELETE ON events FOR EACH ROW EXECUTE FUNCTION forbid_mutation();
CREATE TRIGGER runs_no_update   BEFORE UPDATE OR DELETE ON agent_runs FOR EACH ROW EXECUTE FUNCTION forbid_mutation();
```
Rôles : `genesis_owner` (migrations) et `genesis_app` (runtime : `INSERT, SELECT` seulement sur
`ledger_entries`, `events` et `agent_runs`). Remise à zéro en dev : `DROP SCHEMA` + migrate (`python -m genesis db reset`),
jamais de DELETE.

Types d'événements du sprint 1 : `company_started, agent_spawned, agent_terminated, budget_allocated,
budget_released, llm_call, spend_refused, task_created, task_completed, task_failed, action_executed,
action_rejected, venture_created, decision_made, cost_guard_tripped, run_finished`.

### 3.2 Interfaces Python (signatures figées ; l'implémentation est libre)

```python
# finance/ledger.py — tous les montants sont des Decimal (quantize 6 décimales), jamais des float
class InsufficientBudget(Exception): ...
@dataclass(frozen=True) class Hold: txn_id: UUID; agent_id: UUID; amount: Decimal
class FinanceService:
    def init_company(self, capital: Decimal, ceo_id: UUID) -> UUID: """Crée le wallet simulé + le funding initial sur le CEO (idempotent)."""
    def available(self, agent_id: UUID) -> Decimal: """Budget disponible de la poche (alloué − dépensé − holds)."""
    def allocate(self, parent_id: UUID, child_id: UUID, amount: Decimal, reason: str) -> UUID: """Paire −parent/+enfant ; lève InsufficientBudget."""
    def release(self, child_id: UUID, parent_id: UUID, amount: Decimal | None, reason: str) -> UUID: """Rend tout ou partie du reliquat de l'enfant au parent."""
    def hold(self, agent_id: UUID, amount: Decimal, reason: str, idempotency_key: str) -> Hold: """Réserve le coût max AVANT exécution ; refuse si > disponible ou > max_per_call."""
    def settle(self, hold: Hold, actual: Decimal, category: str, provider: str, ref: str) -> UUID: """Libère le hold et débite le coût réel (exige actual ≤ hold.amount, sinon incident)."""
    def cancel(self, hold: Hold) -> None: """Libère un hold sans dépense (échec avant appel)."""
    def company_snapshot(self) -> CompanySnapshot: """Cash, held, poches par agent, dépense par catégorie."""
    def check_invariants(self) -> list[str]: """Liste des violations (vide = OK) : poches ≥ 0, Σ poches = cash − held."""

# llm/provider.py
class LLMRequest(BaseModel): system: str; messages: list[dict]; max_tokens: int; json_schema: dict | None = None; effort: str | None = None
class LLMResponse(BaseModel): text: str; parsed: dict | None; input_tokens: int; output_tokens: int; provider: str; model: str; stop_reason: str; latency_ms: int
class LLMProvider(Protocol):
    name: str
    def estimate_input_tokens(self, req: LLMRequest, model: str) -> int: """Majorant du nombre de tokens d'entrée (heuristique prudente ou count_tokens)."""
    def complete(self, req: LLMRequest, model: str) -> LLMResponse: """Un appel, sans retry métier ; lève LLMError."""

# llm/router.py
class RouteRequest(BaseModel): task_type: str; tier: Literal['economy','standard','manager','expert']; max_cost_eur: Decimal | None = None
class ModelChoice(BaseModel): provider: str; model: str; tier: str; price_in: Decimal; price_out: Decimal  # EUR/Mtok
class ModelRouter:
    def route(self, req: RouteRequest) -> ModelChoice: """Tier → modèle via config/models.yaml ; descend d'un tier si max_cost_eur est dépassé."""
    def estimate_max_cost(self, choice: ModelChoice, req: LLMRequest) -> Decimal: """input_estimé × prix_in + max_tokens × prix_out."""
    def actual_cost(self, choice: ModelChoice, resp: LLMResponse) -> Decimal: """Coût réel depuis usage."""
    def call(self, agent_id: UUID, task_id: UUID | None, route: RouteRequest, req: LLMRequest) -> LLMResponse:
        """guard → hold → complete → settle → agent_runs → event llm_call ; refus = spend_refused, sans appel réseau."""

# runtime/events.py
class EventLog:
    def emit(self, type: str, *, payload: dict, agent_id=None, venture_id=None, task_id=None, action_id=None, run_id=None) -> int: """Ajoute un événement, renvoie seq."""
    def since(self, seq: int = 0, limit: int = 500, types: list[str] | None = None) -> list[Event]: """Lecture ordonnée pour timeline/API."""

# runtime/cost_guard.py
class CostGuard:
    def check_before_call(self, agent_id: UUID, est_cost: Decimal) -> None: """Lève GuardTripped : plafond réel du run, max appels/agent, max steps."""
    def record(self, agent_id: UUID, cost: Decimal, progressed: bool) -> None: """Suspend un agent après N décisions sans progrès."""

# agents/schemas.py — Annexe B, validé par pydantic, schéma envoyé en structured output
class SpawnAgent(BaseModel): type: Literal['spawn_agent']; role: str; mission: str; budget: Decimal; model_tier: Tier = 'economy'; capabilities: list[str] = []; kind: Literal['worker','manager','consultant'] = 'worker'
class TerminateAgent(BaseModel): type: Literal['terminate_agent']; agent_id: UUID; reason: str
class CreateVenture(BaseModel): type: Literal['create_venture']; name: str; hypothesis: str; budget: Decimal; kill_if: list[str]; scale_if: list[str]
class AssignTask(BaseModel): type: Literal['assign_task']; agent_id: UUID; goal: str; input: dict = {}
class Wait(BaseModel): type: Literal['wait']; reason: str
class CEODecision(BaseModel):
    summary: str; objective: str
    actions: list[Annotated[SpawnAgent | TerminateAgent | CreateVenture | AssignTask | Wait, Field(discriminator='type')]]
    expected_value: Decimal; max_loss: Decimal; recheck_when: list[str]
class Opportunity(BaseModel): title: str; problem: str; target: str; monetization: str; evidence_urls: list[str]; score: float
class ResearchReport(BaseModel): opportunities: list[Opportunity] = Field(min_length=3, max_length=3); cost_note: str

# agents/service.py
class AgentService:
    def create_ceo(self, capital: Decimal) -> Agent: """CEO + poche + funding (une transaction)."""
    def spawn_agent(self, parent_id: UUID, spec: SpawnAgent, venture_id: UUID | None = None) -> Agent:
        """Une transaction : contrôles (permission spawn, capacités ⊆ parent, profondeur ≤ 3, max agents actifs),
        insert agent + poche + allocate(parent→enfant) + event. Refus sans effet de bord."""
    def terminate_agent(self, agent_id: UUID, reason: str) -> Decimal: """Termine les descendants, rend le reliquat au parent, renvoie le montant rendu."""

# agents/ceo.py, agents/research.py, runtime/runtime.py
class CEOAgent:      def run(self, state: CompanyState) -> CEODecision: """Prompt Annexe A + état minimal (capital, poches, agents, ventures, derniers résultats)."""
class ResearchAgent: def run(self, task: Task) -> ResearchReport: """web_search (2–4 requêtes) puis synthèse economy-tier en 3 opportunités."""
class AgentRuntime:
    def execute_decision(self, ceo_id: UUID, d: CEODecision) -> list[ActionResult]: """Valide puis exécute chaque action via services/registry ; un refus n'arrête pas les autres."""
    def step(self) -> StepResult | None: """Prend 1 tâche (FOR UPDATE SKIP LOCKED), build_minimal_state, run, exécute, journalise, planifie la suite."""
    def run_until_idle(self, max_steps: int) -> RunSummary: """Boucle §10.1 jusqu'à : file vide, max_steps, ou cost guard."""

# capabilities/registry.py
class CapabilityProvider(Protocol):
    name: str; capabilities: list[str]; risk_class: str; supports_dry_run: bool
    def estimate_cost(self, capability: str, params: dict) -> Decimal: ...
    def execute(self, capability: str, params: dict) -> dict: """Secrets lus côté serveur via credential_id, jamais dans params."""
class ActionRequest(BaseModel): agent_id: UUID; venture_id: UUID | None; capability: str; parameters: dict; estimated_cost: Decimal; risk_class: str; idempotency_key: str; reason: str
class CapabilityRegistry:
    def register(self, p: CapabilityProvider) -> None: ...
    def search(self, query: str, agent_id: UUID) -> list[CapabilityInfo]: """Capacités connectées ET permises à cet agent."""
    def execute(self, req: ActionRequest) -> ActionResult: """Permission → idempotence (renvoie le résultat existant) → hold → execute → settle → actions + event."""
```
CLI (`src/genesis/cli.py`) : `python -m genesis` (= `run`), `run [--mode fake|real] [--max-steps 20]
[--max-real-eur 1.00] [--capital 100]`, `db migrate|reset`, `state`, `timeline [--since N]`, `api`.
Le mode par défaut est **`fake`**. Le mode `real` exige la clé dans `.env` et affiche le plafond réel avant de démarrer.

## 4. Tâches

Règle de propriété : un fichier partagé appartient à UNE tâche. Si une autre tâche en a besoin, elle
l'écrit dans sa PR et c'est l'intégrateur qui applique la modification. **P0-1 pré-déclare toutes les
dépendances du sprint 1** (fastapi, uvicorn, psycopg[binary,pool], pydantic, pydantic-settings, pyyaml,
anthropic, httpx, rich, pytest, ruff, respx) et toutes les variables de `.env.example`.

### Phase 0 — Foundation

| # | Tâche | Livrable | Critère « terminé » | Dép. | Fichiers possédés | Profil |
|---|---|---|---|---|---|---|
| P0-1 | Squelette et CI | dépôt, uv, ruff, pytest, compose, CI, `config.py`, `__main__` minimal | `docker compose up -d` + `uv run pytest` vert ; CI verte ; README avec les étapes sous Windows | — | `pyproject.toml`, `uv.lock`, `docker-compose.yml`, `.env.example`, `.gitignore`, `.gitattributes`, `.github/`, `config.py`, `docs/README` | Dev Python outillage (uv/CI/Docker Windows) |
| P0-2 | Schéma et migrations | `0001_core.sql` (§3.1 à l'identique), runner, `db.py`, fixtures de test BDD | migrate idempotent ; tests : UPDATE/DELETE/TRUNCATE ledger refusés, poche négative refusée, paire déséquilibrée refusée | P0-1 | `migrations/`, `database/`, `tests/conftest.py`, `tests/db/` | Dev PostgreSQL (triggers, rôles) |
| P0-3 | Service Finance | `finance/` selon §3.2 | tests : 100 EUR initialisés ; allocate/release ; hold > disponible refusé AVANT ; settle ; 2 holds concurrents (threads) sans dépassement ; `check_invariants` vide | P0-2 | `finance/`, `tests/unit/finance*`, `tests/db/finance*` | Dev backend Python, comptabilité/concurrence |
| P0-4 | Passerelle de modèles | `llm/` : provider Anthropic + Fake, router, pricing, `config/models.yaml` | ≥ 2 tiers routés ; coût estimé ≥ coût réel sur les cas de test ; refus si budget insuffisant, sans appel réseau (vérifié par mock) ; ligne `agent_runs` + event ; 1 test réel marqué `@pytest.mark.real`, exclu de la CI | P0-2 (P0-3 via interface) | `llm/`, `config/models.yaml`, `tests/unit/llm*`, `tests/fixtures/fake_llm/` | Dev intégration LLM (SDK anthropic) |
| P0-5 | Event log et API | `runtime/events.py`, `api/` (`/health`, `/ready`, `GET /state/company`, `/ledger`, `/events`, `/agents`) | `/health` 200 ; `/ready` vérifie la BDD ; chaque écriture des services émet un event (test) ; API en lecture seule | P0-2 | `runtime/events.py`, `api/`, `tests/*api*`, `tests/*events*` | Dev FastAPI |

Parallélisme : P0-1 → P0-2 en séquence (courtes). Ensuite **P0-3, P0-4, P0-5 en parallèle, un worktree
chacun**, contre les interfaces §3.2. Puis l'**intégrateur** (branchement de `ModelRouter.call` sur le
`FinanceService` réel, CI verte) ouvre la PR « Phase 0 ».

### Phase 1 — CEO + agents (fin du sprint 1)

| # | Tâche | Livrable | Critère « terminé » | Dép. | Fichiers possédés | Profil |
|---|---|---|---|---|---|---|
| P1-1 | Agents et budgets parent→enfant | `agents/service.py` (create_ceo, spawn, terminate), `ventures/service.py` | tests : spawn réserve le budget ; spawn > disponible refusé sans effet ; capacités ⊄ parent refusées ; profondeur > 3 refusée ; terminate rend le reliquat et termine les descendants ; invariants OK | Phase 0 | `agents/service.py`, `ventures/`, tests associés | Dev backend domaine |
| P1-2 | Capabilities et web_search | `capabilities/` : registry, policies, Tavily + Fake, actions §19.2 | tests : capacité non accordée refusée ; même `idempotency_key` = un seul débit ; coût débité ; résultats balisés « données non fiables » ; timeout et fallback fake → `action failed` | Phase 0 | `capabilities/`, `tests/fixtures/search/`, tests associés | Dev intégration API externes + sécurité |
| P1-3 | Scheduler et cost guard | `runtime/scheduler.py`, `cost_guard.py`, `runtime.py` (step, run_until_idle, execute_decision) | tests : tâches consommées dans l'ordre ; retry ≤ max_attempts ; plafond réel atteint → arrêt + `cost_guard_tripped` ; agent sans progrès → `suspended` ; aucune boucle au-delà de max_steps | Phase 0 | `runtime/scheduler.py`, `runtime/cost_guard.py`, `runtime/runtime.py` | Dev runtime / systèmes |
| P1-4 | CEO et Research | `agents/ceo.py`, `research.py`, `schemas.py`, `prompts/` ; scripts fake du scénario | CEO.run → `CEODecision` valide (fake et réel) ; JSON invalide → 1 relance puis `invalid_output` ; Research → 3 opportunités ; prompts sans secret ni donnée de wallet réel | P1-1..3 (interfaces) | `agents/ceo.py`, `agents/research.py`, `agents/schemas.py`, `agents/prompts/`, `tests/fixtures/fake_llm/scenario_*` | Prompt engineer + dev Python (structured outputs) |
| P1-5 | CLI, timeline et scénario E2E | `cli.py`, `console/timeline.py`, `tests/e2e/test_scenario.py` | `python -m genesis --mode fake` affiche le scénario §25.4 (voir §0.1), test doré en CI ; `--mode real` exécuté une fois par Hugo/le Chef avec un coût total ≤ plafond, ledger = somme des `agent_runs` ; `timeline` reconstruit toutes les décisions depuis la BDD | P1-1..4 | `cli.py`, `__main__.py`, `console/`, `tests/e2e/` | Dev CLI/QA E2E |

Parallélisme : **P1-1, P1-2 et P1-3 en parallèle** (worktrees). **P1-4** peut démarrer en même temps
sur les interfaces figées avec des doublures, mais il est fusionné après eux. **P1-5** passe en dernier.
Ensuite l'intégrateur, puis la PR « Phase 1 ». Estimation : 10 agents de dev + 2 intégrateurs
(+ relecteurs si besoin). Le mode réel ne part qu'après accord de Hugo (dépense réelle).

Scénario fake attendu (déterministe) : CEO décide `spawn_agent research 0.50` → tâche research →
2 web_search fake + 1 synthèse economy → 3 opportunités → tâche `ceo_review` → CEO `create_venture` (budget
de validation alloué) + `terminate_agent research` (reliquat rendu) → `wait` → file vide → affichage du capital
(cash 100 − coûts) et de la timeline.

### Sprints 2 et 3 (survol)

- **Sprint 2** : Skill Registry (tags puis embeddings, pgvector), format `manifest.yaml`/`SKILL.md`,
  Skill Builder `candidate`, sandbox (sous-processus avec timeout, réseau coupé ; Docker plus tard),
  score minimal, scopes et promotion experimental→active ; capabilities GitHub/filesystem. Critère : §25.4 complet.
- **Sprint 3** : cycle de vie des ventures (kill/scale), email limité (draft puis send plafonné),
  génération d'image, preview deploy, dashboard web, simulations comparées (seed + mode fake/réel).

## 5. Risques et parades

| Risque | Parade |
|---|---|
| Coûts API réels qui dérivent | Clé dédiée dans un workspace Anthropic avec **limite de dépense mensuelle** ; `GENESIS_REAL_SPEND_CAP_EUR` vérifié par CostGuard avant chaque appel ; `max_tokens` par tier ; tier `expert` désactivé ; mode `fake` par défaut, CI sans clé. |
| Boucle LLM sans fin | Pas de boucle infinie (§10) : tâches seulement ; `max_steps`, max appels par agent et par tâche, détection « N décisions sans nouvelle action » → suspension, `max_attempts`. |
| Coût estimé < coût réel (thinking facturé en output) | Hold = majorant (entrée surestimée + `max_tokens` complet) ; `settle` vérifie actual ≤ hold, sinon incident et agent suspendu ; test sur les usages réels. |
| Course sur le budget | Verrou advisory par poche dans le trigger + contrôle au COMMIT ; test concurrent dans P0-3. |
| Secrets | `.env` hors git (`.gitignore` + `.env.example`) ; les clés ne sont lues que par les providers ; jamais dans un prompt, un event ou un log (filtre de redaction + test) ; secret scanning GitHub. |
| Injection de prompt via web_search | Résultats encadrés `<untrusted_web_content>` et présentés comme données ; Research n'a que `web_search` (pas de spawn ni de dépense) ; sortie validée par schéma ; le CEO reçoit le rapport structuré, pas le HTML brut ; URLs conservées pour l'audit. |
| JSON LLM invalide ou refus du modèle | Structured outputs + validation pydantic ; 1 relance, puis `invalid_output` journalisé ; `stop_reason=refusal` géré. |
| Tarifs ou modèles obsolètes | Tout dans `config/models.yaml` ; test qui échoue si un modèle routé n'a pas de prix. |
| Windows | `.gitattributes` (LF pour `.sql`/`.sh`), pathlib partout, pas de Makefile (commandes `uv run` / CLI Python), port 5432 configurable (conflit avec un Postgres local), Docker Desktop avec WSL2, encodage UTF-8 de la console (`PYTHONUTF8=1`). |
| Fournisseur indisponible | Timeouts et retries du SDK ; échec → `task failed` + replanification par le CEO ; pas de fallback silencieux vers un modèle plus cher. |

## 6. Questions pour Hugo (avec valeur par défaut)

1. **Clé LLM et plafond réel.** Défaut : clé Anthropic dédiée « genesis-dev » avec une limite mensuelle
   de 10 USD côté console, plafond runtime de **1 EUR par exécution** et **5 EUR au total** pour les essais du sprint 1.
2. **Recherche web.** Défaut : **Tavily** (offre gratuite, clé à créer). Alternative : l'outil
   `web_search` serveur d'Anthropic (sans clé de plus, mais dépendant du fournisseur LLM).
3. **Langue.** Défaut : code, identifiants et docstrings en **anglais** ; `docs/`, journal de
   mission et messages console en **français**.
4. **Lignes « skill » du scénario §25.4.** Défaut : reportées au **sprint 2**. Le sprint 1 se termine à
   « create venture » et affiche `skipped (sprint 2)` pour ces lignes. Alternative : un skill factice codé en dur dès le sprint 1.
