<!-- dashboard_template_referencia.md | Vitalia Kit v0.0.1 | 24-11-2026 16:38(GMT-04:00) -->
**dashboard_template_referencia.md | Vitalia Kit v0.0.1 | 24-11-2026 16:38(GMT-04:00)**

> **Origem:** Resgatado de `vitalia-spec-kit/source/templates/dashboard_template.md` (versão mais elaborada do painel de contexto multi-máquina).  
> **Nota de Referência (P04/P11/Camada 3):** Ao adaptar este template para o componente único de dashboards no Vitalia Kit v0.0.1+, atualizar a marca para `Vitalia Kit v0.0.1`, a 2ª linha para `**README.md | Vitalia Kit v0.0.1 | {timestamp}**` (sem `> `) e usar links relativos compatíveis com GitHub (substituindo referências a `grounding-domains-local.yaml` por `GROUNDING_DOMAINS.md`).

---

<!-- dashboard_template.md | Vitalia SPEC Kit v0.0.1 | 08-11-2026 20:12(GMT-04:00) -->
**README.md | Vitalia SPEC Kit v0.0.1 | {timestamp}**

# 🧠 Painel de Contexto — {project_name}

{badge_status}
{badge_semaforo}
{badge_modo}
{badge_sync}
{badge_grounding}

> **Vitalia SPEC Kit v0.0.1 — Ledger de Memória Persistente e Orquestração Multi-Máquina.**  
> Este repositório armazena o histórico distribuído, aprendizados consolidados e o controle de concorrência das sessões de trabalho do framework Vitalia.

---

## 📡 Topologia de Shards & Sincronização

<div align="center">

```mermaid
flowchart TD
{mermaid_content}
```

</div>

---

## 🖥️ Máquinas e Status Atual

<table>
  <thead>
    <tr>
      <th align="left">Máquina / ID</th>
      <th align="left">Tarefa Atual</th>
      <th align="center">Ambiente</th>
      <th align="center">Status</th>
      <th align="left">Último Sync</th>
      <th align="left">Próximo Passo (P0)</th>
    </tr>
  </thead>
  <tbody>
{shards_table}
  </tbody>
</table>

---

## 🎯 Sessão Ativa em Destaque

{sessao_ativa}

---

## 📚 Histórico, Decisões & Guard Rails

<details>
<summary><strong>🔍 Clique para expandir o Histórico Completo de Sessões</strong></summary>

<br/>

{historico_sessoes}
</details>

<details>
<summary><strong>⚖️ Clique para expandir as Decisões Arquiteturais Consolidadas</strong></summary>

<br/>

| Máquina (ID) | Decisão Arquitetural | Impacto / Racional |
| :--- | :--- | :--- |
{decisoes_arquiteturais}
</details>

<details>
<summary><strong>🛡️ Clique para expandir os Guard Rails de Grounding e Domínios</strong></summary>

<br/>

| Arquivo de Regras | Status | Domínios Monitorados | Pendentes de Curadoria HITL |
| :--- | :---: | :--- | :---: |
| `grounding_domains.yaml` (Global) | {global_status} | `provedores_de_ia`, `python_packages`, `web_technologies`, `external_apis`, `security_practices`, `regulations_compliance`, `cloud_services`, `scientific_claims`, `clinical_safety`, `software_architecture` | — |
| {local_link} (Projeto) | {local_status} | Domínios locais específicos do workspace | {pending_badge} |

</details>

---

<sub>Painel gerado automaticamente pelo motor de contexto do Vitalia Kit (<code>vitalia_context_engine.py --action consolidate</code>).</sub>

