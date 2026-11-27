<!-- handoff-spec-009-determinismo-schemas-camada3-v0.0.1.md | Vitalia Kit v0.0.1 | 27-11-2026 10:35(GMT-04:00) -->
**handoff-spec-009-determinismo-schemas-camada3-v0.0.1.md | Vitalia Kit v0.0.1 | 27-11-2026 10:35(GMT-04:00)**

# Handoff de Engenharia — Spec 009: Schemas Faltantes, Determinismo Python, Fluxo 1A P03 e Higiene da Camada 3 (Vitalia Kit v0.0.1)

> **Destino:** Sessão/máquina responsável pela execução da **Spec 009** (`specs/009-determinismo-schemas-camada3/`) no repositório `vitalia-kit` (`HEAD >= 8ca2398`).  
> **Oráculo Canônico de Estado-Alvo (Âncora para `SC-xxx`):** [CONTRATOS_WORKFLOWS_SDD.md](./CONTRATOS_WORKFLOWS_SDD.md) (já atualizado pós-descongelamento da `D-20261124-01` com as Seções 7, 7.5, 8 e as definições JSON integrais dos **36 schemas `#01` a `#36`** na Seção 11.2).  
> **Documento Base de Auditoria:** [brainstorming-auditoria-kit-ativo-spec-009-010-v0.0.1.md](./brainstorming-auditoria-kit-ativo-spec-009-010-v0.0.1.md) (Seções 2 a 7.1).  
> **Artefatos Prontos para Cópia:** [docs/schemas/](./schemas/README.md) (`12` novos arquivos `.schema.json` em OpenAPI 3.0.3 Limpo — P11, cobrindo 100% dos `.json`, `.jsonl` e `.yaml`/`.yml`).  
> **Etapa Subsequente (Separada por Design):** [handoff-spec-010-contratos-toml-catalogos-v0.0.1.md](./handoff-spec-010-contratos-toml-catalogos-v0.0.1.md).

---

## 0. Diretrizes Inegociáveis de Escopo, Pipeline SDD (P01) e TDD (P05)

1. **Fronteira Estrita da Spec 009 (Decisões `8a012998` e `a27ce8d0`):**
   - A Spec 009 concentra **100% da engenharia determinística em Python (`source/src/`) e contratos de dados (`source/src/schemas/`)**.
   - **Fora de escopo da Spec 009 (reservado à Spec 010):** reescrita declarativa dos `34` arquivos `source/layers/extensions/*.toml` e adição das `10` novas fontes autoritativas externas em `source/layers/grounding/`. Entregar e testar primeiro as CLIs determinísticas e a injeção `guardian.schemas` garante que a Spec 010 apoie-se sobre primitivas já provadas.
2. **Inviolabilidade do Ciclo SDD (`P01`) e Matriz `P05`:**
   - Executar o fluxo completo (`specify → clarify → plan → tasks → task-verifier → analyze → implement → converge`) comprovando **Red Phase $\rightarrow$ Green Phase** em cada tarefa e classificando explicitamente `[TEST:NEW]`, `[TEST:BUGFIX]`, `[TEST:REFACTOR]` e `[TEST:MIGRATION]`.
3. **Política de Retrocompatibilidade (P05):**
   - Manter retrocompatibilidade nas assinaturas CLI já consumidas pelos testes das Specs 005–008 (`--payload` em `vitalia_handoff.py create`, `--input` e JSON com `weights` em `sdd_ambiguity_scorer.py`).
   - **Zero retrocompatibilidade em `GroundingDomainUpdater` (`N2`):** remover qualquer fallback legado para `grounding_decision == "pending"` e migrar explicitamente as fixtures de teste para o contrato canônico (`evidence_record.schema.json` + Via Derivada) sob `[TEST:MIGRATION]`.

---

## 1. Bloco 1 — Expansão Canônica de `source/src/schemas/` (`24` $\rightarrow$ `36` Schemas OpenAPI 3.0.3 P11)

