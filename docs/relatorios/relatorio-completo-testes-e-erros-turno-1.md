<!-- relatorio-completo-testes-e-erros-turno-1.md | Vitalia Kit v0.0.1 | 23-11-2026 14:15(GMT-04:00) -->
**relatorio-completo-testes-e-erros-turno-1.md | Vitalia Kit v0.0.1 | 23-11-2026 14:15(GMT-04:00)**

# Relatório Completo de Testes de Prontidão e Mapeamento de Erros — Turno 1 (Vitalia Kit v0.0.1)

> **Escopo:** Consolidação integral dos testes executados e da auditoria cruzada entre o kit instalado (`~/.vitalia-kit/`), o repositório de desenvolvimento (`/home/andre/projetos/assistidos/vitalia-kit`, `git@github.com:andreticba/vitalia-kit.git`), o código legado (`kit_src` e `vitalia-spec-kit`) e a documentação normativa em `vitalia-docs/docs/`.

---

## 1. Sumário Executivo da Prontidão

| Área Auditada | Status | Resumo Objetivo |
|---|---|---|
| **Instalação Global (`source/kit-bootstrap.sh`)** | ❌ Falha Parcial | Executa com `exit 0`, mas gera apenas 2 linhas no `stdout`, zero arquivos `.log`, ignora `--verbose`, não injeta `VITALIA_KIT_DIR`/`PATH` em `~/.bashrc`/`~/.zshrc` e mantém marca `"Vitalia SPEC Kit"`. |
| **Instalação no Projeto (`bin/install-project`)** | ❌ Falhas Críticas | Sem logs passo-a-passo nem arquivo `.log`; sem prompts interativos para `session_repo`/`data_repo`; `git_storage_manager.sh` cria commits órfãos incompatíveis com repositórios GitHub já populados; `.vitalia/config.yml` incompleto; ausentes `feature.json`, `extensions.yml`, `session_ledger.json` e regras no `.gitignore`. |
| **Runtime & Split-Brain Camada 3 (`src/`)** | ❌ Falhas Críticas | Módulos ainda leem/escrevem em `.vitalia/session/` e `.vitalia/session/data/` em vez de `.vitalia/memory/session/` e `.vitalia/memory/data_storage/`. `vitalia-clean` não enxerga os envelopes criados pelos hooks. |
| **Dashboards de Nuvem (`session_repo` e `data_repo`)** | ❌ Falhas Críticas | `dashboard_renderer.py` grava fora do repo Git; `data_storage_dashboard_renderer.py` quebra sem `PYTHONPATH` (`ModuleNotFoundError: No module named 'core'`), não grava arquivo em disco e emite links `file:///...`. |
| **Contratos (`CONTRATOS_WORKFLOWS_SDD.md`) vs Código** | ❌ Divergência Severa | Prompts dos 34 `.toml` comprimidos (64–115 chars); módulos do `T028` (`core/context_engine.py`, `core/shard_manager.py`, `rendering/view_renderer.py`, `constitution_renderer.py`) ausentes; flags do `vitalia_context_engine.py` inexistentes; `guardian_json.phase_guardrails` e `phase_profiles` vazios (`{}`) no runtime. |
| **Constituição (`constitution.yaml`) & P04** | ❌ Falhas Médias/Altas | P04 simplificada perdeu formato de 2 linhas; `maintenance/header_stamp.py` e `hooks/llm_judge.py` declarados como validadores não existem no kit; 3 CLIs falham sem `PYTHONPATH`. |
| **Sensores VRSS, Sintaxe Python e Schemas** | ⚠️ Parcial | 0 erros de sintaxe Python; 23/23 CLIs passam em `--help` (P10); `sensor_tokens_leak`, `sensor_path_hygiene` e `sensor_purity` passam; `sensor_memory_ledger` falha (`session_ledger.json` ausente). |

---

## 2. Bateria de Testes Executados e Evidências Reais

### Teste 1 — Instalação Global em `$HOME` Isolado (`source/kit-bootstrap.sh`)
**Objetivo:** Validar a instalação limpa do Vitalia Kit a partir de `vitalia-kit/source/` em ambiente isolado (`/tmp/vitalia-fakehome`).

**Comando:**
```bash
HOME=/tmp/vitalia-fakehome bash ./kit-bootstrap.sh --kit-dir /tmp/vitalia-fakehome/.vitalia-kit --yes
```

**Saída Observada (`EXIT=0`, duração `7,45s`):**
```text
--- STDOUT (2 linhas) ---
[LayerCompiler] ✅ PASS: 68 arquivos verificados.
✅ Vitalia SPEC Kit instalado com sucesso em: /tmp/vitalia-fakehome/.vitalia-kit
--- STDERR (4 linhas, em branco) ---
```

