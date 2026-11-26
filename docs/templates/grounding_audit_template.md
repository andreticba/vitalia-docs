<!-- grounding_audit_template.md | Vitalia Kit v0.0.1 | 25-11-2026 19:54(GMT-04:00) -->
**grounding_audit_template.md | Vitalia Kit v0.0.1 | 25-11-2026 19:54(GMT-04:00)**

<!-- GROUNDING_AUDIT.md | Vitalia Kit v0.0.1 | {timestamp} -->
**GROUNDING_AUDIT.md | Vitalia Kit v0.0.1 | {timestamp}**

# 🔎 Auditoria de Grounding P03 — {topic}

{badge_status}
{badge_total_urls}
{badge_verified_ratio}
{badge_cache_window}
{badge_hitl_gate}
{badge_timestamp}

> **Vitalia Kit v0.0.1 — Portão Determinístico de Evidências Externas (P03 / D18 / W3C PROV-O).**  
> **Projeto:** `{project_name}` · **Modo de Validação:** `{validation_mode}` · **Ledger Fonte:** [evidence.jsonl](./evidence.jsonl)  
> **Navegação:** [← Índice de Pesquisas](../README.md) · [🏠 Data Storage (Camada 3)](../../README.md)

---

## 📊 Resumo Executivo da Auditoria

<table>
  <thead>
    <tr>
      <th align="center">Fontes Auditadas</th>
      <th align="center">✅ Verificadas (HTTP 200)</th>
      <th align="center">⚠️ Unverified (D18)</th>
      <th align="center">❌ Inválidas / Expiradas</th>
      <th align="center">🌐 Não Catalogadas</th>
      <th align="center">🛡️ Escalação HITL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center"><strong>{total_urls}</strong></td>
      <td align="center"><strong>{verified_count}</strong></td>
      <td align="center"><strong>{unverified_count}</strong></td>
      <td align="center"><strong>{failed_count}</strong></td>
      <td align="center"><strong>{uncataloged_count}</strong></td>
      <td align="center"><strong>{hitl_status}</strong></td>
    </tr>
  </tbody>
</table>

---

## 🧭 Cadeia de Proveniência (P03 / P16)

<div align="center">

```mermaid
flowchart LR
{mermaid_provenance}
```

</div>

---

## 📑 Tabela de Proveniência e Snapshots

<table>
  <thead>
    <tr>
      <th align="center">#</th>
      <th align="left">Domínio</th>
      <th align="left">URL Auditada</th>
      <th align="center">HTTP</th>
      <th align="center">Snapshot Local</th>
      <th align="center">SHA-256</th>
      <th align="left">Coleta (GMT-04:00)</th>
      <th align="center">Status</th>
    </tr>
  </thead>
  <tbody>
{provenance_rows}
  </tbody>
</table>

---

## 🔍 Detalhamento Forense, Domínios & Portão D18

<details>
<summary><strong>📜 Clique para expandir Citações Forenses (<code>literal_quote</code>) e Redirecionamentos</strong></summary>

<br/>

| # | URL Solicitada (`url_requested`) | URL Final (`final_url`) | Citação Literal Verificada (`literal_quote`) | Quote Check |
| :---: | :--- | :--- | :--- | :---: |
{forensic_quotes_rows}

</details>

<details>
<summary><strong>⚠️ Clique para expandir Domínios Não Catalogados (<code>general_web</code> — Curadoria 1A)</strong></summary>

<br/>

> **Fluxo 1A:** Os domínios abaixo caíram em `general_web` durante a pesquisa. O `vitalia-researcher` **não** altera `grounding_domains.yaml`; a curadoria HITL (`global` / `local` / `rejected`) ocorre no encerramento via `/vitalia-session-consolidate` (`--action list-pending-domains`).

| Domínio Sugerido | URLs Coletadas | Motivo / Contexto | Status de Curadoria |
| :--- | :--- | :--- | :---: |
{uncataloged_domains_rows}

</details>

<details>
<summary><strong>🛡️ Clique para expandir Verificado vs. Não Verificado & Política D18</strong></summary>

<br/>

- **Política Ativa (D18):** `Retry + Degrade` (default) · `HITL Escalation` em `clinical_safety` / `regulations_compliance` · `Hard Block` em `--strict`.
- **Janela de Validade (Cache):** `{cache_validity_days}` dias (`GMT-04:00`).

#### ✅ Fontes Verificadas
{verified_list}

#### ⚠️ Fontes Degradadas (`UNVERIFIED`) ou com Falha
{unverified_or_failed_list}

#### 🛡️ Alertas de Escalação HITL (Domínios Críticos)
{hitl_alerts_list}

</details>

---

<sub>Relatório gerado automaticamente por <code>GroundingAuditRenderer</code> (<code>rendering/grounding_audit_renderer.py</code>) a partir de <a href="./evidence.jsonl"><code>evidence.jsonl</code></a> validado por <code>vitalia_grounding_validator.py --mode evidence</code>.</sub>

