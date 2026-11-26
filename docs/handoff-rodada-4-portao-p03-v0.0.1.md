<!-- handoff-rodada-4-portao-p03-v0.0.1.md | Vitalia Kit v0.0.1 | 25-11-2026 19:54(GMT-04:00) -->
**handoff-rodada-4-portao-p03-v0.0.1.md | Vitalia Kit v0.0.1 | 25-11-2026 19:54(GMT-04:00)**

# Handoff de Evolução e Alinhamento P03 — Rodada 4 / Spec 008 (Vitalia Kit v0.0.1)

> **Destino:** Máquina/sessão responsável pela futura **Spec 008** no repositório `vitalia-kit` (`source/` e `docs/`).  
> **Origem:** Auditoria profunda de alinhamento entre o código instalado (`48c9eac` / Spec 007) e [RASCUNHO_PORTAO_GROUNDING_P03.md](./RASCUNHO_PORTAO_GROUNDING_P03.md).  
> **Template Visual de Referência:** [grounding_audit_template.md](./templates/grounding_audit_template.md)  
> **Decisões Aprovadas pelo Usuário:** **Opção A em todos os 6 pontos** + correção obrigatória dos **2 Achados Críticos Extras (A e B)**.

---

## 0. Regra Inegociável de Compilação & SDD (P01)

- Toda alteração em `source/profiles/`, `source/extensions/` ou `source/profiles/grounding_domains.yaml` ocorre **exclusivamente** em `source/layers/` e é compilada por `vitalia_layer_compiler.py` (`--check` = `PASS`).
- Toda implementação segue o pipeline SDD (`specify → plan → tasks → task-verifier → implement → converge`) com testes P05 `[TEST:BUGFIX]` / `[TEST:NEW]` / `[TEST:MIGRATION]`.

---

## 1. Achados Críticos Extras (Correção Obrigatória — Prioridade P0)

### Extra A — Restauração de `source/layers/extensions/research.toml` (Bug de Recursão Infinita)
- **Defeito Atual:** `source/extensions/research.toml` foi sobrescrito com o prompt `## Identidade — Revisor Científico de Metodologia`, que manda o próprio `research` delegar para `vitalia-researcher` (recursão infinita), além de estar com `[context] reads = [] / writes = []`.
- **Correção:**
  1. Restaurar em `source/layers/extensions/research.toml` o protocolo autocontido de **8 passos** do `vitalia-researcher` (recepção → classificação contra `grounding_domains.yaml` / `general_web` → cache 90d → `search_web` + fetch HTTP 200 + SHA-256 + snapshot `./snapshots/<sha>.txt` + `evidence.jsonl` + `urls.txt` → portão `vitalia_grounding_validator.py --mode evidence --evidence-dir ...` → renderização `grounding_audit_renderer.py` → handoff reverso via `vitalia_handoff.py create --direction reverse` → exibição de `GROUNDING_AUDIT.md`).
  2. Restaurar `[context].writes = [".vitalia/memory/data_storage/research/"]` e manter `search_web` exclusivo em `research.toml`.
  3. Recompilar e validar `vitalia_layer_compiler.py --check`.

### Extra B — Unificação do Contrato de Campos de Evidência (`evidence_record.schema.json` ↔ Validador ↔ Renderer)
- **Defeito Atual:** Quebra em 3 vias:
  - `evidence_record.schema.json` exige `url`, `fetched_at`, `domain_match`, `http_status`, `sha256`, `snapshot_path`, `literal_quote`, `status`.
  - `vitalia_grounding_validator.py --mode evidence` lê `retrieved_at` e `domain_id`/`domain_tags`, e não valida `http_status == 200`, `sha256` nem existência do `snapshot_path`.
  - `grounding_audit_renderer.py` lê `source_url`, `evidence_id`, `domain_id`, `relevance_score`, `source_tier`, `valid` e faz `json.loads()` de arquivo único (quebra em `.jsonl`).
- **Correção:** Adotar `evidence_record.schema.json` como contrato canônico único (mantendo aliases retrocompatíveis leves onde testes legados exigirem, ou migrando via `[TEST:MIGRATION]`).

