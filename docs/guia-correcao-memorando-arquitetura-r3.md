<!-- guia-correcao-memorando-arquitetura-r3.md | Vitalia Kit v0.0.1 | 24-11-2026 18:05(GMT-04:00) -->
**guia-correcao-memorando-arquitetura-r3.md | Vitalia Kit v0.0.1 | 24-11-2026 18:05(GMT-04:00)**

# Guia de Correções — `MEMORANDO DE ARQUITETURA.md` (Rodada 3)

> **Propósito:** Documento autossuficiente para acompanhar [handoff-rodada-3-vitalia-kit-v0.0.1.md](./handoff-rodada-3-vitalia-kit-v0.0.1.md) na máquina de desenvolvimento do `vitalia-kit`, contendo as diretrizes arquiteturais atualizadas e os trechos exatos de correção para `MEMORANDO DE ARQUITETURA.md`.

---

## 1. Diretrizes Arquiteturais Soberanas para a Rodada 3

1. **Descontinuação de `.vitalia/pipeline.json` e `.vitalia/pipe.json`:**
   - Nenhum arquivo do kit (`extensions/*.toml`, `src/`, `bin/`) deve exigir, gerar ou referenciar `.vitalia/pipeline.json` ou `.vitalia/pipe.json` como transporte ativo.
   - O transporte de estado em filesystem entre etapas do pipeline opera exclusivamente via:
     - Envelopes de contexto em `.vitalia/memory/data_storage/context/<machine_id>/`
     - Handoffs inter-agentes em `.vitalia/memory/data_storage/handoffs/` (`vitalia_handoff.py`)
     - Payloads transitórios na raiz de `.vitalia/` (`end_payload.json`, `grounding_decisions_payload.json`)
2. **Erradicação Total de `.vitalia/session/data`:**
   - Proibido em código, docstrings, comentários, schemas e testes.
   - Os arquivos `.jsonl` de sessão residem diretamente na raiz de `.vitalia/memory/session/` (`decisions.jsonl`, `learnings.jsonl`, `grounding-domains.jsonl`).
   - Os artefatos operacionais de runtime residem em `.vitalia/memory/data_storage/`.
3. **Restauração de `GuardianContextV2` (Sem Expansão de Escopo):**
   - `phase_guardrails` (`phase`, `role`, `mandatory_checks`) deve voltar a existir em todos os `gp_*.yaml` (`source/layers/profiles/`).
   - `phase_profiles` deve voltar a existir em `gp_sdd-judge.yaml` (expansão para outros agentes fica para spec futura).
4. **Preservação e Limpeza de `core/data_storage_io.py`:**
   - O módulo `source/src/core/data_storage_io.py` permanece como núcleo atômico de I/O da Camada 3, removendo apenas as referências legadas (`.vitalia-spec-kit` e `.vitalia/session/data/`).

---

## 2. Correções Exatas no `MEMORANDO DE ARQUITETURA.md`

### Alteração 1 — Seção 4.2, Item 3 (Linha 72 — Topologia da Camada 3)
- **Localizar:**
  ```markdown
  3. **Camada 3 — Memória e Governança Local (`<project_root>/.vitalia/`):** Metadados locais (`feature.json`, `pipeline.json`), Memória 3-Tier distribuída (`memory/session/`) e armazenamento operacional (`memory/data_storage/`, incluindo `context/`, `diagnostics/` e `research/`).
  ```
- **Substituir por:**
  ```markdown
  3. **Camada 3 — Memória e Governança Local (`<project_root>/.vitalia/`):** Metadados locais (`feature.json`, `config.yml`, `extensions.yml`), payloads transitórios (`end_payload.json`, `grounding_decisions_payload.json`), Memória 3-Tier distribuída (`memory/session/`) e armazenamento operacional (`memory/data_storage/`, incluindo `context/`, `handoffs/`, `diagnostics/` e `research/`). *(Os artefatos legados `pipeline.json` e `pipe.json` foram descontinuados; o transporte de estado em filesystem opera via `data_storage/context/`, `data_storage/handoffs/` e payloads transitórios).*
  ```

### Alteração 2 — Seção 5, Item 3 (Linha 88 — Nota de Transição `GuardianContextV2`)
- **Localizar:**
  ```markdown
     - *(Nota de transição — ADR-0014: os perfis compilados `profiles/gp_*.yaml` atuais adotam o esquema enxuto `mandatory_principles` / `guardrails` / `phase` / `role`, enquanto `GuardianContextV2` ainda lê `phase_guardrails` e `phase_profiles`, resultando em objetos vazios `{}` no envelope até o alinhamento previsto na spec de correções).*
  ```
- **Substituir por:**
  ```markdown
     - **Restauração Canônica (`phase_guardrails` & `phase_profiles`):** Os perfis compilados `profiles/gp_*.yaml` preservam o bloco estruturado `phase_guardrails` (`phase`, `role`, `mandatory_checks`), e `gp_sdd-judge.yaml` preserva `phase_profiles`, garantindo que `GuardianContextV2.build_context()` popule ambos os campos no envelope §11 sem objetos vazios `{}`.
  ```

### Alteração 3 — `ADR-0014`, Decisão 1 (Linha 261)
- **Localizar:**
  ```markdown
    1. **Formalização da Poda Restrita e Alinhamento de `GuardianContextV2`:** Registrar que `resolve_constitution()` aplica poda por `active_domain_gates` apenas aos overlays da Constituição (mantendo base, `grounding_domains` e catálogos integrais com `token_budget = 0`), e alinhar `GuardianContextV2` ao esquema real dos `gp_*.yaml` (ou restaurar `phase_guardrails`/`phase_profiles` via `gp_base.yaml` + `layers/profiles/`).
  ```
- **Substituir por:**
  ```markdown
    1. **Formalização da Poda Restrita, Restauração de `GuardianContextV2` e Descontinuação de `pipeline.json`:** Registrar que `resolve_constitution()` aplica poda por `active_domain_gates` apenas aos overlays da Constituição (mantendo base, `grounding_domains` e catálogos integrais com `token_budget = 0`); restaurar `phase_guardrails` nos `gp_*.yaml` e `phase_profiles` em `gp_sdd-judge.yaml` (`source/layers/profiles/`) para alinhamento integral com `GuardianContextV2` e `guardian_profile.schema.json`; manter `core/data_storage_io.py` livre de referências legadas (`.vitalia-spec-kit` e `.vitalia/session/data`); e descontinuar `.vitalia/pipeline.json` / `.vitalia/pipe.json`, consolidando o transporte em filesystem em `data_storage/context/`, `data_storage/handoffs/` e payloads transitórios em `.vitalia/`.
  ```

### Alteração 4 — Limpeza de Referências Legadas em ADRs anteriores (`L235`, `L247` e `L290`)
- **Regra de revisão:** Substituir eventuais citações a `.vitalia/memory/session/data/*.jsonl` por `.vitalia/memory/session/*.jsonl` na descrição da topologia vigente (preservando notas puramente históricas apenas se identificadas como legado).

