<!-- handoff-correcoes-vitalia-kit-v0.0.1.md | Vitalia Kit v0.0.1 | 23-11-2026 14:15(GMT-04:00) -->
**handoff-correcoes-vitalia-kit-v0.0.1.md | Vitalia Kit v0.0.1 | 23-11-2026 14:15(GMT-04:00)**

# Documento de Transferência (Handoff) — Spec de Correções do Vitalia Kit v0.0.1

> **Propósito:** Guia técnico completo para conduzir o ciclo SDD (`specify -> plan -> tasks -> implement -> converge`) na máquina de desenvolvimento do **Vitalia Kit v0.0.1** (`git@github.com:andreticba/vitalia-kit.git`), consolidando todos os defeitos mapeados, decisões arquiteturais (`ADR-0014`), regras de runtime e a definição dos novos papéis dos agentes de revisão.
>
> **Evidências e logs verbosos de origem:** [relatorio-completo-testes-e-erros-turno-1.md](file:///home/andre/projetos/assistidos/vitalia-docs/docs/relatorios/relatorio-completo-testes-e-erros-turno-1.md) e [teste-prontidao-e-defeitos-v0.0.1.md](file:///home/andre/projetos/assistidos/vitalia-docs/docs/relatorios/teste-prontidao-e-defeitos-v0.0.1.md)

---

## 1. Regras Arquiteturais Inegociáveis

1. **Topologia Real da Camada 3 (Dois Repositórios Git Independentes):**
   - **Sessão (`session_repo`):** `.vitalia/memory/session/`
   - **Dados (`data_repo`):** `.vitalia/memory/data_storage/`
   - **Proibição:** Nenhum componente do kit pode ler ou gravar em `.vitalia/session/` ou `.vitalia/session/data/`.
2. **Padrão de Runtime Python (`${KIT_PYTHON}`):**
   - `${KIT_PYTHON}` (`<kit_dir>/.venv/bin/python3`) é o interpretador padrão em **todo o kit** (instaladores, libs shell, hooks, sensores, renderizadores e shims).
   - Toda chamada Python nos scripts de instalação (`install-project`, `agent_preserver.sh`, etc.) deve ocorrer **após** a instalação/resolução do `.venv` global e utilizar `${KIT_PYTHON}` (nunca `python3` nu do sistema).
   - O ambiente virtual do projeto alvo (`${PROJECT_PYTHON}`) é usado **exclusivamente** para testes do código da aplicação cliente durante o ciclo SDD.
3. **Defesa P03 Universal (`vitalia-researcher` obrigatório):**
   - Todo acesso externo / busca web em qualquer workflow — incluindo os novos agentes de revisão — **deve** ser delegado exclusivamente ao subagente `vitalia-researcher`.

---

## 2. Escopo Detalhado de Correções

### Bloco 1 — Instalação Global e de Projeto (`bin/`)

| ID | Alvo | Problema Atual | Correção Esperada |
|---|---|---|---|
| **A1** | `kit-bootstrap.sh` & `bin/install-project` | Instalação global não exibe progresso detalhado no terminal nem salva log. `install-project` importa `logger.sh` e `preflight.sh`, mas não os utiliza. | Emitir passos claros no terminal (`log_step`, `log_ok`, `log_warn`, `log_error`) e gravar log persistente (global: `<kit_dir>/logs/install.log`; projeto: `.vitalia/memory/data_storage/telemetry/install.log`). Executar `preflight.sh` (`check_python_version_semver`, `check_venv_capability`, `check_writable_target`, `check_git_requirements`). Preservar `kit-bootstrap.sh` no repositório/distribuição do kit. |
| **A2** | `bin/install-project` | Não solicita interativamente `--session-repo` e `--data-repo` quando omitidos em modo interativo (`NON_INTERACTIVE=0`). | Se `NON_INTERACTIVE=0` e `[ -t 0 ]`, perguntar ao usuário as URLs Git de `SESSION_REPO` e `DATA_REPO` (aceitando Enter para modo apenas local). |
| **A3** | `bin/install-project:93` & `bin/lib/agent_preserver.sh:32,95,107` | `install-project:93` usa `python3` antes de definir `KIT_PYTHON` (L96). `agent_preserver.sh` usa `python3` do sistema em `update_agents_md`, `compute_file_sha256` e `update_manifest`. | Resolver `KIT_DIR` em bash puro (`cd`/`pwd`) antes de definir `KIT_PYTHON`; passar `"${KIT_PYTHON}"` para `agent_preserver.sh` e garantir que todas as chamadas ocorram após a garantia do `.venv`. |
| **A4** | `bin/lib/env_manager.sh` | `update_project_config` grava apenas `domain_gates` e `memory` em `.vitalia/config.yml`. | Gravar também `kit_version`, `kit_dir` e `project` (nome do projeto) mantendo merge não-destrutivo. |
| **A5** | `bin/install-project` | Não inicializa `.vitalia/feature.json`, `.vitalia/extensions.yml`, `session_ledger.json` nem atualiza `.gitignore`. | Criar `feature.json` (`{"feature_directory": ""}`), `extensions.yml` padrão, `.vitalia/memory/session/session_ledger.json` (`{"shards": []}`) e adicionar `.env`, `.vitalia/config.yml`, `.vitalia/memory/`, `.vitalia/backups/` ao `.gitignore` de forma idempotente. |
| **A6** | `bin/lib/git_storage_manager.sh` | Cria commit órfão local (`init: session tracking` / `init: data storage`) e apenas `git remote add origin`. Em testes reais com `git@github.com:andreticba/vitalia-kit-context.git` (`2564d1be`) e `vitalia-kit-data.git` (`b8027a90`), o histórico local divergiu do remoto, o que bloqueará `git push`. Além disso, se `.git` já existir, ignora nova URL informada. | 1) Se `remote_url` for informada e `.git` não existir: verificar `git ls-remote`; se o remoto já tiver branch `main` (ou `HEAD`), clonar/fazer `fetch` + `checkout` antes de criar subdiretórios, preservando histórico remoto.<br>2) Se `.git` já existir e `remote_url` for informada: atualizar `origin` (`git remote set-url origin` ou `add`). |
| **A7** | `source/kit-bootstrap.sh` | Ignora as variáveis `VERBOSE` e `NON_INTERACTIVE`, chama `python3` na L57 antes do `.venv`, não atualiza `~/.bashrc`/`~/.zshrc` com `VITALIA_KIT_DIR` e `PATH`, e imprime `"Vitalia SPEC Kit"`. | Honrar `--verbose`, resolver `KIT_DIR` em bash puro antes do `.venv`, configurar `VITALIA_KIT_DIR`/`PATH` nos arquivos rc (quando solicitado/padrão) e padronizar identidade para `"Vitalia Kit"`. |
| **A8** | Todo `source/` (`138` arquivos) | Referências legadas à marca `"Vitalia SPEC Kit"` em código, cabeçalhos, scripts e templates. | Substituir para `"Vitalia Kit"` e ler versão dinamicamente de `<KIT_DIR>/VERSION`. |

