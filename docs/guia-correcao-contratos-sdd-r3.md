<!-- guia-correcao-contratos-sdd-r3.md | Vitalia Kit v0.0.1 | 25-11-2026 08:44(GMT-04:00) -->
**guia-correcao-contratos-sdd-r3.md | Vitalia Kit v0.0.1 | 25-11-2026 08:44(GMT-04:00)**

# Guia de Correções — `CONTRATOS_WORKFLOWS_SDD.md` (Rodada 3)

> **Propósito:** Documento autossuficiente para acompanhar [handoff-rodada-3-vitalia-kit-v0.0.1.md](./handoff-rodada-3-vitalia-kit-v0.0.1.md) na máquina de desenvolvimento do `vitalia-kit`, especificando os contratos que devem ser restaurados no código e os ajustes normativos exatos para `CONTRATOS_WORKFLOWS_SDD.md`.

---

## 1. Contratos que o Código da Rodada 3 Deve Restaurar

1. **Estrutura de `phase_guardrails` (todos os `gp_*.yaml` em `source/layers/profiles/`):**
   ```yaml
   phase_guardrails:
     phase: "<nome_da_fase>"
     role: "<papel_do_agente>"
     mandatory_checks:
       - "<checagem_1>"
   ```
   *(Em `gp_sdd-judge.yaml`, incluir também `mandatory_fmea_questions` dentro de `phase_guardrails` e o bloco `phase_profiles` com os 6 modos: `specify`, `plan`, `tasks`, `after_task`, `clarify`, `converge`).*
2. **Flags CLI de `source/src/vitalia_context_engine.py`:**
   - `--action {end,list-pending-domains,consolidate}` *(o provisionamento inicial da Camada 3 compete exclusivamente ao `install-project`)*
   - `--payload-file <caminho>` (ex.: `.vitalia/end_payload.json`)
   - `--render-session-dashboard` / `--render-data-dashboard` / `--render-grounding`
   - `--update-grounding-domains` / `--grounding-decisions-file`
3. **Transporte em `[transport]` (`source/layers/extensions/_base.toml` e `*.toml`):**
   - Remover `file_path = ".vitalia/pipeline.json"` e `file_path = ".vitalia/pipe.json"`.

---

## 2. Correções Exatas no `CONTRATOS_WORKFLOWS_SDD.md`

### Alteração 1 — Seção 1, Item 6 (Linha 38 — Topologia de 3 Camadas)
- **Substituir a descrição da Camada 3 por:**
  ```markdown
       - **Camada 3 — Memória e Governança Local (`<project_root>/.vitalia/`):** Metadados locais (`config.yml`, `feature.json`, `extensions.yml`), contexto de sessão (`memory/session/`) e dados operacionais (`memory/data_storage/`, incluindo `context/`, `handoffs/`, `research/`, `diagnostics/`). O transporte de estado entre etapas opera exclusivamente via envelopes em `data_storage/context/`, handoffs em `data_storage/handoffs/` e payloads transitórios (`end_payload.json`, `grounding_decisions_payload.json`), sem uso de `pipeline.json` ou `pipe.json`.
  ```

### Alteração 2 — Bloco `[transport]` dos 34 Contratos `.toml` (Seções 3, 4 e 5)
- **Localizar em todos os 34 blocos `.toml`:**
  ```toml
  [transport]
  preferred     = "filesystem"
  file_path     = ".vitalia/pipeline.json"
  session_shard = true
  ```
- **Substituir em todos os 34 blocos `.toml` por:**
  ```toml
  [transport]
  preferred     = "filesystem"
  session_shard = true
  ```

### Alteração 3 — Caminhos de Sessão (`.vitalia/memory/session/data/` $\rightarrow$ `.vitalia/memory/session/`)
- **Locais:** `session-start.toml` (`L203`), `session-end.toml` (`L1531`), `session-consolidate.toml` (`L1661–1662`), `skill-evaluation.toml` (`L2618`, `L2644`), `gp_session-consolidate.yaml` (`L3272`), Seção 7.5 (`L3866`) e Seção 11 (`L3978`, `L3985–3987`).
- **Regra:** Substituir toda referência a `.vitalia/memory/session/data/` por `.vitalia/memory/session/`, e remover menções a `SESSION_STATE.md` / `LEARNINGS.md` / `DECISIONS.md` caso não sejam gerados na versão atual ou consolidá-los conforme a arquitetura de dashboard único (`session/README.md`).

### Alteração 4 — `core/data_storage_io.py` e Validadores da Constituição (Seção 7.5)
- Confirmar que `core/data_storage_io.py` opera exclusivamente sobre `.vitalia/memory/data_storage/` (sem `.vitalia/session/data/` e sem `.vitalia-spec-kit`).
- Confirmar que `profiles/constitution.yaml` (`P07`, `P10`, `P12`) referencia validadores existentes (sem `hooks/llm_judge.py`).

