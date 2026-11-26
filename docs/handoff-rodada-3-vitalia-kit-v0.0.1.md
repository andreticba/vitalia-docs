<!-- handoff-rodada-3-vitalia-kit-v0.0.1.md | Vitalia Kit v0.0.1 | 25-11-2026 09:27(GMT-04:00) -->
**handoff-rodada-3-vitalia-kit-v0.0.1.md | Vitalia Kit v0.0.1 | 25-11-2026 09:27(GMT-04:00)**

# Handoff de Correções — Rodada 3 (Vitalia Kit v0.0.1)

> **Destino:** Máquina/sessão responsável pelas correções no repositório `vitalia-kit` (`source/`).  
> **Documentos irmãos neste repositório (`vitalia-docs/docs/`):**
> - Mapa repetível de testes: [mapa-testes-vitalia-kit.md](./relatorios/mapa-testes-vitalia-kit.md)
> - Relatório da Rodada 2: [relatorio-rodada-2-vitalia-kit.md](./relatorios/relatorio-rodada-2-vitalia-kit.md)
> - Template de referência do Dashboard: [dashboard_template_referencia.md](./templates/dashboard_template_referencia.md)
> - Guia de correções do Memorando: [guia-correcao-memorando-arquitetura-r3.md](./guia-correcao-memorando-arquitetura-r3.md)
> - Guia de correções dos Contratos SDD: [guia-correcao-contratos-sdd-r3.md](./guia-correcao-contratos-sdd-r3.md)

---

## 0. Regra de Compilação em Camadas (`source/layers/` $\rightarrow$ `source/profiles/` & `source/extensions/`)

- **Nunca editar diretamente** os arquivos compilados em `source/profiles/` (`constitution.yaml`, `gp_*.yaml`) ou `source/extensions/*.toml`.
- Editar sempre as fontes declarativas em **`source/layers/`** (`source/layers/constitution/base.yaml`, `source/layers/profiles/gp_*.yaml`, `source/layers/extensions/_base.toml` e `*.toml`) e recompilar com `vitalia_layer_compiler.py` (`--check` deve retornar `PASS`).

---

## 1. Regras Inegociáveis de Higiene e Topologia

1. **Erradicação Total de `.vitalia/session/data` e `.vitalia/memory/session/data` (Zero Tolerância):**
   - Os caminhos legados `.vitalia/session/data` e `.vitalia/memory/session/data` (`session/data` / `"session" / "data"`) são inexistentes e **NÃO devem existir no código, nem em docstrings, nem em comentários**. Todo arquivo de sessão reside diretamente na raiz `.vitalia/memory/session/`, e `grounding-domains.jsonl` é projetado em `.vitalia/memory/session/grounding-domains.jsonl` e `.vitalia/memory/data_storage/grounding-domains.jsonl`.
   - Locais já identificados para limpeza imediata em `source/`:
     - `source/src/core/data_storage_io.py:102` (docstring `category_dir`)
     - `source/src/core/grounding_domain_updater.py:11` (docstring)
     - `source/src/vitalia_tmp_cleanup.py:311,329` (remover flag/texto `--include-legacy`)
     - `source/src/hooks/guardian_context.py:138,148-150` (`"session/data/*.jsonl"` + `session_dir / "data"` + `memory / "session" / "data"`)
     - `source/src/rendering/dashboard_renderer.py:29-30` (`legacy = self.session_dir / "data"`)
     - `source/src/sensors/sensor_memory_ledger.py:92` e `source/tests/test_sensor_memory_ledger.py:66` (`"session" / "data"`)
   - **Gate 0 Obrigatório no Início do Ciclo SDD:** Executar a varredura `grep -rnI --exclude-dir=__pycache__ "\.vitalia/session/data\|memory/session/data\|session/data\|session_dir\s*/\s*\"data\"\|\"session\"\s*/\s*\"data\"" source/` e validar a raiz física de `.vitalia/memory/session/` **logo no início da implementação deste turno (antes de evoluir qualquer módulo)** e novamente ao concluir, garantindo **0 ocorrências** e dotando `sensor_path_hygiene.py` e `sensor_memory_ledger.py` de bloqueio preventivo contínuo nos hooks VRSS.