---

### Bloco 2 — Unificação de Caminhos na Camada 3 (Fim do Split-Brain)

Substituir todas as ocorrências residuais de `.vitalia/session/` e `.vitalia/session/data/` pelos caminhos oficiais `.vitalia/memory/session/` e `.vitalia/memory/data_storage/`:

| ID | Arquivo | Linha(s) | Correção |
|---|---|---|---|
| **B1** | `src/vitalia_context_engine.py` | L32, L64 | Trocar `.vitalia/session` por `.vitalia/memory/session` e passar `storage_dir = cwd / ".vitalia" / "memory" / "data_storage"` ao `DashboardRenderer`. |
| **B2** | `src/rendering/dashboard_renderer.py` | L18–20 | Receber `session_dir` (`.vitalia/memory/session`) e `storage_dir` (`.vitalia/memory/data_storage`) como parâmetros independentes no `__init__`. |
| **B3** | `src/vitalia_tmp_cleanup.py` | L110 | Trocar `self.project_root / ".vitalia" / "session" / "data"` por `self.project_root / ".vitalia" / "memory" / "data_storage"`. |
| **B4** | `src/core/grounding_domain_updater.py` | L68 | Trocar `self.project_root / ".vitalia" / "session" / "data"` por `self.project_root / ".vitalia" / "memory" / "data_storage"`. |
| **B5** | `src/rendering/grounding_renderer.py` | L26–27 | Trocar para `.vitalia/memory/data_storage` (`storage_dir`) e `.vitalia/memory/session` (`session_dir`). |
| **B6** | `src/rendering/data_storage_dashboard_renderer.py` | L131 | Alterar `--storage-dir` default para `.vitalia/memory/data_storage`. |
| **B7** | `src/sensors/sensor_memory_ledger.py` | L81 | Corrigir fallback 3-Tier de `memory_dir / "session" / "data"` para `memory_dir / "data_storage"` e aceitar `session_ledger.json` recém-inicializado (`{"shards": []}`). |
| **B8** | `integrations/antigravity/SKILL.md.template` & `src/vitalia_handoff.py` | L18 / L269 | Atualizar instruções e exemplos `--help` para `.vitalia/memory/data_storage/`. |
| **C3** | `src/tools/pre_commit_runner.py` | L197–198 | Substituir `$HOME/.vitalia-spec-kit/.venv/bin/python3` por `$HOME/.vitalia-kit/.venv/bin/python3` (respeitando `${VITALIA_KIT_DIR}`). |
| **D1** | `profiles/constitution.yaml` (P04) & `src/maintenance/header_stamp.py` | L59–64 | A regra `P04` foi resumida demais e perdeu o formato obrigatório de 2 linhas para `.md` (`<!-- ... -->` + `**...**` sem `> `), além de `maintenance/header_stamp.py` não existir no kit atual (existe apenas versão antiga em `kit_src/source/src/maintenance/header_stamp.py`). Portar e atualizar `header_stamp.py` com: (1) remoção de `kit_env_bootstrap.init()`; (2) leitura dinâmica de `<KIT_DIR>/VERSION` no formato `Vitalia Kit v{VERSION}`; (3) 2ª linha `.md` como `**{file} | Vitalia Kit v{VERSION} | {ts}**` (sem `> `); (4) consumo defensivo apenas de carimbos Vitalia; (5) filtro de diretórios ocultos usando caminho relativo ao `project_root`; (6) suporte `--json` (VRSS) e `--cwd`; (7) arquivos `.json`: apenas validação sintática (`json.loads`) sem mutação de conteúdo (compatível com P11 OpenAPI 3.0 limpo). Atualizar também os renderizadores (`dashboard_renderer.py`, `data_storage_dashboard_renderer.py`, `grounding_renderer.py`). |
| **D2** | `profiles/constitution.yaml` (P07, P10, P12) | L86, L98, L112 | Os princípios `P07`, `P10` e `P12` ainda declaram `validator: hooks/llm_judge.py --mode after_task`, porém `hooks/llm_judge.py` foi removido na migração de `kit_src` para o Vitalia Kit v0.0.1. Atualizar o campo `validator:` para `/vitalia-judge` (ou o sensor determinístico correspondente). |
| **C4** | CLIs sem `sys.path` (`grounding_audit_renderer.py`, `vitalia_tmp_cleanup.py`, `data_storage_dashboard_renderer.py`) | Topo dos arquivos | Fazem imports internos (`from core...`) dentro de funções sem adicionar `src/` ao `sys.path`, causando `ModuleNotFoundError` quando executados diretamente via `${KIT_PYTHON}` sem `PYTHONPATH`. |
| **C5** | `extensions/*.toml` (`transport.file_path`) | `[transport]` | `17` `.toml` declaram `.vitalia/pipe.json` e `17` declaram `.vitalia/pipeline.json`. Padronizar um único caminho em `source/layers/extensions/` e recompilar. |
| **D3** | `src/hooks/guardian_context.py` vs `profiles/gp_*.yaml` | L249–250 | `GuardianContextV2` lê `phase_guardrails` e `phase_profiles`, mas os `25` `gp_*.yaml` em camadas usam `guardrails:` / `phase:` / `role:` no topo, resultando em `{}` no envelope JSON. Alinhar o schema dos perfis ou o parser de `GuardianContextV2`. |
| **D4** | `source/layers/extensions/*.toml` vs `CONTRATOS_WORKFLOWS_SDD.md` | `[guardian]` (`prompt`) | Todos os `34` `.toml` têm prompts comprimidos (64–115 chars), omitindo os protocolos multi-etapas, portões HITL e a cláusula P03 `vitalia-researcher` especificados em `CONTRATOS_WORKFLOWS_SDD.md`. |
| **D5** | `specs/005` `T028` & `vitalia_context_engine.py` | `source/src/` | Módulos `core/shard_manager.py`, `core/context_engine.py`, `rendering/view_renderer.py` e `constitution_renderer.py` não existem no Git apesar de `T028` marcado `[X]`; e `vitalia_context_engine.py` não implementa `--action init`, `--payload-file`, `--render-*-dashboard` nem `--render-grounding`. |
| **B9** | `source/tests/` (3 suítes) | `test_sensor_memory_ledger.py`, `test_context_engine_and_grounding_flow_1a_2a.py`, `test_vitalia_tmp_cleanup_spec007.py` | Fixam o caminho legado `.vitalia/session/data` nos testes automatizados; atualizar para `.vitalia/memory/data_storage` e `.vitalia/memory/session`. |