**Verificações Pós-Instalação:**
- **Arquivos `.log` criados:** nenhum (`find /tmp/vitalia-fakehome -iname '*.log'` retornou vazio).
- **Flags mortas no script (`kit-bootstrap.sh`):** `VERBOSE=0/1` (L18, L45) e `NON_INTERACTIVE=0/1` (L17, L41) são atribuídas no `while`, mas **nunca são consultadas** no restante do script.
- **Shell RC (`~/.bashrc` / `~/.zshrc`):** não foram tocados nem atualizados com `export VITALIA_KIT_DIR` e `PATH`.
- **Chamada `python3` antes do `.venv`:** L57 normaliza `KIT_DIR` usando `python3` do sistema antes de criar `${KIT_DIR}/.venv`.
- **Identidade do produto:** imprime `"Vitalia SPEC Kit"` em vez de `"Vitalia Kit"`.

---

### Teste 2 — Instalação no Projeto com Repositórios GitHub Reais (`bin/install-project`)
**Objetivo:** Validar ativação em projeto alvo (`/tmp/vitalia-test-project`) conectando aos repositórios reais `git@github.com:andreticba/vitalia-kit-context.git` e `git@github.com:andreticba/vitalia-kit-data.git`.

**Comando:**
```bash
/home/andre/.vitalia-kit/bin/install-project \
  --project-dir /tmp/vitalia-test-project \
  --session-repo "git@github.com:andreticba/vitalia-kit-context.git" \
  --data-repo "git@github.com:andreticba/vitalia-kit-data.git" \
  --yes
```

**Saída Observada (`EXIT=0`):**
```text
Skills geradas com sucesso: 34
✅ Vitalia SPEC Kit instalado com sucesso no projeto: /tmp/vitalia-test-project
```

**Artefatos Gerados e Lacunas:**
- `.vitalia/config.yml` gravado sem `kit_version`, `kit_dir` e `project`:
  ```yaml
  domain_gates: []
  memory:
    session_repo: git@github.com:andreticba/vitalia-kit-context.git
    data_repo: git@github.com:andreticba/vitalia-kit-data.git
  ```
- **Ausentes após instalação:** `.vitalia/feature.json`, `.vitalia/extensions.yml`, `.vitalia/memory/session/session_ledger.json`, regras de proteção no `.gitignore` e qualquer arquivo `.log`.
- **Conflito Git com o GitHub (`git ls-remote`):**
  - Local `session`: commit órfão `5784972 (HEAD -> main) init: session tracking` vs Remoto `vitalia-kit-context`: `2564d1be7e8eff3e8b1bb7d640d212d0de6876b0 refs/heads/main` (`EXIT_CONTEXT=0`).
  - Local `data_storage`: commit órfão `936d000 (HEAD -> main) init: data storage` vs Remoto `vitalia-kit-data`: `b8027a90b82b282889bd2b463971f4cce5b44547 refs/heads/main` (`EXIT_DATA=0`).
  - **Impacto:** Qualquer `git push` posterior será rejeitado por históricos não relacionados.

---

### Teste 3 — Runtime, Hooks e `GuardianContextV2`
**Comandos executados em `/tmp/vitalia-test-project`:**
```bash
/home/andre/.vitalia-kit/.venv/bin/python3 /home/andre/.vitalia-kit/src/kit_env_bootstrap.py --cwd /tmp/vitalia-test-project --json
/home/andre/.vitalia-kit/.venv/bin/python3 /home/andre/.vitalia-kit/src/vitalia_hook_runner.py /home/andre/.vitalia-kit/extensions/session-start.toml --phase before --args=""
/home/andre/.vitalia-kit/.venv/bin/python3 /home/andre/.vitalia-kit/src/vitalia_hook_runner.py /home/andre/.vitalia-kit/extensions/session-start.toml --phase after
```

**Resultados:**
- `kit_env_bootstrap.py`: `status: HEALTHY`, `kit_dir: /home/andre/.vitalia-kit`.
- `--phase before`: executou `scan_environment.py`, gravou envelope em `.vitalia/memory/data_storage/context/72a556ad/session-start_c044c0e57624.json`.
- **Achado crítico no envelope JSON:** `"phase_guardrails": {}` e `"phase_profiles": {}` vieram vazios porque `guardian_context.py` busca `profile.get("phase_guardrails")` e `profile.get("phase_profiles")`, mas todos os `25` arquivos `profiles/gp_*.yaml` compilados de `layers/profiles/` usam a chave de topo `guardrails:` (sem `phase_guardrails` nem `phase_profiles`).
- `--phase after`: retornou `status: ready`, `findings: []`.

