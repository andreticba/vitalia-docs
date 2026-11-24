<!-- teste-prontidao-e-defeitos-v0.0.1.md | Vitalia Kit v0.0.1 | 22-11-2026 12:50(GMT-04:00) -->
**teste-prontidao-e-defeitos-v0.0.1.md | Vitalia Kit v0.0.1 | 22-11-2026 12:50(GMT-04:00)**

# Relatório Verboso de Testes de Prontidão e Mapeamento de Defeitos — Vitalia Kit v0.0.1

> **Escopo:** Auditoria estática integral de `/home/andre/.vitalia-kit` e execução de testes reais em sandbox isolado (`/tmp/vitalia-test-project`) com repositórios GitHub reais, servindo de insumo direto para a nova `spec` de correções do Kit.

---

## 1. Sumário Executivo

| Categoria | Status | Achados Principais |
|---|---|---|
| **Instalação Global & Projeto (`bin/`)** | ❌ Falhas Críticas | Sem logs em terminal/arquivo na instalação global e no `install-project`; sem prompts interativos para `session_repo`/`data_repo`; `git_storage_manager.sh` cria histórico órfão conflitante com repositórios remotos existentes; `config.yml` incompleto; ausência de `.vitalia/feature.json`, `extensions.yml` e `.gitignore`. |
| **Camada 3 & Split-Brain de Caminhos** | ❌ Falhas Críticas | Divergência entre `.vitalia/memory/data_storage/` + `.vitalia/memory/session/` vs `.vitalia/session/data/` + `.vitalia/session/` em 7 módulos/templates e 6 arquivos `.toml`. |
| **Dashboards de Nuvem (`session_repo` / `data_repo`)** | ❌ Falhas Críticas | `dashboard_renderer.py` grava fora do repo Git e conta `0` itens; `data_storage_dashboard_renderer.py` falha com `ModuleNotFoundError: No module named 'core'` ao rodar diretamente e não grava arquivo em `data_storage/`. |
| **Referências Legadas (`~/.vitalia-spec-kit`)** | ⚠️ Defeito | `pre_commit_runner.py` gera hook buscando `$HOME/.vitalia-spec-kit/.venv/bin/python3`. |
| **Sensores & Higiene (`src/sensors/`)** | ⚠️ Parcial | `sensor_tokens_leak`, `sensor_path_hygiene` e `sensor_purity` passaram; `sensor_memory_ledger` falha (`session_ledger.json` não inicializado pelo instalador). |
| **Sintaxe Python, P10 (`--help`) e Schemas** | ✅ Saudável | 0 erros de sintaxe; 23/23 CLIs com `--help` didático; 34 `.toml`, `.yaml` e `.json` válidos. |

---

## 2. Catálogo Completo de Defeitos (Para a Nova Spec)