---

### Bloco 3 — Dashboards Independentes para Nuvem (`session_repo` e `data_repo`)

Ambos os repositórios Git da Camada 3 devem gerar seus próprios dashboards prontos para visualização no GitHub, de forma independente:

1. **`data_storage_dashboard_renderer.py` (`data_repo` -> `.vitalia/memory/data_storage/`):**
   - **Corrigir `sys.path`:** adicionar `src/` ao `sys.path` no topo do script para eliminar `ModuleNotFoundError: No module named 'core'` quando executado diretamente via `${KIT_PYTHON}`.
   - **Persistência em disco:** além de imprimir ou retornar string, gravar `README.md` (Nível 0) e visões Nível 1 dentro de `.vitalia/memory/data_storage/`.
   - **Links relativos:** substituir `file:///...` por links relativos Markdown (`./context/`, `./handoffs/`, etc.) para funcionar no GitHub.
   - **Indexação real (`index.jsonl`):** `vitalia_hook_runner.py` (ou o fluxo de consolidação) deve registrar os envelopes criados em `context/index.jsonl` via `register_data_storage_item()` para que os contadores não fiquem zerados.
2. **`dashboard_renderer.py` (`session_repo` -> `.vitalia/memory/session/`):**
   - Gravar `README.md` dentro de `.vitalia/memory/session/` (nunca em `.vitalia/session/`), lendo os contadores reais de `.vitalia/memory/data_storage/` e usando links relativos.