### 1.1 Cópia dos 12 Novos Schemas (`#25` a `#36`)
Copiar os 12 schemas validados em `vitalia-docs/docs/schemas/*.schema.json` para `source/src/schemas/`:
1. `ambiguity_findings.schema.json` (`#25` — entrada semântica de `/vitalia-clarify`: `schema_version`, `spec_file`, `ambiguities` + alias opcional `detected_ambiguities`).
2. `end_payload.schema.json` (`#26` — payload transitório `.vitalia/end_payload.json` consumido por `vitalia_context_engine.py --action end`).
3. `grounding_decisions_payload.schema.json` (`#27` — payload transitório `.vitalia/grounding_decisions_payload.json` do Fluxo 1A).
4. `architect_mode_diagnostics.schema.json` (`#28` — saída RRF de `sdd_architect_mode_classifier.py`).
5. `architect_modes.schema.json` (`#29` — catálogo compilado `profiles/architect_modes.yaml`, exigindo `["id", "concern", "file_triggers", "keywords"]` com `layer`, `name`, `description` opcionais).
6. `ambiguity_taxonomy_catalog.schema.json` (`#30` — catálogo YAML `profiles/ambiguity_taxonomy.yaml`, separado de `ambiguity_taxonomy.schema.json` que valida o relatório `ambiguity_metrics.json`).
7. `data_storage_index_entry.schema.json` (`#31` — linha de `.vitalia/memory/data_storage/<cat>/index.jsonl`).
8. `session_ledger.schema.json` (`#32` — documento raiz `.vitalia/memory/session/session_ledger.json`).
9. `project_config.schema.json` (`#33` — configuração `.vitalia/config.yml`).
10. `design_tokens.schema.json` (`#34` — contrato W3C DTCG / Tailwind v4 de `docs/design/design-tokens.json` para `figma-maestro`).
11. `agent_preserver_manifest.schema.json` (`#35` — manifesto SHA-256 de skills `.vitalia/agent_preserver.manifest.json`).
12. `extensions_config.schema.json` (`#36` — configuração `.vitalia/extensions.yml`).

### 1.2 Adições Cirúrgicas nos Schemas Existentes `#01` e `#07`
1. **`source/src/schemas/context-lifting-envelope.schema.json` (`#01`):**
   - Em `properties.guardian.properties`, adicionar `"schemas": {"type": "array", "items": {"type": "string"}}`.
   - Em `properties.guardian_json.properties`, adicionar `"schemas": {"type": "object"}`.
2. **`source/src/schemas/grounding_domains.schema.json` (`#07` — Ponto 6c):**
   - Em `properties.domains.items.properties`, adicionar `"hitl_on_use": {"type": "boolean"}`.

### 1.3 Migração Obrigatória em `source/tests/test_schemas_openapi30_p11.py` (`[TEST:MIGRATION]`)
> [!IMPORTANT]
> **Armadilha Real Detectada na Inspeção:** `source/tests/test_schemas_openapi30_p11.py` (`L17-42, L71-78`) possui `EXPECTED_24_SCHEMAS` e a asserção rígida `assert len(actual_schemas) == 24`.
- Atualizar a lista canônica para os **36 schemas** (`EXPECTED_36_SCHEMAS`, mantendo alias ou migrando o teste via `[TEST:MIGRATION]`), parametrizando `test_schema_clean_openapi30_rules` sobre os 36 arquivos e adicionando validação cruzada de `profiles/architect_modes.yaml` contra `architect_modes.schema.json` e `profiles/ambiguity_taxonomy.yaml` contra `ambiguity_taxonomy_catalog.schema.json`.

---

## 2. Bloco 2 — Refatoração Determinística de `source/src/sdd_ambiguity_scorer.py` (`/vitalia-clarify`)

### 2.1 Defeitos Atuais a Reproduzir em Red Phase (`[TEST:BUGFIX]`)
1. **`json.loads()` em arquivos YAML (`L198` e `L221`):** Quebra com `JSONDecodeError` ao receber `profiles/ambiguity_scoring.yaml` e `profiles/ambiguity_taxonomy.yaml`; no bloco `--taxonomy`, captura a exceção imprimindo apenas `AVISO` e retorna `exit 0` (violando *fail-closed* P11/P12).
2. **Chaves fantasmas e fórmula incorreta em `score_full()` (`L91-141`):** Lê `weights` e `severity_multipliers` (inexistentes em `ambiguity_scoring.yaml`), calculando `1.0` para todos os itens sem `priority_score` prévio, e divide `defect_density` por `fr_count` em vez de `(open_count / max(total_lines, 1)) * 100`.
3. **Confusão de schemas e escrita não-atômica (`L222, L261`):** Tenta usar o catálogo YAML de `--taxonomy` como schema do `jsonschema.validate(instance=result, ...)` e grava a saída com `write_text()` direto.

