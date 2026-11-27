<!-- README.md | Vitalia Kit v0.0.1 | 26-11-2026 22:28(GMT-04:00) -->
**README.md | Vitalia Kit v0.0.1 | 26-11-2026 22:28(GMT-04:00)**

# Catálogo de Novos Schemas Canônicos — Specs 009 e 010 (`OpenAPI 3.0.3 Limpo — P11`)

> **Destino no Repositório `vitalia-kit`:** `source/src/schemas/` (`#25` a `#34`, elevando o total canônico de `24` para `34` schemas).  
> **Oráculo Mestre Consolidado:** [CONTRATOS_WORKFLOWS_SDD.md](../CONTRATOS_WORKFLOWS_SDD.md) (Seção 11 — contém a tabela mestra e o JSON literal completo de todos os **34** schemas `#01` a `#34`).  
> **Documento de Origem:** [brainstorming-auditoria-kit-ativo-spec-009-010-v0.0.1.md](../brainstorming-auditoria-kit-ativo-spec-009-010-v0.0.1.md) (Seção 6.3).  
> **Handoffs Associados:** [handoff-spec-009-determinismo-schemas-camada3-v0.0.1.md](../handoff-spec-009-determinismo-schemas-camada3-v0.0.1.md) · [handoff-spec-010-contratos-toml-catalogos-v0.0.1.md](../handoff-spec-010-contratos-toml-catalogos-v0.0.1.md).

---

## 1. Regras Invioláveis de Conformidade OpenAPI 3.0.3 Limpo (`P11`)

Todos os 10 arquivos `.schema.json` neste diretório obedecem estritamente às invariantes verificadas por `source/tests/test_schemas_openapi30_p11.py`:
- Raiz `type: "object"` com chaves de topo restritas a `["description", "type", "required", "properties"]`.
- **Zero** `$schema` em qualquer nível.
- **Zero** `title` na raiz.
- **Zero** `additionalProperties: false` em qualquer nível.
- **Zero** `type` em formato array (`["number", "string"]`) e **zero** `type: "null"`.

---

## 2. Inventário dos 10 Novos Schemas (`#25` a `#34`)

| # | Arquivo em `docs/schemas/` | Artefato Físico Regido | Consumidor / Produtor Principal |
| :---: | :--- | :--- | :--- |
| **25** | [ambiguity_findings.schema.json](./ambiguity_findings.schema.json) | `specs/<feature>/ambiguity_findings.json` | Escrito pelo LLM em `/vitalia-clarify`; validado e atualizado (`--resolve`) por `sdd_ambiguity_scorer.py`. |
| **26** | [end_payload.schema.json](./end_payload.schema.json) | `.vitalia/end_payload.json` | Escrito pelo LLM em `/vitalia-session-end` e `/vitalia-skill-evaluation`; validado por `vitalia_context_engine.py --action end`. |
| **27** | [grounding_decisions_payload.schema.json](./grounding_decisions_payload.schema.json) | `.vitalia/grounding_decisions_payload.json` | Escrito pelo LLM em `/vitalia-session-consolidate` após `ask_question`; validado por `GroundingDomainUpdater.apply_decisions()`. |
| **28** | [architect_mode_diagnostics.schema.json](./architect_mode_diagnostics.schema.json) | `.vitalia/memory/data_storage/diagnostics/architect_mode_diagnostics.json` | Produzido e validado por `sdd_architect_mode_classifier.py`; injetado em `architect.toml`. |
| **29** | [architect_modes.schema.json](./architect_modes.schema.json) | `profiles/architect_modes.yaml` | Compilado por `vitalia_layer_compiler.py` e consumido por `sdd_architect_mode_classifier.py` / `GuardianContextV2`. |
| **30** | [ambiguity_taxonomy_catalog.schema.json](./ambiguity_taxonomy_catalog.schema.json) | `profiles/ambiguity_taxonomy.yaml` | Catálogo YAML das 10 categorias de ambiguidade (distinto de `ambiguity_taxonomy.schema.json`, que valida `ambiguity_metrics.json`). |
| **31** | [data_storage_index_entry.schema.json](./data_storage_index_entry.schema.json) | `.vitalia/memory/data_storage/<cat>/index.jsonl` | Validado em `register_data_storage_item()` (`core/data_storage_io.py`) e lido pelos renderizadores e `vitalia_tmp_cleanup.py`. |
| **32** | [session_ledger.schema.json](./session_ledger.schema.json) | `.vitalia/memory/session/session_ledger.json` | Validado em `vitalia_context_engine.py --action end` e `sensor_memory_ledger.py`. |
| **33** | [project_config.schema.json](./project_config.schema.json) | `.vitalia/config.yml` | Gerado por `install-project` e lido por `kit_env_bootstrap.py`, `vitalia_hook_runner.py` e renderizadores. |
| **34** | [design_tokens.schema.json](./design_tokens.schema.json) | `docs/design/design-tokens.json` | Injetado via `guardian.schemas` em `figma-maestro.toml` (W3C DTCG + Tailwind CSS v4). |

---

## 3. Adições Cirúrgicas nos 2 Schemas Já Existentes (`#01` e `#07`)

Além de copiar os 10 novos arquivos para `source/src/schemas/`, a **Spec 009** aplica duas extensões aditivas em schemas existentes:

### 3.1 Schema `#01` (`source/src/schemas/context-lifting-envelope.schema.json`)
1. Em `properties.guardian.properties`, adicionar:
   ```json
   "schemas": {
     "type": "array",
     "items": {
       "type": "string"
     }
   }
   ```
2. Em `properties.guardian_json.properties`, adicionar:
   ```json
   "schemas": {
     "type": "object"
   }
   ```

### 3.2 Schema `#07` (`source/src/schemas/grounding_domains.schema.json` — Ponto 6c)
Em `properties.domains.items.properties`, adicionar:
```json
"hitl_on_use": {
  "type": "boolean"
}
```