---

## 3. Novos Agentes de Revisão e Atualização dos `.toml`

### 3.1. Novos Papéis: Agentes de Revisão de Achados (Escopo Enxuto)
Em vez de duplicar a coleta de `research.toml` e `literature-curator.toml`, `science-review.toml` e `integrative-review.toml` passam a atuar como **revisores independentes dos achados já coletados**.
- **Sem expansão de escopo:** reutilizam `gp_research` e `scan_environment.py` (sem criar novos `.py` nem novos `gp_*.yaml`).
- **Regra P03 obrigatória:** qualquer acesso externo/checagem adicional deve ser delegado exclusivamente a **`vitalia-researcher`**.

1. **`extensions/science-review.toml` (Revisor Científico & Metodológico — Sem Sobrepor o Validador Determinístico):**
   - *Separação de responsabilidades:* A checagem mecânica (HTTP 200, SHA-256, snapshot, `literal_quote`, janela de 90 dias) é delegada ao script determinístico `vitalia_grounding_validator.py` (`--mode artifact` / `--mode evidence`). O agente LLM foca exclusivamente na **análise crítica científica** (nível de evidência A/B/C, risco de viés, validade metodológica e lacunas).
   ```toml
   [context]
   reads = [".vitalia/memory/data_storage/research/"]
   writes = [".vitalia/memory/data_storage/reviews/"]

   [guardian]
   guardian_profile = "gp_research"
   prompt = "Validar integridade técnica dos achados em .vitalia/memory/data_storage/research/ executando vitalia_grounding_validator.py e conduzir revisão científica/metodológica crítica (nível de evidência, vieses e lacunas), registrando o parecer em .vitalia/memory/data_storage/reviews/. Qualquer acesso externo é obrigatório via vitalia-researcher."
   yaml_sources = ["constitution", "grounding_domains"]
   ```

