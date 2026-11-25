<!-- relatorio-rodada-2-vitalia-kit.md | Vitalia Kit v0.0.1 | 24-11-2026 11:58(GMT-04:00) -->
**relatorio-rodada-2-vitalia-kit.md | Vitalia Kit v0.0.1 | 24-11-2026 11:58(GMT-04:00)**

# Relatório de Testes — Rodada 2 e Plano de Mudanças para Rodada 3 (Vitalia Kit v0.0.1)

> **Mapa de referência:** [mapa-testes-vitalia-kit.md](file:///home/andre/projetos/assistidos/vitalia-docs/docs/relatorios/mapa-testes-vitalia-kit.md)  
> **Ambiente de teste isolado:** `/tmp/vitalia-test-r2` e repositório `/home/andre/projetos/assistidos/vitalia-kit`

---

## 1. Sumário Executivo da Rodada 2

| Suíte | Status | Evidência Principal |
|---|---|---|
| **T1 — Bootstrap Global (`kit-bootstrap.sh`)** | ⚠️ Parcial | Flags `VERBOSE` e `NON_INTERACTIVE` definidas (L17–18, L41–45), mas sem log persistente global (`<kit_dir>/logs/install.log`). |
| **T2 — `install-project` & `install.log`** | ⚠️ Parcial (Falha Crítica em `install.log`) | Progresso 8 passos e artefatos (`config.yml`, `feature.json`, `extensions.yml`, `session_ledger.json`) OK. **Porém:** `install.log` **não tem data/hora** e é **apagado/sobrescrito** a cada execução (`32` linhas na 1ª -> `30` linhas na 2ª). `.gitignore` não protege `.env`, `.vitalia/config.yml` nem `.vitalia/backups/`. |
| **T3 — Runtime, Hooks e `GuardianContextV2`** | ❌ Falha Parcial | `kit_env_bootstrap.py` (`HEALTHY`) e `index.jsonl` OK. **Porém:** `phase_guardrails: {}` e `phase_profiles: {}` seguem vazios; `constitution.yaml` mantém `hooks/llm_judge.py` em `P07`/`P10`/`P12`; `extensions/*.toml` seguem divididos (`17` `pipe.json` / `17` `pipeline.json`) e com prompts de `64–115` chars. |
| **T4 — Dashboards (`session_repo` e `data_repo`)** | ❌ Falha de Conteúdo & UX | Execução direta sem `PYTHONPATH` e gravação em `.vitalia/memory/{session,data_storage}/README.md` com links relativos OK. **Porém:** ambos geram painéis rudimentares (`29` e `23` linhas); `session/README.md` omite `session_ledger.json`, shards, `decisions.jsonl`, `learnings.jsonl` e `feature.json`; `vitalia_context_engine.py` ainda rejeita `--action init` e `--render-data-dashboard` (`exit 2`). |
| **T5 — Faxineiro, Sensores e `header_stamp.py`** | ✅ / ⚠️ | `vitalia-clean`, `sensor_memory_ledger.py`, `sensor_tokens_leak.py`, `sensor_path_hygiene.py`, `sensor_purity.py` e `header_stamp.py --help` passaram (`exit 0`). **Pendente:** `pre_commit_runner.py:197-198` ainda referencia `$HOME/.vitalia-spec-kit/.venv/bin/python3`. |
| **T6 — CLIs sem `PYTHONPATH` & Higiene** | ⚠️ Parcial | 100% dos CLIs em `src/` respondem `--help` sem `PYTHONPATH`. **Porém:** `20+` ocorrências de `"Vitalia SPEC Kit"`, `.vitalia-spec-kit` (`data_storage_io.py:27`) e `.vitalia/session/data` em docstrings/comentários. |
| **T7 — Suíte `pytest` (`vitalia-kit/source`)** | ✅ PASS (com ressalva de CWD) | `181 passed in 34.15s` executando a partir de `vitalia-kit/` (`test_finding_and_registry.py` exige raiz no `PYTHONPATH`). |

---

## 2. Evolução: O que Foi Corrigido da Rodada 1 para a Rodada 2

1. `install-project` agora cria `.vitalia/feature.json`, `.vitalia/extensions.yml`, `.vitalia/memory/session/session_ledger.json` e `.vitalia/config.yml` completo (`kit_version`, `kit_dir`, `project`, `storage`, `memory`, `installed_at`).
2. `vitalia_hook_runner.py` agora registra automaticamente os envelopes criados em `.vitalia/memory/data_storage/context/index.jsonl`.
3. `dashboard_renderer.py` e `data_storage_dashboard_renderer.py` injetam `src/` no `sys.path`, gravam `README.md` dentro dos repositórios da Camada 3 e usam links relativos (sem `file:///`).
4. `sensor_memory_ledger.py` agora retorna `PASS` (`exit 0`) em projeto recém-instalado.
5. `src/maintenance/header_stamp.py` foi restaurado no kit.

---

## 3. Mudanças Necessárias para a Rodada 3 (Priorizadas)

### Prioridade 1 — Bloqueadores de Diagnóstico e UX (Novos da Rodada 2)
1. **`install.log` com Timestamp GMT-04:00 e Modo Append (Não-Destrutivo):**
   - Prefixar cada execução em `.vitalia/install.log` (e `<kit_dir>/logs/install.log` no `kit-bootstrap.sh`) com separador claro e timestamp `DD-MM-YYYY HH:MM:SS(GMT-04:00)` em cada linha/bloco.
   - Trocar sobrescrita (`>`) por **append** (`>>`) para preservar o histórico de tentativas de instalação, além de opcionalmente espelhar cópia em `.vitalia/memory/data_storage/telemetry/install.log`.
2. **Dashboards Ricos (`session_repo` e `data_repo`):**
   - **`dashboard_renderer.py` (`session_repo/README.md`):** Restaurar e evoluir o painel rico de sessão exibindo:
     - Status da sessão, `machine_id`, feature ativa (`feature.json`) e último sync;
     - Tabela de **Shards** (`session_ledger.json`: `shard_id`, `task`, `next_step`, `status`, `last_sync`);
     - Resumo de **Decisões** (`decisions.jsonl`) e **Aprendizados** (`learnings.jsonl`);
     - Diagrama Mermaid (`flowchart` / `stateDiagram` com labels entre aspas duplas — P16) e métricas da Camada 3 com links relativos.
   - **`data_storage_dashboard_renderer.py` (`data_repo/README.md`):** Evoluir o Nível 0 e Nível 1 com visão visual rica (cards/métricas por categoria, cobertura de `index.jsonl`, timestamp da última atualização e navegação clara Nível 0 ↔ Nível 1 ↔ Nível 2).

### Prioridade 2 — Pendências Estruturais da Rodada 1 Ainda Abertas
3. **`GuardianContextV2` (`src/hooks/guardian_context.py`):** Alinhar leitura de `guardrails:`/`phase:`/`role:` dos `gp_*.yaml` para que `phase_guardrails` e `phase_profiles` não cheguem vazios (`{}`) no envelope JSON.
4. **Constituição (`profiles/constitution.yaml`):** Substituir `validator: hooks/llm_judge.py --mode after_task` em `P07`, `P10`, `P12` por validadores reais existentes.
5. **`.gitignore` no `install-project`:** Adicionar `.env`, `.vitalia/config.yml` e `.vitalia/backups/` além das regras atuais.
6. **Contratos & CLI (`extensions/*.toml` e `vitalia_context_engine.py`):**
   - Unificar `transport.file_path` em todos os 34 `.toml` (`.vitalia/pipe.json` ou `.vitalia/pipeline.json`).
   - Restaurar prompts completos nos `.toml` (não comprimidos em 1 linha).
   - Implementar flags faltantes em `vitalia_context_engine.py` (`--action init`, `--payload-file`, `--render-data-dashboard`).
7. **Limpeza de Legado (`C3` e `A8`):**
   - Corrigir `pre_commit_runner.py:197-198` (`$HOME/.vitalia-spec-kit` -> `$HOME/.vitalia-kit`).
   - Corrigir `data_storage_io.py:27` (`~/.vitalia-spec-kit` -> `~/.vitalia-kit`) e remover strings `"Vitalia SPEC Kit"` nos cabeçalhos/descrições em `src/` e `bin/vitalia-clean`.