### 2.2 Especificação Determinística da Correção (`[TEST:BUGFIX]` + `[TEST:NEW]`)
1. **Loader Universal YAML/JSON e Defaults Automáticos:**
   - Implementar `_load_yaml_or_json(path: Path) -> Any` usando `yaml.safe_load` para `.yaml`/`.yml` e `json.loads` para `.json` (rejeitando arquivo inválido com `exit 1`).
   - Ativar `full_mode` sempre que `--findings` for informado; quando `--scoring` ou `--taxonomy` forem omitidos, resolver automaticamente `<kit_dir>/profiles/ambiguity_scoring.yaml` e `<kit_dir>/profiles/ambiguity_taxonomy.yaml` via `kit_env_bootstrap.resolve_kit_dir()`.
2. **Validação de Entrada e Catálogo (`Fail-Closed`):**
   - Aceitar a lista de entrada em `findings_data` tanto na chave canônica `"ambiguities"` quanto no alias legado `"detected_ambiguities"`.
   - Quando o arquivo usar o contrato canônico (`"ambiguities"`), validar *fail-closed* (`exit 1`) contra `src/schemas/ambiguity_findings.schema.json`.
   - Quando `--taxonomy` apontar para um catálogo YAML (`schema_type == "ambiguity_taxonomy"` / `"categories"`), validá-lo contra `src/schemas/ambiguity_taxonomy_catalog.schema.json` e verificar se `amb["category"]` pertence às categorias ativas; se um teste legado passar um JSON Schema em `--taxonomy` (`"properties"` na raiz), validá-lo sem suprimir exceções (`exit 1` em falha).
3. **Cálculo FMEA / NASA ARM sobre `ambiguity_scoring.yaml`:**
   - Quando `scoring_config` contiver `"impact_scale"` (formato real do kit):
     - Converter `impact` (número `1.0`–`5.0` ou rótulo em `impact_scale`: `low=1.0, medium=2.0, high=3.5, critical=5.0`).
     - Converter `uncertainty` (número `1.0`–`3.0` ou rótulo em `uncertainty_scale`: `low=1.0, medium=2.0, high=3.0`).
     - Resolver `domain_multiplier` em `domain_multipliers` via `domain` / `domain_id` / `domain_weight` (fallback `general: 1.0`).
     - Calcular `priority_score = round(impact_val * uncertainty_val * domain_mult, 4)`.
     - Derivar `severity` comparando `priority_score` com `severity_thresholds` ordenados de forma decrescente (`critical >= 15.0`, `high >= 8.0`, `medium >= 4.0`, `low >= 0.0`).
   - Preservar fallback para `weights` $\times$ `severity_multipliers` **apenas** quando `"weights"` estiver explicitamente presente em `scoring_config` (garantindo 0 regressão em testes legados).
4. **Ciclo `open` $\rightarrow$ `resolved` e Nova Flag CLI `--resolve`:**
   - Adicionar `--resolve <AMB-ID>` e `--resolution <texto>`:
     - Localiza o item `id == args.resolve` em `--findings` (erro `exit 1` se ID inexistente), atualiza `"status": "resolved"` e `"resolution": args.resolution` e persiste `--findings` via escrita atômica (`atomic_write_json`).
   - Separar `open_items = [a for a in scored if a.get("status", "open") != "resolved"]` e `resolved_items = [a for a in scored if a.get("status") == "resolved"]`:
     - `total_ambiguities`: `len(open_items)`
     - `resolved_ambiguities`: `len(resolved_items)`
     - `total_lines`: número de linhas de `--spec` (ou `1` se `--spec` omitido/vazio)
     - `defect_density`: `round((len(open_items) / max(total_lines, 1)) * 100, 4)`
     - `categories_breakdown` e `severity_breakdown`: calculados sobre `open_items`
     - `detected_ambiguities`: `open_items` ordenados por `(-priority_score, id)`