2. **Erradicação de Legado `vitalia-spec-kit` / `"Vitalia SPEC Kit"`:**
   - Corrigir `source/src/tools/pre_commit_runner.py:197-198` (`$HOME/.vitalia-spec-kit` $\rightarrow$ `$HOME/.vitalia-kit`).
   - Corrigir `source/src/core/data_storage_io.py:27` (remover `Path.home() / ".vitalia-spec-kit"`).
   - Substituir `"Vitalia SPEC Kit"` por `"Vitalia Kit"` em: `source/bin/vitalia-clean:2`, `source/src/schemas/vitalia-handoff-v1.0.0.json:2`, `source/src/__init__.py:2`, `source/src/kit_env_bootstrap.py:4`, `source/src/vitalia_hook_runner.py:4`, `source/src/vitalia_handoff.py:263`, `source/src/vitalia_context_engine.py:82`, `source/src/vitalia_grounding_validator.py:201`, `source/src/vitalia_tmp_cleanup.py:294`, `source/src/sdd_ambiguity_scorer.py:153`, `source/src/core/utils.py:3`, `source/src/telemetry/__init__.py:2`, `source/src/hooks/domain_detector.py:4`, `source/src/hooks/guardian_context.py:4,58`, `source/src/hooks/scan_environment.py:4,140`, `source/src/sensors/sensor_contract_dbc.py:4,62`, `source/src/sensors/sensor_purity.py:4,72`, `source/src/sensors/sensor_path_hygiene.py:4,57`.

---

## 2. Escopo de Correções para a Rodada 3

### Bloco 1 — Instalação & Diagnóstico (`install.log` e `.gitignore`)
1. **`source/bin/install-project`:**
   - **Selo de data/hora e Modo Append (`L150-169`):** Remover o truncamento `: > "${INSTALL_LOG}"` na `L151`. Em `_out()` (`L153-155`) e no banner (`L166-169`), prefixar com timestamp no fuso imutável `America/Cuiaba` (`DD-MM-YYYY HH:MM:SS(GMT-04:00)`), fazendo **append** (`tee -a "${INSTALL_LOG}"`) com delimitador claro entre tentativas de instalação.
   - **`.gitignore` (`L343-348`):** Na lista `for entry in ...` da `L345`, adicionar `".env"`, `".vitalia/config.yml"` e `".vitalia/backups/"` (e remover `".vitalia/pipe.json"` / `".vitalia/pipeline.json"` se não forem mais gerados, mantendo se desejar apenas proteção residual).
2. **`source/kit-bootstrap.sh` (`L16-93`):**
   - Gravar `<kit_dir>/logs/install.log` também com timestamp `GMT-04:00` e em modo append.

### Bloco 2 — Componente Único de Dashboards (Foco Rodada 3: `session_repo`)
- **Arquitetura Alvo:** Unificar a renderização de dashboards em **um único componente** (`source/src/rendering/dashboard_renderer.py`) orientado por templates específicos (ex.: `source/templates/dashboard_template.md` para `session_repo` e template dedicado para `data_repo`), eliminando duplicação de lógica de badges, cabeçalhos P04 e links relativos.
- **Escopo Aceito para a Rodada 3 (Sem Inchar Escopo):**
  - Priorizar e entregar **corretamente** o **Dashboard de Sessão (`.vitalia/memory/session/README.md`)** baseado no template elaborado resgatado em [dashboard_template_referencia.md](./templates/dashboard_template_referencia.md).
  - O `session/README.md` deve apresentar:
    - Badges `shields.io` (`Status`, `Semáforo`/Estado, `Ambiente`/Modo, `Sync GMT-04:00`, `Grounding`);
    - Diagrama **Mermaid** (`flowchart TD` com labels protegidos por aspas duplas — P16) da topologia de shards multi-máquina;
    - Tabela de **Máquinas / Shards** lida de `session_ledger.json` (`machine_id`, `task`, `status`, `last_sync`, `next_step`);
    - Destaque da sessão ativa, **Histórico de Sessões**, **Decisões Arquiteturais** (`decisions.jsonl`), **Aprendizados** (`learnings.jsonl`) e **Guard Rails / Domínios** (`GROUNDING_DOMAINS.md`);
    - Cabeçalho P04 no padrão atual (`<!-- README.md | Vitalia Kit v0.0.1 | {ts} -->` e `**README.md | Vitalia Kit v0.0.1 | {ts}**` sem `> `) e links relativos compatíveis com GitHub (sem `file:///` e sem `grounding-domains-local.yaml`).
  - A evolução visual equivalente do `data_storage/README.md` fica formalmente postergada para a rodada seguinte (mantendo a geração básica atual sem regressão de `index.jsonl`).