---

## 2. Escopo Aprovado — Opção A nos 6 Pontos

### Ponto 1 (Opção A) — Camada 3: Checklist Pré-Síntese (`- [ ]`) nos Workflows de Concepção, Especificação e Especialistas
- **Escopo:** Adicionar o bloco verificável de checklist pré-síntese (`- [ ]`) em `source/layers/extensions/` para os workflows de concepção/especificação/revisão e os 10 especialistas (`brainstorming`, `architect`, `spec-specify`, `spec-plan`, `science-review`, `integrative-review`, `caio-strategist`, `medical-gate`, `literature-curator`, `design-thinking`, `visual-creator`, `figma-maestro`, `django-stark`, `react-vinci`, `infra-nomad`, `cortex-analyst`).
- Manter cláusula afirmativa P03 concisa nos utilitários operacionais (`session-start`, `session-end`, `session-consolidate`, `route`, `pair`, `debug`, `release`, etc.).
- Recompilar via `vitalia_layer_compiler.py --check`.

### Ponto 2 (Opção A) — `vitalia_grounding_validator.py --mode evidence` Unificado
- Suportar **tanto** `--evidence-dir <dir>` (varrendo `evidence.jsonl` de um tema) **quanto** `--evidence-file <arquivo>` (retrocompatível para JSON ou JSONL).
- Validar integralmente cada registro contra o contrato canônico:
  - `http_status == 200` (ou `status == "unverified"` degradado conforme D18, bloqueado em `--strict`);
  - `sha256` não vazio e arquivo `snapshot_path` existente em disco;
  - Janela de `90` dias (`GMT-04:00`) sobre `fetched_at` (com fallback retrocompatível para `retrieved_at`);
  - `literal_quote` presente no snapshot (obrigatório em `--strict`);
  - `hitl_escalation_required: true` obrigatório quando `domain_match` (ou `domain_id`/`domain_tags`) incluir `clinical_safety` ou `regulations_compliance`.
- Manter SRP estrito (sem `--render` no validador; renderização permanece em `grounding_audit_renderer.py`).

### Ponto 3 (Opção A) — `grounding_audit_renderer.py` Completo com JSONL e Links Relativos
- Aceitar `--evidence-dir <dir>` (lendo `<dir>/evidence.jsonl`), `--output <md>`, `--domains-yaml <yaml>` opcional e manter `--evidence-file` / `--project-root` retrocompatíveis.
- Consumir o contrato canônico de `evidence_record.schema.json` (`url`, `url_requested`, `final_url`, `http_status`, `sha256`, `snapshot_path`, `fetched_at`, `domain_match`, `literal_quote`, `status`, `hitl_escalation_required`).
- Trocar links `file://` por **links relativos** (`./snapshots/<sha>.txt`, `./evidence.jsonl`, `../README.md`, `../../README.md`) para compatibilidade total tanto na IDE quanto no GitHub (`data_repo`).
- Renderizar todas as seções do template visual: resumo numérico, Mermaid P16 de proveniência, tabela de proveniência, `<details>` de citações forenses/redirects, `<details>` de `⚠️ Domínios Não Catalogados (general_web)` e `<details>` de `Verificado vs. Não Verificado & D18` (incluindo badge `⚠️ UNVERIFIED`).

### Ponto 4 (Opção A) — Template Visual `source/templates/grounding_audit_template.md`
- Instalar o template visual elaborado em [docs/templates/grounding_audit_template.md](./templates/grounding_audit_template.md) como `source/templates/grounding_audit_template.md` (removendo apenas as 2 linhas de cabeçalho de referência do wrapper ao copiar para `source/templates/`).
- Carregar `source/templates/grounding_audit_template.md` por padrão em `GroundingAuditRenderer` mantendo fallback inline seguro se o template estiver ausente.
- *Postergado conscientemente:* `data_storage_dashboard_template.md` segue reservado para a evolução visual do `data_repo`.