5. **Validação de Saída e Escrita Atômica:**
   - Validar `result` em `full_mode` **sempre** contra `src/schemas/ambiguity_taxonomy.schema.json` via `jsonschema.validate()` (`exit 1` em violação) e gravar `--output` via `core.data_storage_io.atomic_write_text`.

---

## 3. Bloco 3 — Modo Híbrido Determinístico em `source/src/vitalia_handoff.py`

### 3.1 Flags CLI de 1ª Classe em `vitalia_handoff.py create` (`[TEST:NEW]` + `[TEST:MIGRATION]`)
Tornar `--payload` **opcional** e `--handoff-dir` com default `".vitalia/memory/data_storage/handoffs"`, adicionando as flags contratuais usadas pelos workflows:
- `--source <workflow>` $\rightarrow$ `envelope.source_workflow`
- `--target <workflow>` $\rightarrow$ `envelope.target_workflow`
- `--direction {forward,reverse,fallback}` (default `"forward"` quando sem `--payload`; override quando passado com `--payload`)
- `--intent <intent_type>` (default inteligente: `"subagent_response"` se `direction == "reverse"`, `"brainstorming_to_route"` se `source == "brainstorming"`, `"route_dispatch"` se `source == "route"`, senão `"sdd_pipeline_advance"`)
- `--input <texto>` $\rightarrow$ `intent.source_input` (default `""`)
- `--summary <texto_ou_path>` $\rightarrow$ `intent.normalized_summary` (default `""`)
- `--domain <dominio>` $\rightarrow$ `intent.detected_domain` (opcional)
- `--feature <feature_dir>` $\rightarrow$ `intent.feature_context` e `session_snapshot.active_feature` (se omitido, lido automaticamente de `.vitalia/feature.json`, fallback `""`)
- `--command <slash_cmd>` $\rightarrow$ `routing.suggested_command` (default `f"/vitalia-{target}"`)
- `--confidence <float>` $\rightarrow$ `intent.confidence` (default `1.0`)

### 3.2 Síntese, Validação P11 e Indexação Automática na Camada 3
- Se `--payload` for fornecido: carregar o JSON, rejeitar `"$schema"` (`exit 1` — P11) e aplicar quaisquer flags CLI explícitas como override.
- Se `--payload` for omitido: exigir `--source` e `--target` (`exit 1` se ausentes) e sintetizar os 4 blocos completos (`schema_type: "workflow_handoff"`, `schema_version: "1.0.0"`, `envelope` com `uuid4()`, `created_at` em `GMT-04:00`, `ttl_seconds: 86400`, `machine_id: get_machine_id()`, `status: "pending"`, `intent`, `session_snapshot`, `routing` com `confirmed_by_user: True`, `fallback_used: direction == "fallback"`).
- Validar *fail-closed* contra `src/schemas/vitalia-handoff-v1.0.0.json`.
- Gravar `<safe_id>.json` atomicamente, gerar a view Nível 2 `<safe_id>.md` via `render_item_view_md()` (com `atomic_write_text`) e registrar em `<handoff_dir>/index.jsonl` via `register_data_storage_item()`.

---

## 4. Bloco 4 — Fechamento do Fluxo 1A P03 (`N2` + Ponto 5 + `N3`)

1. **`source/src/core/grounding_domain_updater.py` (`N2` + preservação 6c):**
   - Em `list_pending_domains()`:
     - Carregar domínios já catalogados em `<kit_dir_resolvido>/profiles/grounding_domains.yaml` (excluindo `"general_web"`) e domínios/URLs já decididos (`approved` ou `rejected`) em `.vitalia/memory/data_storage/grounding_decisions.jsonl`.
     - Ler `.vitalia/memory/data_storage/research/*/evidence.jsonl`:
       - **Contrato canônico pós-Spec 008 (Zero Retrocompatibilidade):** identificar pendências exclusivamente pela **Via Derivada** (`rec.get("domain_match") == "general_web"` cujo `domain_candidate` — extraído de `rec.get("suggested_domain")` ou hostname normalizado de `final_url`/`url` — ainda não conste em `grounding_decisions.jsonl` nem nos domínios catalogados), retornando chaves canônicas (`domain_match`, `url`, `sha256`, `theme`).
       - Remover suporte legado a `grounding_decision == "pending"` e migrar fixtures de teste via `[TEST:MIGRATION]`.
   - Em `apply_decisions()`: quando receber `dict` com `"decisions"`, validar contra `grounding_decisions_payload.schema.json`.
   - Em `_promote_to_catalog()`: preservar `hitl_on_use` (`bool`) e `constitution_principles` (`list[str]`) tanto quando informados na decisão quanto quando já existentes na entrada prévia do catálogo.