### Categoria A — Instalação Global e de Projeto (`bin/`)
1. **A1 — Ausência de Logs e Pré-voo:**
   - A instalação global não persiste log nem detalha etapas no terminal (e `kit-bootstrap.sh` não é preservado/referenciado no kit instalado).
   - `install-project` ([install-project](file:///home/andre/.vitalia-kit/bin/install-project)) importa `logger.sh` e `preflight.sh`, mas não chama `log_step`/`log_ok` nem as validações (`check_git_requirements`, `check_writable_target`, `check_python_version_semver`) e não grava arquivo de log.
2. **A2 — Falta de Prompts Interativos para Repositórios de Nuvem:**
   - Quando `--session-repo` e `--data-repo` são omitidos em modo interativo (`NON_INTERACTIVE=0`), `install-project` não solicita as URLs ao usuário.
3. **A3 — Ordem de `${KIT_PYTHON}`:**
   - `install-project:93` chama `python3` do sistema antes de resolver `KIT_PYTHON` na linha 96.
   - `agent_preserver.sh` (linhas 32, 95, 107) chama `python3` do sistema em vez de `${KIT_PYTHON}` (mesmo sendo invocado após a criação/resolução do `.venv`).
   - *Regra:* Toda chamada Python do kit deve usar `${KIT_PYTHON}` após a resolução/instalação do `.venv` global; apenas testes do ciclo SDD utilizam o `.venv` do projeto cliente.
4. **A4 — `config.yml` Incompleto (`env_manager.sh`):**
   - `update_project_config` grava apenas `domain_gates` e `memory`, omitindo `kit_version`, `kit_dir` e `project`.
5. **A5 — Artefatos Iniciais Ausentes no Projeto:**
   - `install-project` não cria `.vitalia/feature.json`, `.vitalia/extensions.yml`, `session_ledger.json` e não atualiza `.gitignore` para proteger `.vitalia/memory/`, `.vitalia/config.yml` e `.env`.
6. **A6 — Histórico Git Divergente com Repositórios Remotos Existentes (`git_storage_manager.sh`):**
   - Quando `.git` ainda não existe localmente, `setup_git_repo` cria um commit inicial órfão (`init: session tracking` / `init: data storage`) e apenas executa `git remote add origin <url>`.
   - Se o repositório remoto no GitHub já tiver commits em `main`, qualquer `git push` futuro será rejeitado (`non-fast-forward` / históricos não relacionados).
   - Se `.git` já existir localmente, `setup_git_repo` retorna `0` sem atualizar `remote origin` caso uma nova URL seja informada.

---

### Categoria B — Split-Brain de Caminhos na Camada 3
Os repositórios reais da Camada 3 são irmãos e independentes:
- **Sessão (`session_repo`):** `.vitalia/memory/session/`
- **Dados (`data_repo`):** `.vitalia/memory/data_storage/`

Componentes ainda apontando para caminhos legados (`.vitalia/session/` ou `.vitalia/session/data/`):
- **B1:** `vitalia_context_engine.py` (L32, L64): usa `.vitalia/session` em `cmd_end` e `cmd_consolidate`.
- **B2:** `dashboard_renderer.py` (L20): deriva `self.storage_dir = self.session_dir / "data"`.
- **B3:** `vitalia_tmp_cleanup.py` (`vitalia-clean`, L110): busca em `.vitalia/session/data` e não enxerga arquivos em `.vitalia/memory/data_storage/context/`.
- **B4:** `core/grounding_domain_updater.py` (L68): usa `.vitalia/session/data`.
- **B5:** `rendering/grounding_renderer.py` (L26-L27): usa `.vitalia/session/data` e `.vitalia/session`.
- **B6:** `rendering/data_storage_dashboard_renderer.py` (L131): default `--storage-dir` é `.vitalia/session/data`.
- **B7:** `integrations/antigravity/SKILL.md.template` (L18) e `vitalia_handoff.py` (epílogo `--help`, L269) citam `.vitalia/session/data/`.
- **B8 — Análise dos 6 arquivos `.toml`:**
  - `session-start.toml`, `session-end.toml`, `session-consolidate.toml`, `route.toml`: atualizar `[context]` (`reads`/`writes`) para `.vitalia/memory/session/` e `.vitalia/memory/data_storage/` (e subpastas `handoffs/`).
  - `science-review.toml`: transformar em **revisor científico/metodológico independente** — `reads = [".vitalia/memory/data_storage/research/"]`, `writes = [".vitalia/memory/data_storage/reviews/"]`, executando `vitalia_grounding_validator.py` para checagem mecânica (sem sobrepor o validador determinístico), mantendo `gp_research` e exigindo delegação exclusiva a `vitalia-researcher` para qualquer acesso externo (P03).
  - `integrative-review.toml`: transformar em **síntese/meta-análise cruzada** — `reads = [".vitalia/memory/data_storage/research/", ".vitalia/memory/data_storage/reviews/"]`, `writes = [".vitalia/memory/data_storage/research/", ".vitalia/memory/data_storage/reviews/"]`, validando o artefato final com `vitalia_grounding_validator.py` e também com `vitalia-researcher` obrigatório para acesso externo.

---

### Categoria C — Dashboards de Nuvem, `sys.path` e Referências Legadas
- **C1:** `data_storage_dashboard_renderer.py` falha com `ModuleNotFoundError: No module named 'core'` ao ser executado diretamente (não adiciona `src/` ao `sys.path`), apenas imprime no `stdout` sem gravar arquivo em `.vitalia/memory/data_storage/`, e usa links `file:///...` inválidos no GitHub.
- **C2:** `vitalia_hook_runner.py` grava envelopes em `.vitalia/memory/data_storage/context/<machine_id>/`, mas não registra no `context/index.jsonl`, deixando os contadores dos dashboards zerados.
- **C3:** `src/tools/pre_commit_runner.py` (L197-L198) referencia `$HOME/.vitalia-spec-kit/.venv/bin/python3` em vez de `$HOME/.vitalia-kit/.venv/bin/python3`.
- **C4:** `src/rendering/grounding_audit_renderer.py` e `src/vitalia_tmp_cleanup.py` também fazem imports internos (`from core...`) dentro de funções sem injetar `src/` no `sys.path`, falhando ao executar diretamente via `${KIT_PYTHON}` sem `PYTHONPATH`.

---

### Categoria D — Constituição (`constitution.yaml`) e Validadores Ausentes
- **D1 (`P04` resumida + `maintenance/header_stamp.py` ausente):**
  - A regra `P04` no `constitution.yaml` atual foi resumida e omitiu o formato de 2 linhas para `.md` (`<!-- {file} | Vitalia Kit v{VERSION} | {ts} -->` + `**{file} | Vitalia Kit v{VERSION} | {ts}**` sem `> `).
  - O validador declarado (`maintenance/header_stamp.py`) não existe no kit atual; a versão encontrada em `kit_src/source/src/maintenance/header_stamp.py` quebra na importação (`kit_env_bootstrap.init()` inexistente), usa versão hardcoded `Vitalia SDD v0.6.0`, consome comentários legítimos no topo, bloqueia caminhos com diretórios ocultos e não trata `.json` como validação sintática apenas (`json.loads`, compatível com `P11` OpenAPI 3.0 limpo).
- **D2 (`P07`, `P10`, `P12` — validador fantasma `hooks/llm_judge.py`):**
  - `profiles/constitution.yaml` (L86, L98, L112) declara `validator: hooks/llm_judge.py --mode after_task`, porém `hooks/llm_judge.py` foi removido na migração de `kit_src` para o Vitalia Kit v0.0.1.

---

## 3. Logs Verbosos dos Testes Executados (`/tmp/vitalia-test-project`)

### Teste 1 — Instalação no Projeto com Repositórios GitHub Reais
**Comando:**
```bash
/home/andre/.vitalia-kit/bin/install-project \
  --project-dir /tmp/vitalia-test-project \
  --session-repo "git@github.com:andreticba/vitalia-kit-context.git" \
  --data-repo "git@github.com:andreticba/vitalia-kit-data.git" \
  --yes
```
**Saída obtida (Exit 0):**
```text
Skills geradas com sucesso: 34
✅ Vitalia SPEC Kit instalado com sucesso no projeto: /tmp/vitalia-test-project
```
**`.vitalia/config.yml` gerado (incompleto — faltam `kit_version`, `kit_dir`, `project`):**
```yaml
domain_gates: []
memory:
  session_repo: git@github.com:andreticba/vitalia-kit-context.git
  data_repo: git@github.com:andreticba/vitalia-kit-data.git
```
**Estado Git local vs Retorno Real do GitHub (`git ls-remote`):**
- Local `session`: `5784972 (HEAD -> main) init: session tracking` | `origin = git@github.com:andreticba/vitalia-kit-context.git`
- GitHub `vitalia-kit-context` (`EXIT_CONTEXT=0`):
  ```text
  2564d1be7e8eff3e8b1bb7d640d212d0de6876b0	HEAD
  2564d1be7e8eff3e8b1bb7d640d212d0de6876b0	refs/heads/main
  ```
- Local `data_storage`: `936d000 (HEAD -> main) init: data storage` | `origin = git@github.com:andreticba/vitalia-kit-data.git`
- GitHub `vitalia-kit-data` (`EXIT_DATA=0`):
  ```text
  b8027a90b82b282889bd2b463971f4cce5b44547	HEAD
  b8027a90b82b282889bd2b463971f4cce5b44547	refs/heads/main
  ```
> **Diagnóstico:** Os hashes locais (`5784972`, `936d000`) divergem dos hashes remotos (`2564d1be`, `b8027a90`). Sem `fetch`/`pull` ou `clone`, o `push` falhará.

---

### Teste 2 — Hook Runner (`--phase before` e `--phase after`) & `kit_env_bootstrap.py`
- `kit_env_bootstrap.py --cwd /tmp/vitalia-test-project --json` -> `status: HEALTHY`, `kit_dir: /home/andre/.vitalia-kit`.
- `vitalia_hook_runner.py extensions/session-start.toml --phase before` -> executou `scan_environment.py`, gravou envelope em `.vitalia/memory/data_storage/context/72a556ad/session-start_c044c0e57624.json` (`status: ready`).
- `vitalia_hook_runner.py extensions/session-start.toml --phase after` -> `status: ready`, `findings: []`.

---

### Teste 3 — Motor de Contexto, Dashboards e Faxineiro (`vitalia-clean`)
1. `vitalia_context_engine.py --action consolidate --cwd /tmp/vitalia-test-project` (`Exit 0`):
   - Criou `/tmp/vitalia-test-project/.vitalia/session/README.md` (fora de `.vitalia/memory/session/`) com `Total: 0 itens`.
   - `/tmp/vitalia-test-project/.vitalia/memory/session/` permaneceu apenas com `.gitkeep`.
2. `data_storage_dashboard_renderer.py --storage-dir /tmp/vitalia-test-project/.vitalia/memory/data_storage --level 0`:
   - Execução direta: **`Exit 1`** (`ModuleNotFoundError: No module named 'core'`).
   - Com `PYTHONPATH=/home/andre/.vitalia-kit/src`: imprimiu Markdown no `stdout` com `context | 0` (mesmo havendo 1 arquivo em `context/72a556ad/`) e links `file:///tmp/...`.
3. `vitalia-clean --cwd /tmp/vitalia-test-project`:
   - Saída: `Nada para limpar.` (procurou em `.vitalia/session/data/` em vez de `.vitalia/memory/data_storage/`).

---

### Teste 4 — Sensores (`src/sensors/`) e Handoff
| Script | Exit Code | Resultado Observado |
|---|---|---|
| `sensor_memory_ledger.py --memory-dir .vitalia/memory --json` | `1` | `FAIL` — `Arquivo session_ledger.json ausente em /tmp/vitalia-test-project/.vitalia/memory` |
| `sensor_tokens_leak.py --context-dir .vitalia/memory/data_storage/context --json` | `0` | `PASS` — `1 arquivos auditados com 0 segredos expostos.` |
| `sensor_path_hygiene.py --scan-dir .agents/skills --json` | `0` | `PASS` — `Higiene aprovada: zero caminhos hardcoded` |
| `sensor_purity.py --skills-dir .agents/skills --json` | `0` | `PASS` — `Pureza de skills aprovada: 100% em conformidade` |
| `vitalia_handoff.py create --payload ...` | `1` | Validação OpenAPI 3.0 / `jsonschema` ativa (`'schema_type' is a required property` quando payload mínimo é enviado). |

---

### Teste 5 — Auditoria Cruzada (`constitution.yaml`, `sys.path`, `extensions/*.toml` e `kit_src`)
1. **Validadores em `profiles/constitution.yaml`:**
   ```text
   MISSING VALIDATOR for P04: maintenance/header_stamp.py
   MISSING VALIDATOR for P07: hooks/llm_judge.py --mode after_task
   MISSING VALIDATOR for P10: hooks/llm_judge.py --mode after_task
   MISSING VALIDATOR for P12: hooks/llm_judge.py --mode after_task
   MISSING VALIDATOR for P16: maintenance/header_stamp.py
   ```
2. **CLIs com imports internos sem `sys.path`:**
   ```text
   CLI WITHOUT sys.path: rendering/data_storage_dashboard_renderer.py
   CLI WITHOUT sys.path: rendering/grounding_audit_renderer.py
   CLI WITHOUT sys.path: vitalia_tmp_cleanup.py
   ```
3. **Verificação de flags `[scripts]` em `extensions/*.toml`:**
   - `clarify.toml` (`sdd_ambiguity_scorer.py --spec --findings --scoring --output`), `session-consolidate.toml`, `session-start.toml` e `skill-evaluation.toml` possuem todas as flags e scripts correspondentes válidos.
4. **Inspeção de `kit_src/source/src/maintenance/header_stamp.py`:**
   - Confirmado que chama `kit_env_bootstrap.init()` (inexistente no kit novo), usa string hardcoded `Vitalia SDD v0.6.0` e consome comentários genéricos no topo.