2. **`extensions/integrative-review.toml` (Síntese e Meta-Análise Cruzada):**
   ```toml
   [context]
   reads = [".vitalia/memory/data_storage/research/", ".vitalia/memory/data_storage/reviews/"]
   writes = [".vitalia/memory/data_storage/research/", ".vitalia/memory/data_storage/reviews/"]

   [guardian]
   guardian_profile = "gp_research"
   prompt = "Sintetizar evidências em .vitalia/memory/data_storage/research/ e pareceres em .vitalia/memory/data_storage/reviews/ em uma revisão integrativa (convergências, divergências e recomendações), validando o artefato final com vitalia_grounding_validator.py. Qualquer acesso externo é obrigatório via vitalia-researcher."
   yaml_sources = ["constitution", "grounding_domains"]
   ```

### 3.2. Demais `.toml` para Atualização de `[context]`
- **`route.toml`:** `reads = [".vitalia/memory/session/", "specs/"]`, `writes = [".vitalia/memory/data_storage/handoffs/"]`
- **`session-consolidate.toml`:** `reads = [".vitalia/memory/data_storage/"]`, `writes = [".vitalia/memory/session/", ".vitalia/memory/data_storage/"]`
- **`session-end.toml`:** `reads = [".vitalia/memory/session/", "specs/"]`, `writes = [".vitalia/end_payload.json", ".vitalia/memory/data_storage/", ".vitalia/memory/session/"]`
- **`session-start.toml`:** `reads = ["specs/", ".vitalia/memory/session/", ".vitalia/config.yml"]`, `writes = [".vitalia/memory/data_storage/"]`

---

## 4. Critérios de Aceitação na Máquina Destino (Checklist de Verificação)

Após implementar a spec de correções, validar todos os itens abaixo:
1. **Instalação global:** gera passos claros no terminal + arquivo de log persistente; instala `.venv` antes de qualquer script consumidor.
2. **Instalação no projeto (`install-project`):**
   - Solicita interativamente `SESSION_REPO` e `DATA_REPO` quando omitidos.
   - Conecta em repositórios GitHub já populados (`git@github.com:andreticba/vitalia-kit-context.git` e `vitalia-kit-data.git`) sem criar histórico órfão conflitante.
   - Grava `.vitalia/config.yml` completo, `feature.json`, `extensions.yml`, `session_ledger.json` e `.gitignore`.
3. **Dashboards independentes:**
   - `vitalia_context_engine.py --action consolidate` grava `README.md` dentro de `.vitalia/memory/session/` com contagens reais e links relativos.
   - `data_storage_dashboard_renderer.py` executa diretamente sem `PYTHONPATH` manual, grava `README.md` dentro de `.vitalia/memory/data_storage/` com contagens reais (`index.jsonl`) e links relativos.
4. **Faxineiro & Sensores:**
   - `vitalia-clean` enxerga e lista arquivos elegíveis em `.vitalia/memory/data_storage/context/`.
   - `sensor_memory_ledger.py --memory-dir .vitalia/memory --json` retorna `PASS` (`exit 0`) em projeto recém-instalado.
   - `pre_commit_runner.py` referencia `$HOME/.vitalia-kit/.venv/bin/python3` (sem referências a `.vitalia-spec-kit`).
5. **Constituição, Context Lifting & `header_stamp.py`:**
   - `guardian_json.phase_guardrails` não retorna `{}` vazio no runtime (`GuardianContextV2` alinhado aos `gp_*.yaml`).
   - `profiles/constitution.yaml` restaurado com `P04` completo e `validator:` válidos (sem `hooks/llm_judge.py` fantasma).
   - `src/maintenance/header_stamp.py` portado de `kit_src`, usando `Vitalia Kit v{VERSION}` de `<KIT_DIR>/VERSION`, 2ª linha `.md` em `**...**` sem `> ` e `.json` em validação sintática apenas (`json.loads`).
   - Todos os CLIs em `src/` executam diretamente via `${KIT_PYTHON}` sem exigir `PYTHONPATH` manual.
6. **Reconciliação Documental Pós-Correção:**
   - `CONTRATOS_WORKFLOWS_SDD.md` em `vitalia-docs` permanece congelado durante a spec de correções e será reconciliado após a validação do Turno 2 sobre o código estabilizado.