---

### Teste 4 — Motor de Contexto, Dashboards de Nuvem e `vitalia-clean`
1. **Motor de Contexto (`vitalia_context_engine.py`):**
   - `--action consolidate --cwd /tmp/vitalia-test-project` (`Exit 0`): gravou `README.md` em `/tmp/vitalia-test-project/.vitalia/session/README.md` (fora de `.vitalia/memory/session/`) com `Total: 0 itens`, deixando `.vitalia/memory/session/` vazio (apenas `.gitkeep`).
   - `--action end --payload-file .vitalia/end_payload.json` (conforme `CONTRATOS_WORKFLOWS_SDD.md`): **`Exit 2`** (`unrecognized arguments: --payload-file`).
   - `--action consolidate --render-data-dashboard` (conforme `CONTRATOS_WORKFLOWS_SDD.md`): **`Exit 2`** (`unrecognized arguments: --render-data-dashboard`).
2. **Dashboard de Dados (`data_storage_dashboard_renderer.py`):**
   - Execução direta sem `PYTHONPATH`: **`Exit 1`** (`ModuleNotFoundError: No module named 'core'`).
   - Execução com `PYTHONPATH=/home/andre/.vitalia-kit/src`: imprimiu Markdown apenas no `stdout` (não gravou `README.md` em `.vitalia/memory/data_storage/`), exibiu `context | 0` (pois `vitalia_hook_runner.py` não registra itens em `context/index.jsonl`) e gerou links `file:///tmp/...` incompatíveis com GitHub.
3. **Faxineiro (`vitalia-clean`):**
   - Saída: `Nada para limpar.` porque `vitalia_tmp_cleanup.py` (L110) aponta para `.vitalia/session/data/` em vez de `.vitalia/memory/data_storage/`.

---

### Teste 5 — Sensores VRSS e Handoff
| Script | Exit Code | Resultado Observado |
|---|---|---|
| `sensor_memory_ledger.py --memory-dir .vitalia/memory --json` | `1` | `FAIL` — `session_ledger.json` ausente (nenhum script do kit cria esse arquivo, e o fallback na L81 busca `.vitalia/memory/session/data/` em vez de `data_storage/`). |
| `sensor_tokens_leak.py --context-dir .vitalia/memory/data_storage/context --json` | `0` | `PASS` — `1 arquivos auditados com 0 segredos expostos.` |
| `sensor_path_hygiene.py --scan-dir .agents/skills --json` | `0` | `PASS` — `Higiene aprovada: zero caminhos hardcoded` |
| `sensor_purity.py --skills-dir .agents/skills --json` | `0` | `PASS` — `Pureza de skills aprovada: 100% em conformidade` |
| `vitalia_handoff.py create --payload ...` | `1` | Validação OpenAPI 3.0 (`jsonschema`) ativa e bloqueando payloads incompletos. |

---

### Teste 6 — Auditoria Cruzada: `vitalia-kit/source`, `~/.vitalia-kit/`, `kit_src` e `CONTRATOS_WORKFLOWS_SDD.md`
1. **Paridade `vitalia-kit/source` vs `~/.vitalia-kit/`:**
   - Praticamente idênticos (`diff -rq` apontou apenas `pyproject.toml`, `layers/`, `tests/`, `kit-bootstrap.sh`, `requirements.txt` exclusivos do repositório `source/`). Ou seja, os bugs do `~/.vitalia-kit/` estão na raiz do repositório `vitalia-kit/source/`.
2. **Validadores fantasma em `profiles/constitution.yaml`:**
   - `P04` e `P16` declaram `validator: maintenance/header_stamp.py` (ausente em `source/src/`; versão antiga encontrada em `kit_src/source/src/maintenance/header_stamp.py` chama `kit_env_bootstrap.init()` inexistente, usa `"Vitalia SDD v0.6.0"`, apaga comentários genéricos no topo e usa `any(part.startswith("."))` no caminho absoluto).
   - `P07`, `P10` e `P12` declaram `validator: hooks/llm_judge.py --mode after_task` (também ausente em `source/src/hooks/`).
3. **CLIs com imports internos sem `sys.path`:**
   - `rendering/data_storage_dashboard_renderer.py`, `rendering/grounding_audit_renderer.py` e `vitalia_tmp_cleanup.py` falham se chamados diretamente sem `PYTHONPATH`.