2. **`source/src/rendering/dashboard_renderer.py` (`N2`):**
   - Atualizar `_count_pending_grounding()` para usar `GroundingDomainUpdater(self.project_root).list_pending_domains()` quando `research/` existir (mantendo fallback sobre `grounding_rows` apenas quando `research/` não existir), garantindo contagem idêntica entre `session/README.md` e `--action list-pending-domains`.
3. **`source/src/vitalia_context_engine.py` (`N3` + Ponto 5 + Schemas `#26`/`#32`):**
   - **`N3` (`L128`):** substituir `get_machine_identity()` por `get_machine_identity()[0]` em `mname = str(payload.get("machine_name") or get_machine_identity()[0])`, eliminando de vez a gravação da tupla `"('servido', '72a556ad')"`.
   - Em `cmd_end()`: validar `payload` contra `end_payload.schema.json` e validar `ledger` completo contra `session_ledger.schema.json` antes de gravar em disco.
   - **Ponto 5 (`cmd_end`):** realizar leitura estritamente *read-only* via `pending_count = len(GroundingDomainUpdater(project_root=cwd).list_pending_domains())` (mantendo intacta a invariante de jamais tocar em `grounding-domains.jsonl` durante `--action end`):
     - Em modo `--json`: incluir `"pending_domains_count": pending_count`.
     - Em modo texto: emitir `ℹ️ Domínios general_web pendentes de curadoria (session-consolidate): {pending_count}` quando `pending_count > 0` (ou sempre na linha de resumo).
   - Em `cmd_list_pending_domains()`: exibir `domain_id`/`domain_candidate` e `url`/`source_url` sem `KeyError`.

---

## 5. Bloco 5 — Evolução de `guardian_context.py` & `vitalia_hook_runner.py` (`N4`/`N5`, `guardian_json.schemas`, TTL e `p0`)

1. **Remoção de legado (`N4`) e Ordenação Cronológica Real (`N5`) em `source/src/hooks/guardian_context.py`:**
   - Remover `self.cwd / ".vitalia" / "data"` da lista de diretórios em `_load_historical_context()` (`L148`).
   - Substituir `entries.sort(key=lambda x: str(x.get("timestamp", "")), reverse=True)` (`L165`) por ordenação cronológica com parser `DD-MM-YYYY HH:MM[:SS](GMT-04:00)` (evitando que `28-10-2026` fique acima de `02-11-2026`).
2. **Injeção Automática de `guardian.schemas` em `guardian_json["schemas"]` (`[TEST:NEW]`):**
   - Implementar `load_schema(self, schema_filename: str) -> Optional[Dict[str, Any]]` buscando em `self.kit_root / "src" / "schemas" / schema_filename`, `self.kit_root / "source" / "src" / "schemas" / schema_filename` e `self.cwd / "source" / "src" / "schemas" / schema_filename` (rejeitando path traversal `..` ou barras `/` — P12).
   - Em `build_context()`: carregar todos os nomes listados em `self.guardian_config.get("schemas", [])` para `result_payload["schemas"] = loaded_schemas` (dict `{filename: schema_dict}`).
3. **Filtro de TTL e `$schema` em `latest_handoff()` e `p0` em `_load_session_context(profile)`:**
   - Em `latest_handoff()`: ignorar handoffs com `"$schema"` na raiz (P11) e handoffs `"pending"` cujo `created_at + ttl_seconds` já tenha expirado (`_is_expired`).
   - Em `_load_session_context(profile)`: respeitar `sc_cfg = profile.get("session_context", {})`; quando `sc_cfg.get("include_p0", True)` estiver ativo e `session_ledger.json` contiver shards, popular `session_ctx["p0"] = shards[0].get("next_step", "")`.
