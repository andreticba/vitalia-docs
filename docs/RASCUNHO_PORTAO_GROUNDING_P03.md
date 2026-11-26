<!-- RASCUNHO_PORTAO_GROUNDING_P03.md | Vitalia Kit v0.0.1 | 25-11-2026 15:33(GMT-04:00) -->
**RASCUNHO_PORTAO_GROUNDING_P03.md | Vitalia Kit v0.0.1 | 25-11-2026 15:33(GMT-04:00)**

# Rascunho: Portão Obrigatório de Grounding P03 — Estratégia Consolidada (Rev. 4)

> [!IMPORTANT]
> **Status:** Rev. 4 consolidada e sincronizada com [specs/005-portao-grounding-p03/plan.md](file:///home/andre/projetos/assistidos/vitalia-kit/specs/005-portao-grounding-p03/plan.md), [MEMORANDO DE ARQUITETURA.md](file:///home/andre/projetos/assistidos/vitalia-kit/docs/MEMORANDO%20DE%20ARQUITETURA.md) (ADR-0012 / ADR-0013) e [CONTRATOS_WORKFLOWS_SDD.md](file:///home/andre/projetos/assistidos/vitalia-kit/docs/CONTRATOS_WORKFLOWS_SDD.md) (`19-11-2026`).
> Preserva o diagnóstico e a base histórica da Rev. 3 de brainstorming (`17-11-2026`), incorporando a Defesa P03 Universal nos `33` workflows sem `search_web`, o fluxo unidirecional `1A` (`GroundingDomainUpdater` $\rightarrow$ Motor na máquina $\rightarrow$ `GroundingRenderer` $\rightarrow$ `DashboardRenderer`), a aposentadoria de `grounding-domains-local.yaml`, os 2 níveis `.md` em `data_storage/` (`2A`) e o destino limpo `~/.vitalia-kit/`.

---

## 1. Problema Raiz — Viés de Atalho por Ferramenta Disponível

### 1.1 Diagnóstico
O LLM em workflows de concepção (brainstorming, architect, spec-specify, spec-plan) teve acesso simultâneo a `search_web` (ferramenta nativa da IDE, execução instantânea) e à instrução em prosa de delegar ao `vitalia-researcher` (sub-agente assíncrono com protocolo complexo). **O viés cognitivo do modelo favorece o caminho de menor latência e menor complexidade de orquestração**, especialmente em contextos com muitos tokens de instrução (Lost in the Middle).

### 1.2 Falha Estrutural
A obrigação de delegar ao vitalia-researcher existia apenas como instrução textual no prompt — **sem validador determinístico que impeça o avanço sem evidência gravada** em `.vitalia/memory/data_storage/research/`.

### 1.3 Regra Fundamental (Validada)
> Agentes de alto nível **sempre** consultam fontes externas porque tratam assuntos de vanguarda, onde padrões da indústria e o estado da arte são imprescindíveis para o critério de sucesso esperado. A consulta externa não é condicional — é pré-requisito estrutural.

### 1.4 Fundamentação no Estado da Arte
A pesquisa identifica este problema como **"Silent Wrong-State Failure"** — até 78% das falhas em agentes com tool-use não produzem exceção ou erro visível (arxiv.org). O agente completa a tarefa violando a lógica de negócio sem levantar flag. O padrão **Fail-Closed Evidence Gate** (Seção 3.4) é o mecanismo recomendado pela indústria para mitigar este tipo de falha.

---

## 2. Arquitetura do Researcher Autocontido (SRP)

### 2.1 Princípio Central (Validado)
O researcher é um **workflow autocontido** que encapsula todo o ciclo de busca, validação, auditoria e emissão de artefatos. O workflow invocador (brainstorming, architect, etc.) **não se preocupa com a mecânica de busca** — delega e recebe o resultado pronto. Cada componente respeita SRP.

### 2.2 Capacidades do Researcher

1. **Invocável como sub-agente** por qualquer workflow de concepção.
2. **Invocável como slash command** (`/vitalia-research`) pelo usuário para buscas independentes e/ou inclusão de domínios não catalogados.
3. **Validação interna (fail-closed):** Se a evidência é inválida, ela não sai do researcher. O researcher conclui validação, auditoria, geração de artefato .md e handoff de retorno ao invocador — tudo internamente.
4. **Detecção de domínios não catalogados:** Quando identifica um tema que cai em `general_web`, registra `⚠️ Domínio não catalogado` no `GROUNDING_AUDIT.md` com sugestão de inclusão. **O researcher NÃO toca em `grounding_domains.yaml`** — apenas encontra e armazena as informações para atualização no final da sessão via session-end/consolidate.
5. **Cache de evidências:** Aceita `evidence.jsonl` de buscas anteriores dentro do `default_validity_days` (90 dias).

### 2.3 Fluxo Interno Autocontido do Researcher (Completo)

```
╔══════════════════════════════════════════════════════════════════╗
║                RESEARCHER — FLUXO AUTOCONTIDO                   ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  1. RECEPÇÃO                                                     ║
║     ├─ Recebe query (via sub-agente ou slash command /research)  ║
║     └─ Identifica tema e cria diretório:                         ║
║        .vitalia/memory/data_storage/research/<tema>/             ║
║                                                                  ║
║  2. CLASSIFICAÇÃO DE DOMÍNIO                                     ║
║     ├─ Lê grounding_domains.yaml                                ║
║     ├─ Classifica query contra domínios catalogados              ║
║     ├─ Se domínio catalogado → aplica sources, trigger, tier    ║
║     └─ Se NÃO catalogado → modo general_web                    ║
║        └─ Registra ⚠️ no evidence.jsonl:                        ║
║           {"domain_match": "general_web", "suggested_domain": …} ║
║                                                                  ║
║  3. VERIFICAÇÃO DE CACHE (90 dias)                               ║
║     ├─ Busca evidence.jsonl existente para URLs do domínio       ║
║     ├─ Se registro < 90 dias + http_status 200 → REUTILIZA      ║
║     └─ Se ausente ou expirado → prossegue para busca            ║
║                                                                  ║
║  4. EXECUÇÃO DE BUSCAS                                           ║
║     ├─ Usa search_web (BypassSandbox: true)                     ║
║     ├─ Para cada resultado relevante:                            ║
║     │   ├─ Faz fetch da URL completa                            ║
║     │   ├─ Registra HTTP status code                            ║
║     │   ├─ Calcula SHA-256 do conteúdo                          ║
║     │   ├─ Salva snapshot em snapshots/<sha256_prefix>.txt      ║
║     │   └─ Grava registro em evidence.jsonl                     ║
║     └─ Grava urls.txt (lista flat das URLs visitadas)           ║
║                                                                  ║
║  5. VALIDAÇÃO INTERNA — PORTÃO FAIL-CLOSED                       ║
║     ├─ Executa vitalia_grounding_validator.py --mode evidence    ║
║     │   ├─ Verifica CADA registro em evidence.jsonl:            ║
║     │   │   ├─ http_status == 200?                              ║
║     │   │   ├─ sha256 não vazio?                                ║
║     │   │   ├─ snapshot existe no filesystem?                   ║
║     │   │   └─ Se quote_check: citação encontrada no snapshot?  ║
║     │   ├─ PASS → prossegue para renderização                  ║
║     │   └─ FAIL → corrige ou reporta. NÃO prossegue.           ║
║     │       ├─ Retenta busca (max 2 retries)                    ║
║     │       ├─ Se persiste: marca registro como "unverified"    ║
║     │       └─ NUNCA emite artefato com evidência inválida      ║
║     └─ Exit code: 0=PASS, 1=FAIL                               ║
║                                                                  ║
║  6. RENDERIZAÇÃO DO ARTEFATO DE AUDITORIA                        ║
║     ├─ Executa vitalia_grounding_validator.py --render           ║
║     │   ├─ Lê evidence.jsonl + template                         ║
║     │   └─ Gera GROUNDING_AUDIT.md com:                         ║
║     │       ├─ Badges shields.io (PASS/FAIL, domínio, URLs)    ║
║     │       ├─ Tabela de Proveniência com links clicáveis       ║
║     │       │   ├─ Link direto para a URL auditada             ║
║     │       │   ├─ Link file:// para o snapshot local          ║
║     │       │   └─ SHA-256, data de coleta, domínio, status    ║
║     │       ├─ Seção "Domínios Não Catalogados" (general_web)  ║
║     │       │   └─ ⚠️ Domínio não catalogado + sugestão       ║
║     │       └─ Seção "Verificado vs. Não Verificado"           ║
║     └─ Artefato persistido em:                                   ║
║        .vitalia/memory/data_storage/research/<tema>/             ║
║        GROUNDING_AUDIT.md                                        ║
║                                                                  ║
║  7. HANDOFF DE RETORNO                                           ║
║     ├─ Se invocado como sub-agente:                              ║
║     │   └─ vitalia_handoff.py create --direction reverse        ║
║     │      (retorna ao workflow invocador com relatório)         ║
║     └─ Se invocado como slash command:                           ║
║         └─ Apresenta GROUNDING_AUDIT.md ao usuário              ║
║                                                                  ║
║  8. EXIBIÇÃO (sempre)                                            ║
║     └─ Researcher exibe o .md de auditoria com links clicáveis  ║
║        (reaproveitável no dashboard de dados na nuvem)           ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

### 2.4 Mecanismo de Validação de Integridade (Detalhado)

| Aspecto | Mecanismo | Critério de Sucesso |
|---|---|---|
| **HTTP Status** | Registro `http_status` no evidence.jsonl | `== 200` obrigatório |
| **SHA-256** | Hash do conteúdo completo do fetch | Não vazio e consistente com snapshot |
| **Snapshot** | Arquivo `.txt` em `snapshots/<sha256_prefix>.txt` | Arquivo existe no filesystem |
| **Quote Check** | Registros `type: "quote_check"` verificam citações literais | `found: true` para cada citação |
| **Temporal** | Campo `fetched_at` comparado com `default_validity_days` | Dentro de 90 dias |
| **Redirect** | Campos `url_requested` vs `final_url` | Ambos registrados para auditoria |

### 2.5 Fluxo de Evidência Inválida — Opções com Trade-offs

| # | Opção | Descrição | Prós | Contras | Recomendação |
|---|---|---|---|---|---|
| A | **Retry + Degrade** | Retenta busca (max 2). Se persiste, marca como `"status": "unverified"` e inclui no GROUNDING_AUDIT.md com badge ⚠️ UNVERIFIED. O invocador recebe o relatório com a marcação. | Resiliência a falhas transitórias. Não bloqueia pipeline. Transparência total. | URL inacessível pode retornar conteúdo incorreto. Invocador precisa interpretar badges. | ✅ **Recomendada** |
| B | **Hard Block** | Qualquer evidência inválida bloqueia completamente o researcher. Retorna FAIL ao invocador sem relatório. | Máxima segurança. Zero risco de informação não verificada. | Bloqueia pipeline por falhas transitórias (CDN, timeout). Frustração do usuário. | Para domínios `trigger: blocking` |
| C | **HITL Escalation** | Evidência inválida pausa o researcher e pergunta ao usuário: "URL X retornou 404. Aceitar sem verificação?" | Decisão humana. Flexibilidade. | Interrompe fluxo. Latência. | Para domínios `clinical_safety` e `regulations_compliance` |

> [!IMPORTANT]
> **Decisão D18:** Opção A (Retry + Degrade) como default. Opção C para domínios `clinical_safety` e `regulations_compliance` onde `trigger: blocking` + `constitution_principles` existem. Opção B disponível como flag `--strict` para auditoria formal.

### 2.6 Renderização — SRP e Padrão Arquitetural

| Componente | Responsabilidade | Arquivo |
|---|---|---|
| `vitalia_grounding_validator.py` | Validação pura (PASS/FAIL) com 2 modos: `--mode evidence` e `--mode artifact` | `source/src/vitalia_grounding_validator.py` |
| `grounding_audit_renderer.py` | Renderização do GROUNDING_AUDIT.md via template | `source/src/rendering/grounding_audit_renderer.py` (**novo**) |
| `grounding_audit_template.md` | Template visual do artefato | `source/templates/grounding_audit_template.md` (**novo**) |

> Seguindo o padrão existente: `dashboard_renderer.py` → `dashboard_template.md`, `grounding_renderer.py` (view), `dashboard_renderer.py` (views especializadas).

---

## 3. Estratégia de Defesa em Profundidade (5 Camadas)

### Camada 1 — Remoção de `search_web` de `tools.required` e `tools.optional` (Redução de Saliência)
- **Escopo:** Todos os `33` workflows fora de `research.toml` (`15` centrais, `10` especialistas e `8` extensões de suporte).
- **Mecanismo:** Manter `search_web` estritamente ausente de `tools.required` e `tools.optional`, habilitando `define_subagent` e `invoke_subagent`.

### Camada 2 — Diretriz Afirmativa Exclusiva P03 (ADR-0003)
- **Escopo:** Prompts de todos os `33` workflows sem `search_web`.
- **Mecanismo:** Cláusula afirmativa explícita no topo da instrução e repetida na Finalização:
  > *"Toda eventual consulta externa durante este workflow opera exclusivamente via sub-agente `vitalia-researcher` (`define_subagent` com `EnableWriteTools: true`), aguardando `GROUNDING_AUDIT.md`, status HTTP 200 e handoff reverso antes da síntese (override apenas mediante comando explícito e literal do usuário)."*

### Camada 3 — Checklist Pré-Síntese (Barreira Sequencial)
- **Escopo:** Pré-requisitos operacionais antes de redigir conclusões ou artefatos em qualquer workflow.
- **Mecanismo:** Itens verificáveis explícitos:
  ```
  - [ ] Sub-agente vitalia-researcher invocado e retornou com GROUNDING_AUDIT.md validado (HTTP 200)
  - [ ] Síntese e artefatos emitidos exclusivamente após recebimento do handoff reverso
  ```

### Camada 4 — Portão Determinístico Interno ao Researcher (Validador Global)
- **Escopo:** Interno ao `vitalia-researcher` — evidência inválida é interceptada antes de sair do pesquisador.
- **Mecanismo:** `{{KIT_PYTHON}} {{VITALIA_KIT_DIR}}/src/vitalia_grounding_validator.py --mode evidence` como portão *fail-closed* antes da renderização de `GROUNDING_AUDIT.md`.
- **SRP:** O workflow invocador delega e consome apenas a evidência auditada.
- **Fundamentação:** Padrão **Fail-Closed Evidence Gate**: interceptar payloads inválidos no gateway antes de cruzar o limite de mudança de estado.

### Camada 5 — Segregação por Impacto (Regra de Dois Mundos)
- **Todos os `33` workflows operacionais (além de `research.toml`):** Delegação obrigatória ao `vitalia-researcher` (`search_web` ausente).
- **`research.toml` e chat livre sem workflow ativo:** `search_web` direto habilitado (em `research.toml`, sob protocolo integral de 8 passos).
- **Override HITL:** Apenas um comando explícito e literal do usuário abre exceção dentro de um workflow SDD.

---

## 4. Auditoria de TOML — Mapa Completo de Segregação de `search_web`

### 4.1 Regra de Classificação Consolidada

| Categoria | Cobertura (`34` `.toml`) | `search_web` | Acesso Externo |
|---|---|---|---|
| **33 Workflows (Centrais, Especialistas & Extensões)** | `15` centrais (exceto `research`), `10` especialistas, `8` extensões | ❌ Ausente | Via `vitalia-researcher` (`define_subagent`/`invoke_subagent`) |
| **Researcher** | `research.toml` (`#03`) | ✅ Exclusivo | Execução direta controlada pelos 8 passos + `--mode evidence` |

> [!IMPORTANT]
> **`spec-implement` e todos os especialistas operam sem `search_web`:** quando houver necessidade de verificação técnica externa, delegam ao especialista adequado ou ao `vitalia-researcher`, preservando trilha HTTP 200 + SHA-256 + `GROUNDING_AUDIT.md`.

### 4.2 Mapa Completo Consolidado (`34` TOMLs: `33` sem `search_web` + `research.toml`)

Em alinhamento integral com [CONTRATOS_WORKFLOWS_SDD.md](file:///home/andre/projetos/assistidos/vitalia-kit/docs/CONTRATOS_WORKFLOWS_SDD.md) (`Parte VIII`) e [specs/005-portao-grounding-p03/plan.md](file:///home/andre/projetos/assistidos/vitalia-kit/specs/005-portao-grounding-p03/plan.md), a **Defesa P03 Universal em 5 Camadas** aplica-se a **todos os `33` workflows além de `research.toml`**, sem distinção por rótulo de "verificação" ou "sessão":

| Categoria | Workflows Cobertos (`34` `.toml` em `source/layers/extensions/` $\rightarrow$ `source/extensions/`) | `search_web` (`tools.optional`/`required`) | Delegação P03 (`define_subagent`/`invoke_subagent`) |
|---|---|---|---|
| **15 Workflows Centrais (exceto `research`)** | `session-start`, `brainstorming`, `route`, `architect`, `spec-specify`, `clarify`, `spec-plan`, `spec-tasks`, `task-verifier`, `analyze`, `spec-implement`, `converge`, `judge`, `session-end`, `session-consolidate` | ❌ Ausente | ✅ Habilitada (`vitalia-researcher`, salvo Override HITL literal; `spec-implement` também pode acionar especialistas) |
| **10 Agentes Especialistas** | `caio-strategist`, `medical-gate`, `literature-curator`, `design-thinking`, `visual-creator`, `figma-maestro`, `django-stark`, `react-vinci`, `infra-nomad`, `cortex-analyst` | ❌ Ausente | ✅ Habilitada (`vitalia-researcher` + portão D18) |
| **8 Extensões de Suporte** | `adr`, `pair`, `debug`, `review`, `release`, `science-review`, `integrative-review`, `skill-evaluation` | ❌ Ausente | ✅ Habilitada (`vitalia-researcher` + portão D18) |
| **Researcher (Portador Exclusivo)** | `research.toml` | ✅ Mantido (único workflow) | ✅ Sub-agente autocontido (`vitalia_grounding_validator.py --mode evidence` + `GROUNDING_AUDIT.md`) |

*(Nota histórica: durante o brainstorming inicial de `17-11-2026`, contavam-se `12` TOMLs pendentes de edição manual em `source/` sobre um total preliminar de `28` arquivos; a consolidação em Entrega 5 estendeu a barreira determinística para todos os `33` workflows de produção.)*

### 4.3 Plano de Execução para Sincronização dos TOML em `source/`

**Passo 1:** Completar os `34` `.toml` em `source/layers/extensions/` garantindo ausência de `search_web` nos `33` workflows, presença de `define_subagent`/`invoke_subagent`, cláusula afirmativa P03 (ADR-0003) e checklist pré-síntese (`research.toml` único portador de `search_web`).

**Passo 2:** Compilar deterministicamente de `source/layers/` para `source/extensions/` via `vitalia_layer_compiler.py` (sem edição manual divergente em `source/extensions/`).

**Passo 3:** Validar ausência de deriva com `{{KIT_PYTHON}} source/src/vitalia_layer_compiler.py --layers-dir source/layers --out-dir source --check`.

**Passo 4:** Instalar globalmente em máquina limpa via `source/kit-bootstrap.sh` (`~/.vitalia-kit/`), criando `.venv` + `requirements.txt` antes do gate `--check` e antes da cópia do runtime.

---

## 5. Modo `general_web` — Fallback Dinâmico para Temas Não Catalogados

### 5.1 Conceito (Validado)
O researcher recebe dinamicamente as regras do domínio em `grounding_domains.yaml`. Quando o tema não casa com nenhum domínio catalogado, entra em modo `general_web`:
1. Executa a busca normalmente (HTTP 200 + snapshot + evidence.jsonl).
2. Registra no evidence.jsonl: `"domain_match": "general_web"`.
3. No `GROUNDING_AUDIT.md`: emite `⚠️ Domínio não catalogado` com sugestão de inclusão.
4. **NÃO faz HITL dentro do researcher.** Apenas registra a notificação.

### 5.2 Fluxo Unidirecional de Atualização do Motor na Máquina (`1A`) — Session-End/Consolidate

```
╔══════════════════════════════════════════════════════════════════════════╗
║      FLUXO UNIDIRECIONAL DE DETECÇÃO E CURADORIA DE DOMÍNIOS (1A)        ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                          ║
║  RESEARCHER (durante a sessão):                                          ║
║    └─ Registra domain_match: "general_web" em research/<tema>/evidence   ║
║    └─ Registra no GROUNDING_AUDIT.md: ⚠️ Domínio não catalogado          ║
║    └─ NÃO toca em grounding_domains.yaml nem em session/                 ║
║                                                                          ║
║  SESSION-END (--action end):                                             ║
║    └─ NÃO grava grounding-domains.jsonl                                  ║
║    └─ Emite contagem informativa de candidatos general_web detectados    ║
║       em .vitalia/memory/data_storage/research/*/evidence.jsonl          ║
║                                                                          ║
║  SESSION-CONSOLIDATE (Passo 3–4 — Motor na Máquina -> Nuvem):            ║
║    └─ --action list-pending-domains --json varre research/*/evidence     ║
║    └─ Curadoria HITL (ask_question): scope "global" | "local" | "rejected"║
║       com name, trigger e >= 1 fonte tipada em                           ║
║       .vitalia/grounding_decisions_payload.json                          ║
║    └─ --action consolidate --update-grounding-domains:                   ║
║       1. GroundingDomainUpdater valida via jsonschema.validate() (exit 1)║
║       2. Atualiza EXCLUSIVAMENTE o Motor na máquina (grounding_domains)  ║
║       3. GroundingRenderer projeta os 4 escopos (global, domain_overlay  ║
║          com gate, local com project, rejected) para                     ║
║          session/grounding-domains.jsonl + data_storage/grounding-domains.jsonl + GROUNDING_DOMAINS.md ║
║       4. Aposenta definitivamente grounding-domains-local.yaml           ║
║                                                                          ║
╚══════════════════════════════════════════════════════════════════════════╝
```

### 5.3 Entrada `general_web` no Catálogo

```yaml
- id: general_web
  name: "Busca Web Genérica (Fallback)"
  description: "Domínio catch-all para temas não cobertos pelos domínios catalogados. Detectado no session-end, curado via HITL no session-consolidate."
  scope: global
  trigger: info
  hitl_on_use: false
  authoritative_sources: []
```

---

## 6. Novos Domínios para Inclusão no Catálogo (Validado)

Inclusão aprovada em `source/profiles/grounding_domains.yaml`:

| # | ID | Nome | Escopo / Gate | Trigger | Fontes Autoritativas |
|---|---|---|---|---|---|
| 1 | `web_frameworks` | Frameworks Web & Full-Stack | `domain_overlay` / `web` | `blocking` | docs.djangoproject.com, fastapi.tiangolo.com, react.dev, vuejs.org |
| 2 | `devops_ci` | DevOps, CI/CD & Containers | `domain_overlay` / `devops` | `blocking` | docs.docker.com, docs.github.com/actions, terraform.io/docs |
| 3 | `cloud_services` | Cloud & Managed Services | `domain_overlay` / `cloud` | `blocking` | docs.aws.amazon.com, cloud.google.com/docs, learn.microsoft.com/azure |
| 4 | `design_patterns` | Padrões Arquiteturais & Design | `global` | `warning` | martinfowler.com, refactoring.guru, ddd-community.org |
| 5 | `javascript_ecosystem` | Ecossistema JavaScript/TypeScript | `domain_overlay` / `javascript` | `blocking` | nodejs.org/docs, deno.com/runtime/manual, npmjs.com |
| 6 | `general_web` | Busca Web Genérica (Fallback) | `global` | `info` | (nenhuma — catch-all com detecção em session-end) |

> **Nota:** O domínio `provedores_de_ia` existente será expandido para absorver IA Generativa & ML Ops (HuggingFace, LangChain, Anthropic docs).

---

## 7. `vitalia_grounding_validator.py` — Refatoração para Validador Global

### 7.1 Engenharia Reversa Funcional em 3 Camadas

| Camada | Conteúdo |
|---|---|
| **Intenção de Negócio Pura (a preservar)** | Validar URLs contra evidence.jsonl com HTTP 200 + SHA-256 + snapshot. Retornar PASS/FAIL com exit code. Interface CLI com `--target`, `--evidence-dir`, `--lock-file`, `--json`. |
| **Legado a Purgar** | Foco exclusivo no ciclo SDD (apenas `--target` para .md). Ausência de modo evidence. Output textual sem links clicáveis. Sem integração com grounding_domains.yaml. Sem renderização de artefato visual. |
| **Contrato do Consumidor (novo)** | 2 modos de validação: `--mode evidence`, `--mode artifact`. Renderização separada em `grounding_audit_renderer.py` (SRP). Integração com `grounding_domains.yaml` para classificação. Exit code 0/1 para portão determinístico. |

### 7.2 Interface CLI Refatorada

```
# Modo 1: Validação direta de evidence.jsonl (portão interno do researcher)
python3 vitalia_grounding_validator.py --mode evidence \
  --evidence-dir .vitalia/memory/data_storage/research/<tema>/ \
  --json

# Modo 2: Validação de artefato .md contra evidências (portão de saída SDD)
python3 vitalia_grounding_validator.py --mode artifact \
  --target specs/NNN-slug/research.md \
  --evidence-dir .vitalia/memory/data_storage/research/ \
  --json
```

### 7.3 `grounding_audit_renderer.py` — Novo Renderizador (SRP)

```
# Renderização do GROUNDING_AUDIT.md
python3 grounding_audit_renderer.py \
  --evidence-dir .vitalia/memory/data_storage/research/<tema>/ \
  --output .vitalia/memory/data_storage/research/<tema>/GROUNDING_AUDIT.md \
  --domains-yaml source/profiles/grounding_domains.yaml
```

**Padrão arquitetural:** Idêntico ao `DashboardRenderer`:
- Classe `GroundingAuditRenderer` em `rendering/grounding_audit_renderer.py`
- Template em `templates/grounding_audit_template.md`
- Fallback inline se template ausente
- Dados extraídos exclusivamente de `evidence.jsonl`

### 7.4 Classificação P05

| Componente | Modalidade | Cenário de Falsificação |
|---|---|---|
| `--mode evidence` | `[TEST:NEW]` | O teste grava um `evidence.jsonl` com entradas HTTP 404 e sem snapshot; o validador DEVE retornar FAIL com exit 1. |
| `--mode artifact` | `[TEST:REFACTOR]` | Behavioral Characterization do modo atual — exercitar entradas/saídas antes e depois da renomeação. Zero perda funcional. |
| `grounding_audit_renderer.py` | `[TEST:NEW]` | O teste grava `evidence.jsonl` com 3 entradas válidas e 1 inválida; o renderer DEVE gerar GROUNDING_AUDIT.md com badges, tabela completa, violação marcada e link file:// para cada snapshot. |
| Template `grounding_audit_template.md` | `[TEST:NEW]` | Ausência do template DEVE causar fallback para template inline (como faz `dashboard_renderer.py`). |
| Cache de evidências (90 dias) | `[TEST:NEW]` | Evidência com 91 dias DEVE ser rejeitada como expirada; evidência com 89 dias DEVE ser aceita. |
| Integração grounding_domains.yaml | `[TEST:NEW]` | URL de domínio catalogado (`python_packages`) DEVE ser classificada corretamente; URL de domínio desconhecido DEVE ser marcada como `general_web`. |
| Edição dos `33` TOML (remoção `search_web`) | `[TEST:MIGRATION]` | Contract Preservation: compilação sem deriva (`--check PASS`). Clean Purge: `search_web` ausente de `tools.required`/`optional` nos `33` workflows (presente apenas em `research.toml`). |
| Cláusula P03 no prompt | `[TEST:MIGRATION]` | O prompt dos `33` workflows contém a Diretriz Exclusiva de Grounding P03 (ADR-0003) e o checklist pré-síntese. |
| Novos domínios no catálogo | `[TEST:NEW]` | `grounding_domains.yaml` DEVE conter os 6 novos domínios com fontes tipadas e gates corretos. |

---

## 8. Data Storage Dashboard — Navegação `.md` em 2 Níveis (`2A`)

### 8.1 Conceito
`DataStorageDashboardRenderer` lê diretamente os ledgers `.jsonl` (`<categoria>/index.jsonl`) e gera navegação `.md` em **2 níveis** sobre `.vitalia/memory/data_storage/` (mantendo `.vitalia/memory/session/README.md` isolado):
- **Nível 0 (`.vitalia/memory/data_storage/README.md`):** painel resumido por categoria (`context`, `handoffs`, `research`, `diagnostics`, `telemetry`) com contagens reais e links clicáveis para cada `<categoria>/README.md`.
- **Nível 1 (`.vitalia/memory/data_storage/<categoria>/README.md`):** tabela agrupada por tipo, data civil (`DD-MM-YYYY GMT-04:00`) e status com badges `shields.io` e links clicáveis para as visões **Nível 2** (`<item>.md` ou `<tema>/GROUNDING_AUDIT.md`).

### 8.2 Padrão Arquitetural

| Componente | Responsabilidade | Arquivo |
|---|---|---|
| `core/data_storage_io.py` | Escrita atômica, Zero Data Leak, manutenção de `<cat>/index.jsonl`, `resync_category_index()` (`(category_dir / entry["json_path"]).exists()`) e visões Nível 2 `.md` | `source/src/core/data_storage_io.py` |
| `DataStorageDashboardRenderer` | Lê exclusivamente `<cat>/index.jsonl` e renderiza Nível 0 e Nível 1 | `source/src/rendering/data_storage_dashboard_renderer.py` |
| `dashboard_template.md` | Template visual | `source/templates/dashboard_template.md` |

### 8.3 Estrutura dos Dashboards Nível 0 $\rightarrow$ Nível 1 $\rightarrow$ Nível 2

- **Nível 0 (`data_storage/README.md`):** badges globais + tabela por categoria com link para `<categoria>/README.md`.
- **Nível 1 (`<categoria>/README.md`):** tabela gerada a partir de `<categoria>/index.jsonl` apontando para visões **Nível 2** (`<item>.md` ou `<tema>/GROUNDING_AUDIT.md`).

### 8.4 Independência dos Dashboards & Projeção `1A`

| Dashboard / Visão | Localização | Produtor / Flag | Conteúdo |
|---|---|---|---|
| **Contexto de Sessão** | `.vitalia/memory/session/README.md` | `DashboardRenderer` (`--render-session-dashboard`) | Lê `session/*.jsonl` e `session_ledger.json`; exibe shards, histórico, decisões e domínios globais/overlays/locais com link para `GROUNDING_DOMAINS.md` |
| **Grounding Domains (Nuvem)** | `.vitalia/memory/session/grounding-domains.jsonl` + `.vitalia/memory/data_storage/grounding-domains.jsonl` + `session/GROUNDING_DOMAINS.md` | `GroundingRenderer` (`--render-grounding`) | Espelho validado por `grounding_decision.schema.json` dos `4` escopos (`global`, `domain_overlay`+`gate`, `local`+`project`, `rejected`); aposenta `grounding-domains-local.yaml` |
| **Repositório de Dados** | `.vitalia/memory/data_storage/README.md` + `<cat>/README.md` | `DataStorageDashboardRenderer` (`--render-data-dashboard`) | Navegação em 2 níveis lendo `<cat>/index.jsonl` com links para visões Nível 2 `.md` |

> [!IMPORTANT]
> **Os dashboards são independentes e acionáveis com flags separadas.** `--action consolidate` executa o pipeline completo por padrão (`GroundingDomainUpdater` $\rightarrow$ `GroundingRenderer` $\rightarrow$ `DashboardRenderer` $\rightarrow$ `DataStorageDashboardRenderer`), mas cada etapa pode ser gerada isoladamente.

---

## 9. Refatoração do Session-End/Consolidate — Módulos Determinísticos

### 9.1 Situação Anterior (Legado)
`session-end` gravava `grounding-domains.jsonl` e manipulava arquivos de `.vitalia/memory/session/` sem schema limpo, enquanto o consolidado escrevia `grounding-domains-local.yaml` desconectado de `grounding_domains.yaml`.

### 9.2 Situação Consolidada (`1A` + `2A`)
**Módulos determinísticos** separam estritamente a atualização do **Motor de Domínios na máquina** (`GroundingDomainUpdater`) da **projeção em nuvem** (`GroundingRenderer`), e todos os painéis `.md` são gerados diretamente a partir dos `.jsonl`.

### 9.3 CLI & Flags do `vitalia_context_engine.py` (via `{{KIT_PYTHON}}`)

```bash
# Encerramento de sessão (NÃO grava grounding-domains.jsonl; apenas contabiliza pendências)
{{KIT_PYTHON}} {{VITALIA_KIT_DIR}}/src/vitalia_context_engine.py --action end --payload-file .vitalia/end_payload.json

# Descoberta de domínios general_web em data_storage/research/*/evidence.jsonl
{{KIT_PYTHON}} {{VITALIA_KIT_DIR}}/src/vitalia_context_engine.py --action list-pending-domains --json

# Consolidação completa com atualização do Motor na máquina (HITL payload)
{{KIT_PYTHON}} {{VITALIA_KIT_DIR}}/src/vitalia_context_engine.py --action consolidate \
  --update-grounding-domains --grounding-decisions-file .vitalia/grounding_decisions_payload.json

# Flags seletivas independentes no --action consolidate:
{{KIT_PYTHON}} {{VITALIA_KIT_DIR}}/src/vitalia_context_engine.py --action consolidate \
  --render-session-dashboard    # session/README.md + views (.jsonl)
  --render-data-dashboard       # data_storage/ Nível 0 + Nível 1 (<cat>/index.jsonl)
  --render-grounding            # session/grounding-domains.jsonl + data_storage/grounding-domains.jsonl + GROUNDING_DOMAINS.md
  --update-grounding-domains    # Aplica decisões HITL ao Motor na máquina (global/local/rejected)
```

### 9.4 Módulos Determinísticos Consolidados

| Módulo | Responsabilidade | Integração |
|---|---|---|
| `GroundingDomainUpdater` | Inspeciona `research/*/evidence.jsonl` (`list-pending-domains`), valida payload HITL via `jsonschema.validate()` (*fail-closed* exit `1`) e atualiza o Motor na máquina (`global`/`local`/`rejected`) | `core/grounding_domain_updater.py` |
| `GroundingRenderer` | Lê Motor atualizado + decisões e projeta `session/grounding-domains.jsonl` + `data_storage/grounding-domains.jsonl` (`4` escopos) + `session/GROUNDING_DOMAINS.md` (Globais, Overlays+`gate`, Locais) | `rendering/grounding_renderer.py` |
| `DashboardRenderer` | Lê `session/*.jsonl` + `session_ledger.json` e gera `session/README.md` unificado | `rendering/dashboard_renderer.py` |
| `DataStorageDashboardRenderer` | Lê `<cat>/index.jsonl` e gera Nível 0 `data_storage/README.md` e Nível 1 `<cat>/README.md` | `rendering/data_storage_dashboard_renderer.py` |
| `GroundingAuditRenderer` | Gera `GROUNDING_AUDIT.md` (Nível 2) por tema a partir de `evidence.jsonl` | `rendering/grounding_audit_renderer.py` |

---

## 10. Pesquisa Web — Achados do Estado da Arte

### 10.1 Padrão PROV-AGENT (W3C PROV-O Estendido para Agentes)
- **Fonte:** semanticscholar.org — framework PROV-AGENT que estende W3C PROV para agentic workflows
- **Relevância:** O Vitalia Kit já usa `evidence.jsonl` como um ledger de proveniência. O padrão PROV-O formaliza as relações:
  - `prov:Entity` → evidence.jsonl, snapshots, GROUNDING_AUDIT.md
  - `prov:Activity` → busca do researcher, validação, renderização
  - `prov:Agent` → vitalia-researcher, workflow invocador, usuário
  - `prov:wasGeneratedBy`, `prov:used`, `prov:wasDerivedFrom` → rastreabilidade
- **Ação:** Mapear campos do evidence.jsonl para vocabulário PROV-O como metadados enriquecidos (futuro)

### 10.2 Fail-Closed Evidence Gate Pattern
- **Fonte:** arxiv.org, github.com — padrão arquitetural para agentes com tool-use
- **Princípio:** "O LLM é um motor de raciocínio não-confiável. Propõe ações, mas um motor de políticas determinístico intercepta e aprova ou nega."
- **Implementação no Vitalia Kit:** `vitalia_grounding_validator.py --mode evidence` como portão pré-renderização
- **Achado crítico:** "Nunca avalie políticas *após* uma ação ter ocorrido. Intercepte payloads no gateway da ferramenta *antes* da execução atingir o boundary de mudança de estado."

### 10.3 Mandatory Delegation Sub-Agent Pattern
- **Fonte:** github.com, dipankar.name — padrão de delegação obrigatória
- **Princípio:** "O orquestrador tem acesso amplo de leitura mas ZERO acesso direto a ferramentas de execução de alto privilégio. Para executar operação sensível, DEVE invocar sub-agente com escopo restrito."
- **Implementação no Vitalia Kit:** Workflows de concepção (brainstorming, architect, etc.) não acessam `search_web` diretamente; DEVEM delegar ao `vitalia-researcher`.
- **Achado:** "Permissões de sub-agente DEVEM apenas estreitar, nunca ampliar, em relação ao pai."

### 10.4 Cascading Hallucinations em Multi-Agent
- **Fonte:** medium.com, arxiv.org — problema de propagação de erros
- **Relevância:** Sem o portão P03, uma ambiguidade ou erro do researcher seria aceita como ground-truth pelo workflow invocador.
- **Mitigação:** evidence.jsonl + snapshot + SHA-256 + quote_check formam uma cadeia de proveniência verificável.

### 10.5 Hash Chain para Evidence (Tamper-Evidence)
- **Fonte:** quox.ai, github.com — cadeia criptográfica para logs de auditoria
- **Conceito:** Cada registro JSONL inclui hash SHA-256 do seu conteúdo + hash do registro anterior (`prev_hash`). Alteração de qualquer linha quebra a cadeia.
- **Ação futura:** Considerar adicionar `prev_hash` ao schema de evidence.jsonl para auditoria forensic-grade (não é MVP, mas é estado da arte).

### 10.6 Padrão de Dashboard para Audit Logs JSONL
- **Fonte:** github.io, praesidia.ai — arquitetura de dashboards de auditoria
- **Padrão:** Timeline de execução + árvore causal de evidências + drill-down de compliance
- **Implementação no Vitalia Kit:** O `DataStorageDashboardRenderer` segue este padrão com `<details>` expansíveis e links clicáveis para navegação contextual.

---

## 11. Resumo de Decisões Validadas

| # | Decisão | Status |
|---|---|---|
| D1 | Busca externa é obrigatória (não condicional) em workflows de concepção | ✅ Validada |
| D2 | Researcher é autocontido (SRP): validação, auditoria, renderização e handoff internos | ✅ Validada |
| D3 | Evidência inválida não sai do researcher (fail-closed interno) | ✅ Validada |
| D4 | Researcher invocável via slash command para buscas independentes | ✅ Validada |
| D5 | vitalia_grounding_validator.py refatorado: `--mode evidence` + `--mode artifact` (validação pura) | ✅ Validada |
| D6 | Renderização separada em `grounding_audit_renderer.py` (SRP) | ✅ Validada |
| D7 | GROUNDING_AUDIT.md persistido na Camada 3 e sincronizável com repositório Git na nuvem | ✅ Validada |
| D8 | Links file:// para snapshots locais no artefato | ✅ Validada |
| D9 | Remoção de `search_web` de **todos os 33 workflows** fora de `research.toml` (15 centrais, 10 especialistas, 8 extensões) em `source/` | ✅ Validada (Rev. 4) |
| D10 | `research.toml` mantém `search_web` (portador exclusivo) | ✅ Validada |
| D11 | TODOS os especialistas operam via delegação ao `vitalia-researcher` | ✅ Validada |
| D12 | `spec-implement` NÃO usa `search_web`; consulta especialistas ou `vitalia-researcher` conforme contexto | ✅ Validada |
| D13 | 6 novos domínios + expansão de `provedores_de_ia` incluídos em `source/layers/grounding/` $\rightarrow$ `grounding_domains.yaml` | ✅ Validada |
| D14 | Cache de evidências aceito dentro de `default_validity_days` (90 dias) | ✅ Validada |
| D15 | Modo `general_web` como fallback — researcher registra; `session-end` apenas contabiliza (zero escrita em `grounding-domains.jsonl`); `session-consolidate` curadoria HITL (`global`/`local`/`rejected`) | ✅ Validada (Rev. 4) |
| D16 | Override HITL explícito do usuário como única exceção em workflow SDD | ✅ Validada |
| D17 | Foco exclusivo de código e testes P05 em `source/` e diretórios temporários (`tmp_path`), instalando para `~/.vitalia-kit/` | ✅ Validada (Rev. 4) |
| D18 | Evidência inválida: Retry+Degrade (default), HITL Escalation (clinical/compliance), Hard Block (`--strict`) | ✅ Validada |
| D19 | Researcher NÃO toca em `grounding_domains.yaml`; `GroundingDomainUpdater` atualiza exclusivamente o Motor na máquina no `session-consolidate` e `GroundingRenderer` projeta `session/grounding-domains.jsonl` + `data_storage/grounding-domains.jsonl` + `GROUNDING_DOMAINS.md` | ✅ Validada (Rev. 4) |
| D20 | Data Storage Dashboard em 2 níveis (Nível 0 $\rightarrow$ Nível 1 lendo `<cat>/index.jsonl` $\rightarrow$ Nível 2 `.md`) independente do Context Dashboard | ✅ Validada (Rev. 4) |
| D21 | `session-end`/`session-consolidate` determinísticos substituem manipulação via LLM | ✅ Validada |
| D22 | Renderizadores separados seguindo SRP (`grounding_audit_renderer.py`, `data_storage_dashboard_renderer.py`, `grounding_renderer.py`) | ✅ Validada (Rev. 4) |

---

## 12. Questões Resolvidas (da Rev. 2 $\rightarrow$ Rev. 4)

| # | Questão | Resolução Consolidada (`005-portao-grounding-p03`) |
|---|---|---|
| Q1 | Escopo e fluxo do `general_web` (`1A`) | Researcher apenas registra em `evidence.jsonl`; `--action end` não escreve `grounding-domains.jsonl`; `GroundingDomainUpdater` aplica HITL (`global`/`local`/`rejected`) no Motor da máquina em `--action consolidate`; `GroundingRenderer` projeta os `4` escopos (`global`, `domain_overlay`+`gate`, `local`+`project`, `rejected`) em `GROUNDING_DOMAINS.md` (aposentando `grounding-domains-local.yaml`). |
| Q2 | Localização do `GROUNDING_AUDIT.md` e Níveis `2A` | `.vitalia/memory/data_storage/research/<tema>/GROUNDING_AUDIT.md` (Nível 2), indexado via `<cat>/index.jsonl` no Nível 0 (`data_storage/README.md`) e Nível 1 (`<cat>/README.md`). |
| Q3 | Handoff de retorno do researcher | `vitalia_handoff.py create --direction reverse` (zero `"$schema"`, `ttl_seconds = 86400`, transição atômica `expired`). |
| Q4 | Templates e badges | `source/templates/{grounding_audit_template,data_storage_dashboard_template,dashboard_template}.md` com fallback inline e badges `shields.io`. |
| Q5 | Separação SRP e bootstrap limpo | Renderizadores isolados em `source/src/rendering/`; `source/kit-bootstrap.sh` cria `.venv` + `requirements.txt` antes do `--check` e antes da cópia para `~/.vitalia-kit/`. |

---

## 13. Escopo Consolidado de Implementação (`specs/005-portao-grounding-p03/plan.md`)

1. Pipeline SDD integralmente seguido: `specify → clarify → brainstorming → plan → tasks → task-verifier → implement → converge`.
2. Escopo completo dos 6 Grupos em `source/`:
   - `24` Schemas OpenAPI 3.0.3 Limpos (`source/src/schemas/`)
   - Dual-Runtime estrito `{{KIT_PYTHON}}` + Hooks (`GuardianContextV2`, `scan_environment.py` Spec 007 `< 50ms`) + `3` sensores VRSS
   - `core/data_storage_io.py` (`resync_category_index` + Nível 2 `.md`), `vitalia_handoff.py`, `vitalia_grounding_validator.py --mode evidence|artifact`, Renderizadores SRP (`1A` + `2A`), `vitalia_tmp_cleanup.py` Spec 007 + `--types` e `sdd_ambiguity_scorer.py` completo
   - `34` `.toml` compilados sem deriva (`33` sem `search_web` + `research.toml`), `24` `gp_*.yaml`, `6` domínios novos + `provedores_de_ia` e `5` catálogos YAML SOTA
   - `source/kit-bootstrap.sh` (`.venv` $\rightarrow$ `--check` $\rightarrow$ cópia `~/.vitalia-kit/` $\rightarrow$ shims), `source/bin/install-project` (sem Gemini CLI, `.vitalia/config.yml`, `34` skills via `{{KIT_PYTHON}}`+`tomllib`), `source/bin/vitalia-clean` e suíte P05 completa em `tmp_path`.

---

<sub>Rascunho originalmente gerado em brainstorming ativo (`17-11-2026 12:50 GMT-04:00`) e consolidado na Rev. 4 em `19-11-2026 07:45(GMT-04:00)` em paridade integral com `specs/005-portao-grounding-p03/plan.md`.</sub>