### Bloco 3 — `GuardianContextV2` (`phase_guardrails` e `phase_profiles` — Apenas Resgate)
- **Problema:** `source/layers/profiles/gp_*.yaml` usam chaves planas (`phase:`, `role:`, `guardrails:`), enquanto `GuardianContextV2` (`source/src/hooks/guardian_context.py:255-256`) e `source/src/schemas/guardian_profile.schema.json` esperam `phase_guardrails:` e `phase_profiles:`. Resultado: ambos chegam `{}` vazios no envelope JSON.
- **Correção (Opção 1 — Resgate Direto, sem expansão de escopo):**
  1. Atualizar `source/layers/profiles/gp_base.yaml` e os `gp_*.yaml` para o schema canônico com `phase_guardrails:` (`phase`, `role`, `mandatory_checks`) e restaurar `phase_profiles:` em `source/layers/profiles/gp_sdd-judge.yaml`, recompilando via `vitalia_layer_compiler.py`. *(A expansão de `phase_profiles` para outros agentes fica postergada para uma spec futura).*

### Bloco 4 — Contratos, Constituição, Transporte e CLI
1. **`source/layers/constitution/base.yaml:64,76,88`:** Substituir `validator: "hooks/llm_judge.py --mode after_task"` em `P07`, `P10` e `P12` por validadores reais existentes (e recompilar `source/profiles/constitution.yaml`).
2. **Descontinuação de `.vitalia/pipeline.json` e `.vitalia/pipe.json` (`source/layers/extensions/`):** Remover `file_path = ".vitalia/pipeline.json"` em `source/layers/extensions/_base.toml:17` e `file_path = ".vitalia/pipe.json"` nos 17 `.toml` em `source/layers/extensions/` (`adr.toml:18`, `analyze.toml:18`, `brainstorming.toml:18`, `clarify.toml:21`, `converge.toml:18`, `debug.toml:18`, `integrative-review.toml:18`, `pair.toml:18`, `release.toml:18`, `review.toml:18`, `route.toml:18`, `science-review.toml:18`, `session-consolidate.toml:21`, `session-end.toml:18`, `session-start.toml:21`, `skill-evaluation.toml:21`, `task-verifier.toml:18`), mantendo `preferred = "filesystem"` e `session_shard = true`, e recompilar `source/extensions/*.toml`. O transporte de estado em filesystem entre etapas opera via `data_storage/context/`, `data_storage/handoffs/` e payloads transitórios em `.vitalia/`.
3. **`source/src/vitalia_context_engine.py:91-106`:** Implementar/expor as flags contratuais faltantes (`--action {end,list-pending-domains,consolidate}`, `--payload-file`, `--render-session-dashboard`, `--render-data-dashboard`, `--render-grounding`, `--update-grounding-domains`, `--grounding-decisions-file`) sem retornar `exit 2` (mantendo o provisionamento estrutural da Camada 3 restrito ao `install-project`).

---

## 3. Critérios de Aceitação e Verificação na Máquina Destino
1. **Compilação sem deriva:** Executar `vitalia_layer_compiler.py --check` (`PASS`).
2. **Zero legado:** `grep -rnI --exclude-dir=__pycache__ "session/data\|\"session\"\s*/\s*\"data\"\|vitalia-spec-kit\|Vitalia SPEC Kit" source/` retorna `0` ocorrências ativas.
3. **Suíte `pytest`:** Executar a partir da **raiz do repositório `vitalia-kit/`**: `<kit_python> -m pytest source/tests/` com 100% de aprovação.
4. **Testes end-to-end:** Executar a bateria definida em [mapa-testes-vitalia-kit.md](./relatorios/mapa-testes-vitalia-kit.md) (T1 a T7) em sandbox isolado (`/tmp/`).