4. **Escritas Atômicas e Placeholder Defensivo:**
   - Usar `atomic_write_json` em `GuardianContextV2.export_and_publish()` (`L285`).
   - Usar `atomic_write_text` ao gravar `context_dir / f"{item_id}.md"` em `vitalia_hook_runner.py:379` e adicionar `cmd = cmd.replace("{{PROJECT_ROOT}}", cwd)` em `resolve_placeholders()`.

---

## 6. Bloco 6 — `sdd_architect_mode_classifier.py`, Indexação `index.jsonl` e Limpeza Física (`vitalia_tmp_cleanup.py`)

1. **Blindagem de `source/src/sdd_architect_mode_classifier.py` (`[TEST:BUGFIX]` + `[TEST:NEW]`):**
   - Aceitar aliases CLI compatíveis com `architect.toml`: `--project-dir` / `--workspace`, `--modes-file` / `--catalog`, além de `--args` (texto livre tokenizado por regex `[a-zA-Z0-9_\+\-]+` somado às keywords de `--signals`) e `--feature` (opcional, incluído em `active_feature`).
   - Validar o catálogo YAML carregado contra `architect_modes.schema.json` e validar o dicionário `diag` de saída contra `architect_mode_diagnostics.schema.json` (`fail-closed`, `exit 1`).
   - Gravar `--output` com `atomic_write_json` e, quando o destino estiver em `data_storage/diagnostics/`, gerar view `.md` Nível 2 e registrar em `diagnostics/index.jsonl` via `register_data_storage_item()`.
2. **Validação e Registro em `index.jsonl` (`data_storage_io.py`, `grounding_audit_renderer.py`, `data_storage_dashboard_renderer.py`):**
   - Em `core/data_storage_io.py:register_data_storage_item()`: validar `entry` contra `data_storage_index_entry.schema.json`.
   - Em `rendering/grounding_audit_renderer.py:main()`: quando `out_path` for gravado dentro de `.vitalia/memory/data_storage/research/<tema>/GROUNDING_AUDIT.md`, chamar `register_data_storage_item(research_root, {"id": topic, "title": f"Auditoria P03: {topic}", "json_path": f"{topic}/evidence.jsonl", "md_path": f"{topic}/GROUNDING_AUDIT.md", "category": "research"})`.
   - Em `rendering/data_storage_dashboard_renderer.py` e `tools/context_saturation_monitor.py`: substituir `write_text` direto por `atomic_write_text` (e aplicar `_shield_escape` nos badges de `data_storage_dashboard_renderer.py`).
3. **Varredura Física de Órfãos e Remoção Completa de `research/<tema>/` em `source/src/vitalia_tmp_cleanup.py` (`[TEST:BUGFIX]`):**
   - Em `_collect_eligible()`:
     - Para `context` e `diagnostics`: varrer fisicamente `cat_dir.rglob("*.json")` e `cat_dir.rglob("*.log")` (ignorando `index.jsonl` e arquivos modificados hoje `_is_today(f)`), capturando **100% dos arquivos órfãos** fora de `index.jsonl` (alinhando `vitalia_tmp_cleanup.py` com `scan_environment.py`!).
     - Para `research`: quando o diretório `tema_dir = ev_file.parent` tiver $> 90$ dias e nenhum arquivo modificado hoje, registrar `{"path": tema_dir, "category": "research", "reason": "research > 90d", "is_dir": True}`.
   - Em `_remove_item()`: se `item.get("is_dir")` for `True` (ou `path.is_dir()`), remover a pasta completa `<tema>/` (`GROUNDING_AUDIT.md`, `evidence.jsonl`, `urls.txt`, `snapshots/`) via `shutil.rmtree(path)`, e ao final chamar `resync_category_index(cat_dir)`.

---

## 7. Matriz Mínima de Testes Comportamentais P05 & Critérios de Aceitação