### Ponto 5 (Opção A) — Contagem Informativa de Pendências `general_web` no `--action end`
- Em `source/src/vitalia_context_engine.py` (`cmd_end`), executar leitura somente-leitura via `GroundingDomainUpdater(cwd).list_pending_domains()` ao final do encerramento.
- Emitir linha informativa no stdout (ex.: `ℹ️ Domínios general_web pendentes de curadoria (session-consolidate): N`) e incluir `pending_domains_count` quando `--json` estiver ativo.
- Preservar intacta a invariante absoluta: `--action end` **nunca** cria nem altera `grounding-domains.jsonl`.

### Ponto 6 (Opção A em 6a, 6b, 6c) — `vitalia_handoff.py`, Expansão de Fontes e `hitl_on_use`
1. **6a (`vitalia_handoff.py create`):** Adicionar flag opcional `--direction {forward,reverse}` que grava `envelope.direction` no handoff criado (preservando 100% de retrocompatibilidade quando omitida e mantendo proibição de `$schema` P11). Atualizar `vitalia-handoff-v1.0.0.json` se necessário mantendo OpenAPI 3.0.3 limpo.
2. **6b (`source/layers/grounding/` → `profiles/grounding_domains.yaml`):** Expandir as fontes autoritativas tipadas (todos os campos `url`, `source_tier`, `source_type`, `authority_scope`, `volatility`) em:
   - `web_frameworks`: adicionar FastAPI (`https://fastapi.tiangolo.com/`), Vue (`https://vuejs.org/`).
   - `devops_ci`: adicionar Terraform (`https://developer.hashicorp.com/terraform/docs`).
   - `cloud_services`: adicionar AWS (`https://docs.aws.amazon.com/`) e Azure (`https://learn.microsoft.com/en-us/azure/`).
   - `design_patterns`: adicionar Martin Fowler (`https://martinfowler.com/`).
   - `javascript_ecosystem`: adicionar Node.js (`https://nodejs.org/docs/latest/api/`) e npm (`https://docs.npmjs.com/`).
   - `provedores_de_ia`: adicionar Hugging Face (`https://huggingface.co/docs`) e LangChain (`https://docs.langchain.com/`).
3. **6c (`grounding_domains.schema.json` + `general_web`):** Adicionar propriedade booleana opcional `hitl_on_use` em `source/src/schemas/grounding_domains.schema.json` (OpenAPI 3.0.3 limpo P11) e declarar `hitl_on_use: false` em `general_web` (e `true` opcionalmente onde aplicável, consumido pelo validador/researcher).

---

## 3. Higiene Cosmética Adicional (Fora de `source/`)

- Atualizar `vitalia-kit/README.md`:
  - Linha 5: `Vitalia SPEC Kit` → `Vitalia Kit`.
  - Linha 27: `24 Guardian Profiles` → `25 Guardian Profiles`.
  - Linha 147: `173 testes automatizados` → contagem atual (`202+`).
- Atualizar `docs/RASCUNHO_PORTAO_GROUNDING_P03.md` para refletir a decisão de links relativos (em vez de `file://` no D8) e SRP sem `--render` no validador.

---

## 4. Critérios de Aceitação para Fechamento da Spec 008

1. `vitalia_layer_compiler.py --layers-dir source/layers --out-dir source --check --json` → `PASS` (`0` derivas).
2. Teste E2E do pipeline completo do `researcher`: `evidence.jsonl` (válido + `unverified` + `general_web` + domínio crítico) → `vitalia_grounding_validator.py --mode evidence --evidence-dir ...` → `grounding_audit_renderer.py --evidence-dir ...` gerando `GROUNDING_AUDIT.md` sem placeholders crus e sem links `file:///` → `vitalia_handoff.py create --direction reverse` → `vitalia_context_engine.py --action end` exibindo contagem de pendentes sem tocar `grounding-domains.jsonl`.
3. `test_schemas_openapi30_p11.py` e toda a suíte `pytest source/tests/` com 100% de aprovação (dual-CWD).
4. `header_stamp.py --check --json` e todos os sensores VRSS (`PASS`).