4. **Módulos do `T028` (`specs/005-portao-grounding-p03/tasks.md`) marcados `[X]` mas ausentes no Git:**
   - `core/shard_manager.py`, `core/context_engine.py`, `rendering/view_renderer.py` e `rendering/constitution_renderer.py` nunca foram commitados em `vitalia-kit/source/src/`.
5. **Compressão dos Prompts (`extensions/*.toml`) vs `CONTRATOS_WORKFLOWS_SDD.md`:**
   - Todos os `34` arquivos `.toml` instalados têm `guardian.prompt` entre **64 e 115 caracteres**, omitindo fluxos, portões HITL, templates de saída e a cláusula P03 (`vitalia-researcher`), enquanto `CONTRATOS_WORKFLOWS_SDD.md` especifica prompts completos com dezenas/centenas de linhas.
6. **Outras Discrepâncias Estruturais:**
   - `transport.file_path` dividido: `17` `.toml` usam `.vitalia/pipe.json` e `17` usam `.vitalia/pipeline.json`.
   - `source/templates/` inexistente (citado por `ADR-0012` e `CONTRATOS_WORKFLOWS_SDD.md` para `grounding_audit_template.md`).
   - Identidade do produto: `138` arquivos em `source/` ainda usam `"Vitalia SPEC Kit"` contra apenas `1` com `"Vitalia Kit v"`; e `43` de `70` arquivos `.md` em `vitalia-kit` usam `> **...**` na 2ª linha.
   - `3` suítes em `source/tests/` (`test_sensor_memory_ledger.py`, `test_context_engine_and_grounding_flow_1a_2a.py`, `test_vitalia_tmp_cleanup_spec007.py`) consolidam o caminho legado `.vitalia/session/data`.

---

## 3. Catálogo Consolidado de Erros e Discrepâncias

| Grupo | IDs | Descrição Resumida |
|---|---|---|
| **A — Instaladores (`bin/` & `kit-bootstrap.sh`)** | **A1–A8** | Ausência de logs/verbose (`A1`); falta de prompts interativos `session_repo`/`data_repo` (`A2`); ordem de `${KIT_PYTHON}` em `install-project:93`, `kit-bootstrap.sh:57` e `agent_preserver.sh:32,95,107` (`A3`); `config.yml` sem `kit_version`/`kit_dir`/`project` (`A4`); falta de `feature.json`, `extensions.yml`, `session_ledger.json` e `.gitignore` (`A5`); histórico Git conflitante com GitHub remoto em `git_storage_manager.sh` (`A6`); ausência de injeção de `PATH`/`VITALIA_KIT_DIR` no `~/.bashrc`/`~/.zshrc` em `kit-bootstrap.sh` (`A7`); marca legada `"Vitalia SPEC Kit"` em `138` arquivos (`A8`). |
| **B — Split-Brain Camada 3** | **B1–B9** | `vitalia_context_engine.py`, `dashboard_renderer.py`, `vitalia_tmp_cleanup.py`, `grounding_domain_updater.py`, `grounding_renderer.py`, `data_storage_dashboard_renderer.py`, `sensor_memory_ledger.py` (L81), `SKILL.md.template`, `vitalia_handoff.py`, `6` arquivos `.toml` (incluindo papéis revisores de `science-review.toml` e `integrative-review.toml`) e `3` arquivos em `source/tests/` ainda usam `.vitalia/session/` ou `.vitalia/session/data/`. |
| **C — Dashboards, `sys.path` e Legado** | **C1–C5** | Dashboards não gravam de forma independente em `.vitalia/memory/session/` e `.vitalia/memory/data_storage/` com links relativos (`C1`); `vitalia_hook_runner.py` não indexa em `context/index.jsonl` (`C2`); `pre_commit_runner.py` aponta para `~/.vitalia-spec-kit` (`C3`); `3` CLIs sem `sys.path` (`C4`); `transport.file_path` dividido entre `pipe.json` e `pipeline.json` (`C5`). |
| **D — Constituição, Context Lifting & Contratos** | **D1–D5** | P04 simplificada + portabilidade pendente de `header_stamp.py` (`D1`); `hooks/llm_judge.py` fantasma em `P07`/`P10`/`P12` (`D2`); `gp_*.yaml` usa `guardrails:` enquanto `GuardianContextV2` lê `phase_guardrails`/`phase_profiles`, deixando ambos `{}` no runtime (`D3`); prompts comprimidos (64–115 chars) nos `.toml` vs `CONTRATOS_WORKFLOWS_SDD.md` (`D4`); módulos do `T028` e flags do `vitalia_context_engine.py` ausentes (`D5`). |