| ID | Modalidade P05 | Alvo | Cenário de Falsificação Obrigatório |
| :---: | :---: | :--- | :--- |
| **V1** | `[TEST:MIGRATION]` | `test_schemas_openapi30_p11.py` | Falha se qualquer um dos 36 schemas contiver `$schema`, `title` raiz, `additionalProperties: false`, `type` array/`null`, ou se `architect_modes.yaml` / `ambiguity_taxonomy.yaml` violarem seus novos schemas. |
| **V2** | `[TEST:BUGFIX]` | `sdd_ambiguity_scorer.py` | Falha se passar `profiles/ambiguity_scoring.yaml` + `profiles/ambiguity_taxonomy.yaml` reais lançar `JSONDecodeError`, se `priority_score` ignorar `impact_scale`/`domain_multipliers` ou se `--taxonomy` corrompido retornar `exit 0`. |
| **V3** | `[TEST:NEW]` | `sdd_ambiguity_scorer.py --resolve` | Falha se resolver `AMB-001` não reduzir `total_ambiguities` e `defect_density` até `0.0` preservando `resolved_ambiguities`. |
| **V4** | `[TEST:NEW]` | `vitalia_handoff.py create` | Falha se invocação apenas por flags CLI (`--source`, `--target`, `--direction`, `--summary`) falhar, omitir `handoffs/index.jsonl` ou emitir `$schema`, ou se `--payload` legado quebrar. |
| **V5** | `[TEST:MIGRATION]` + `[TEST:BUGFIX]` | Fluxo 1A (`N2` + `N3` + Ponto 5 + **Guardrail G3**) | Falha se `evidence.jsonl` com `domain_match: "general_web"` retornar `0` pendentes, se `machine_name` ou `source_origin` gravarem tupla `"('...', '...')"`, ou se `--action end` mutar `grounding-domains.jsonl`. |
| **V6** | `[TEST:NEW]` | `GuardianContextV2` (`schemas`, TTL, `$schema`, `p0`, `N5` + **Guardrail G3**) | Falha se `guardian.schemas` não for injetado em `guardian_json["schemas"]`, se handoff expirado ou com `"$schema"` for injetado em `session_context.handoff`, ou se na virada de mês `25-11-2026` ordenar acima de `01-12-2026`. |
| **V7** | `[TEST:BUGFIX]` | `vitalia_tmp_cleanup.py` + `scan_environment.py` | Falha se arquivos `.json`/`.log` órfãos fora de `context/index.jsonl` permanecerem após `--types context --force` mantendo `needs_cleanup: true`, ou se `research/<tema>/snapshots/` sobreviver após limpeza $> 90$d. |
| **V8** | `[TEST:NEW]` | Fluxo 1A E2E (**Guardrail G4**) | Falha se o ciclo completo (`research/evidence.jsonl` com `general_web` $\rightarrow$ `list-pending-domains` $\rightarrow$ `session-consolidate` $\rightarrow$ `grounding_decisions.jsonl` $\rightarrow$ `grounding_domains.yaml` $\rightarrow$ `pending == 0`) quebrar em qualquer etapa. |

### Portão Final de Fechamento da Spec 009
1. Paridade 100% dos **36 schemas** em `source/src/schemas/` contra a Seção 11.2 do Oráculo [CONTRATOS_WORKFLOWS_SDD.md](./CONTRATOS_WORKFLOWS_SDD.md) e `test_schemas_openapi30_p11.py` $\rightarrow$ `PASS`.
2. `<kit_python> source/src/vitalia_layer_compiler.py --layers-dir source/layers --out-dir source --check --json` $\rightarrow$ `PASS`.
3. `<kit_python> -m pytest source/tests/` (executado em dual-CWD) $\rightarrow$ `100% PASS`.
4. `<kit_python> source/src/maintenance/header_stamp.py --check --root . --json` e sensores VRSS (`sensor_purity.py`, `sensor_path_hygiene.py`, `sensor_tokens_leak.py`, `sensor_memory_ledger.py`) $\rightarrow$ `PASS`.
5. Sincronizar o runtime instalado (`~/.vitalia-kit`) via `./source/kit-bootstrap.sh` antes de iniciar a **Spec 010**.
