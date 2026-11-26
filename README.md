<!-- README.md | Vitalia Kit v0.0.1 | 25-11-2026 19:54(GMT-04:00) -->
**README.md | Vitalia Kit v0.0.1 | 25-11-2026 19:54(GMT-04:00)**

# vitalia-docs

Repositório de apoio, testes de prontidão e documentação oficial do **Vitalia Kit v0.0.1** (código-fonte do produto em `git@github.com:andreticba/vitalia-kit.git` e instalação global padrão em `~/.vitalia-kit/`).

---

## Mapa da Documentação (`docs/`)

| Ordem / Função | Documento | Estado |
|---|---|---|
| **1. Âncora Histórica & ADRs** | [MEMORANDO DE ARQUITETURA.md](./docs/MEMORANDO%20DE%20ARQUITETURA.md) | Atualizado (Spec 007 / Rodada 3 + `ADR-0014`) |
| **2. Contratos Normativos SDD** | [CONTRATOS_WORKFLOWS_SDD.md](./docs/CONTRATOS_WORKFLOWS_SDD.md) | Reconciliado com Spec 007 (`34` `.toml`, `25` GPs, Schemas `#02`/`#10`/`#19`) |
| **3. Portão de Grounding P03** | [RASCUNHO_PORTAO_GROUNDING_P03.md](./docs/RASCUNHO_PORTAO_GROUNDING_P03.md) | Atualizado (Fluxo 1A/2A, sem `ViewRenderer` / `session/data/`) |
| **4. Mapa Repetível de Testes** | [mapa-testes-vitalia-kit.md](./docs/relatorios/mapa-testes-vitalia-kit.md) | Atualizado (Rodadas 1, 2 e 3 registradas) |
| **5. Templates Visuais de Referência** | [dashboard_template_referencia.md](./docs/templates/dashboard_template_referencia.md) · [grounding_audit_template.md](./docs/templates/grounding_audit_template.md) | `dashboard_template` aplicado na Spec 007; `grounding_audit_template` pronto para Spec 008 |
| **6. Relatório Rodada 3 (Spec 007)** | [relatorio-rodada-3-vitalia-kit.md](./docs/relatorios/relatorio-rodada-3-vitalia-kit.md) | Concluído (`✅ PASS` — `202/202` `pytest`, `2× install-project` append verificado) |
| **7. Histórico Rodadas 1 e 2** | [relatorio-completo-testes-e-erros-turno-1.md](./docs/relatorios/relatorio-completo-testes-e-erros-turno-1.md) · [relatorio-rodada-2-vitalia-kit.md](./docs/relatorios/relatorio-rodada-2-vitalia-kit.md) · [teste-prontidao-e-defeitos-v0.0.1.md](./docs/relatorios/teste-prontidao-e-defeitos-v0.0.1.md) | Arquivados como linha histórica |
| **8. Handoffs & Guias de Correção** | [handoff-rodada-4-portao-p03-v0.0.1.md](./docs/handoff-rodada-4-portao-p03-v0.0.1.md) · [handoff-rodada-3-vitalia-kit-v0.0.1.md](./docs/handoff-rodada-3-vitalia-kit-v0.0.1.md) · [handoff-correcoes-vitalia-kit-v0.0.1.md](./docs/handoff-correcoes-vitalia-kit-v0.0.1.md) · [guia-correcao-memorando-arquitetura-r3.md](./docs/guia-correcao-memorando-arquitetura-r3.md) · [guia-correcao-contratos-sdd-r3.md](./docs/guia-correcao-contratos-sdd-r3.md) | Rodadas 1–3 concluídas; **Rodada 4 (Spec 008)** pronta para execução futura |

---

## Status do Ciclo de Testes
- **Rodada 1 (`23-11-2026`):** ❌ Falhas críticas catalogadas (`A1–A8`, `B1–B9`, `C1–C5`, `D1–D5`), resolvidas na Spec 006.
- **Rodada 2 (`24-11-2026`):** ⚠️ Parcial (`install.log`, dashboards rudimentares, `GuardianContextV2`, higiene residual e contratos SDD), consolidada no handoff da Rodada 3.
- **Rodada 3 (`25-11-2026` — Spec 007 / `48c9eac`):** ✅ **PASS** — `202/202` testes `pytest`, `68/68` arquivos sem deriva, `25/25` CLIs sem `PYTHONPATH`, logs append com `GMT-04:00` comprovados em 2 execuções seguidas e documentação normativa integralmente sincronizada.
- **Rodada 4 (Spec 008 — Pacote Pronto em Handoff):** [handoff-rodada-4-portao-p03-v0.0.1.md](./docs/handoff-rodada-4-portao-p03-v0.0.1.md) + [grounding_audit_template.md](./docs/templates/grounding_audit_template.md) (correção de `research.toml`, unificação de contrato de evidências P03, Opção A nos 6 pontos de alinhamento, além dos itens postergados de `data_storage/README.md` e `phase_profiles`).
