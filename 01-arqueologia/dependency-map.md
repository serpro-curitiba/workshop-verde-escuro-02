<!-- markdownlint-disable MD013 MD025 MD026 MD028 MD029 MD034 MD040 MD051 MD060 -->

# Mapa de Dependências — SIFAP Legado

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **dependency-map**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Este documento totalmente preenchido com os dados reais do legado SIFAP
> 2. Rastreabilidade para `01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs)
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Use diagramas Mermaid para mapear as dependências entre programas Natural e DDMs Adabas.
> O objetivo é visualizar "quem chama quem" e "quem lê/escreve o quê".

## Como descobrir dependências

- `grep` por `CALLNAT` nos 15 `.NSN` → **resultado: ZERO ocorrências**. Não há chamadas inter-programa.
- `grep` por `VIEW OF` → mapeia que DDM cada programa abre.
- `grep` por `READ`, `FIND`, `HISTOGRAM` → leitura.
- `grep` por `STORE`, `UPDATE`, `DELETE` → escrita.
- `grep` por `PERFORM` → subroutines **internas** (dentro do próprio `.NSN`), não dependência externa.
- `grep` por `DEFINE WORK FILE` / `INPUT USING MAP` → entradas externas (CNAB, terminal 3270).

## Descoberta Crítica — Acoplamento por Dados (não por código)

> **Resultado da varredura:** os 15 programas `.NSN` **não se chamam entre si**. Não existe `CALLNAT` no legado SIFAP.
>
> Cada programa é **monolítico**: declara suas próprias `VIEW OF`, abre e fecha sua própria transação Adabas. Subroutines (`PERFORM`) existem somente **dentro** do mesmo arquivo.
>
> **Onde está a dependência então?** Nos 4 DDMs Adabas compartilhados. Se duas pessoas mudarem `CALCBENF.NSN` e `BATCHPGT.NSN` no mesmo dia, o "chicote estala" porque ambos gravam em `PAGAMENTO` — mas o compilador Natural não detecta. Isto é **acoplamento implícito via banco**, o pesadelo clássico do monolito legado.

## Diagrama de Dependências — 15 Programas × 4 DDMs

```mermaid
flowchart LR
  classDef online fill:#E5F6FD,stroke:#00A4EF,color:#0A0A0A
  classDef batch  fill:#FFF7E0,stroke:#FFB900,color:#0A0A0A
  classDef calc   fill:#F1F8E3,stroke:#7FBA00,color:#0A0A0A
  classDef val    fill:#FDE7E9,stroke:#F25022,color:#0A0A0A
  classDef rel    fill:#EFE3F8,stroke:#7B68EE,color:#0A0A0A
  classDef ddm    fill:#1A1A1A,stroke:#FFFFFF,color:#FFFFFF
  classDef ext    fill:#737373,stroke:#0A0A0A,color:#FFFFFF

  subgraph ONLINE["Online (terminal 3270)"]
    CADBENEF["CADBENEF.NSN<br/>Cadastro Beneficiário"]:::online
    CADDEPEND["CADDEPEND.NSN<br/>Cadastro Dependentes"]:::online
    CADPROG["CADPROG.NSN<br/>Cadastro Programa"]:::online
    CONSBENF["CONSBENF.NSN<br/>Consulta Beneficiário"]:::online
  end

  subgraph BATCH["Batch (1º dia útil/mês)"]
    BATCHPGT["BATCHPGT.NSN<br/>Geração Pagamentos"]:::batch
    BATCHCON["BATCHCON.NSN<br/>Conciliação CNAB"]:::batch
    BATCHREL["BATCHREL.NSN<br/>Relatórios mensais"]:::batch
  end

  subgraph CALC["Cálculo / Atualização"]
    CALCBENF["CALCBENF.NSN<br/>Cálculo benefício"]:::calc
    CALCCORR["CALCCORR.NSN<br/>Correção monetária"]:::calc
    CALCDSCT["CALCDSCT.NSN<br/>Descontos"]:::calc
  end

  subgraph VAL["Validação"]
    VALBENEF["VALBENEF.NSN<br/>Valida cadastro"]:::val
    VALDOCS["VALDOCS.NSN<br/>Valida CPF/RG"]:::val
    VALELEG["VALELEG.NSN<br/>Valida elegibilidade"]:::val
  end

  subgraph REL["Relatórios"]
    RELPGT["RELPGT.NSN<br/>Relatório pagamentos"]:::rel
    RELAUDIT["RELAUDIT.NSN<br/>Relatório auditoria"]:::rel
  end

  subgraph DDMS["Adabas DBID 57"]
    BENEF[("BENEFICIARIO<br/>FNR 150")]:::ddm
    PROG[("PROGRAMA-SOCIAL<br/>FNR 151")]:::ddm
    PGTO[("PAGAMENTO<br/>FNR 152")]:::ddm
    AUDIT[("AUDITORIA<br/>FNR 153")]:::ddm
  end

  subgraph EXT["Entradas externas"]
    CNAB["CNAB 240<br/>retorno Banco do Brasil"]:::ext
    MAP3270["Maps 3270<br/>CONSBENF-M01 etc."]:::ext
    REAL["[DEAD CODE] CNAB Banco Real"]:::ext
  end

  %% BENEFICIARIO
  CADBENEF -->|STORE/UPDATE| BENEF
  CADDEPEND -->|FIND/UPDATE| BENEF
  VALBENEF -->|FIND| BENEF
  VALDOCS -->|FIND| BENEF
  VALELEG -->|FIND| BENEF
  CONSBENF -->|FIND| BENEF
  CALCBENF -->|FIND| BENEF
  BATCHPGT -->|READ| BENEF
  BATCHREL -->|FIND| BENEF
  RELPGT -->|FIND| BENEF

  %% PROGRAMA-SOCIAL
  CADPROG -->|STORE| PROG
  CALCBENF -->|FIND| PROG
  BATCHPGT -->|FIND| PROG
  VALELEG -->|FIND| PROG

  %% PAGAMENTO
  CALCBENF -->|STORE| PGTO
  CALCCORR -->|FIND/UPDATE| PGTO
  CALCDSCT -->|FIND/UPDATE| PGTO
  BATCHPGT -->|STORE| PGTO
  BATCHCON -->|FIND/UPDATE| PGTO
  BATCHREL -->|READ| PGTO
  CONSBENF -->|READ| PGTO
  RELPGT -->|READ| PGTO

  %% AUDITORIA
  BATCHCON -->|STORE| AUDIT
  RELAUDIT -->|READ| AUDIT

  %% Externas
  MAP3270 --> CONSBENF
  MAP3270 -.-> CADBENEF
  MAP3270 -.-> CADDEPEND
  MAP3270 -.-> CADPROG
  CNAB --> BATCHCON
  REAL -.->|comentado L207-223| BATCHCON
```

## Diagrama de Fluxo de Dados (pipeline mensal)

```mermaid
flowchart TD
  classDef step fill:#FFF7E0,stroke:#FFB900,color:#0A0A0A
  classDef ddm  fill:#1A1A1A,stroke:#FFFFFF,color:#FFFFFF

  S0["1. Operador cadastra/altera<br/>CADBENEF · CADDEPEND · CADPROG"]:::step
  S1["2. Validação prévia<br/>VALBENEF · VALDOCS · VALELEG"]:::step
  S2["3. Cálculo do benefício<br/>CALCBENF (fatores K, REG, FAM, RND, IDADE)"]:::step
  S3["4. Geração batch mensal<br/>BATCHPGT (1º dia útil)"]:::step
  S4["5. Descontos / Correção<br/>CALCDSCT · CALCCORR"]:::step
  S5["6. Envio ao banco<br/>arquivo CNAB 240 → Banco do Brasil"]:::step
  S6["7. Conciliação retorno<br/>BATCHCON lê retorno CNAB"]:::step
  S7["8. Relatórios mensais<br/>BATCHREL · RELPGT · RELAUDIT"]:::step
  S8["9. Consulta operacional<br/>CONSBENF (terminal 3270)"]:::step

  BENEF[("BENEFICIARIO")]:::ddm
  PROG[("PROGRAMA-SOCIAL")]:::ddm
  PGTO[("PAGAMENTO")]:::ddm
  AUDIT[("AUDITORIA")]:::ddm

  S0 --> BENEF & PROG
  S1 --> BENEF
  BENEF --> S2
  PROG --> S2
  S2 --> PGTO
  PGTO --> S3
  S3 --> PGTO
  PGTO --> S4 --> PGTO
  PGTO --> S5
  S5 -.-> S6
  S6 --> PGTO
  S6 --> AUDIT
  PGTO --> S7
  AUDIT --> S7
  BENEF --> S8
  PGTO --> S8
```

## Tabela de Dependências (todos os 15 programas)

> **Coluna "Chama":** nenhuma chamada inter-programa via `CALLNAT` foi encontrada. A coluna lista subroutines **internas** (`PERFORM`) relevantes, que ajudam a entender a estrutura do programa.

| #   | Programa         | Chama (PERFORM interno)                                 | Lê DDMs (FIND/READ)              | Escreve DDMs (STORE/UPDATE)      | Entradas externas                | Observações                                                                          |
| --- | ---------------- | ------------------------------------------------------- | -------------------------------- | -------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------ |
| 1   | `CADBENEF.NSN`   | `VALIDA-CPF` (L112)                                     | BENEFICIARIO                     | BENEFICIARIO                     | Map 3270                         | Cadastro online de titulares                                                         |
| 2   | `CADDEPEND.NSN`  | —                                                       | BENEFICIARIO                     | BENEFICIARIO (PE `GRP-DEPENDENTE`) | Map 3270                       | Manipula sub-registros periódicos                                                    |
| 3   | `CADPROG.NSN`    | `CONSULTA-PROG` (L57)                                   | PROGRAMA-SOCIAL                  | PROGRAMA-SOCIAL                  | Map 3270                         | Cadastra `FATOR-K` sem rastreabilidade — MYS-003                                     |
| 4   | `CONSBENF.NSN`   | `MASCARA-CPF` (L107)                                    | BENEFICIARIO, PAGAMENTO          | —                                | `INPUT USING MAP 'CONSBENF-M01'` | Consulta read-only                                                                   |
| 5   | `VALBENEF.NSN`   | `VALIDA-CPF-COMPLETO`, `VALIDA-DATA`, `VALIDA-NOME`     | BENEFICIARIO                     | —                                | —                                | Validação síncrona de cadastro                                                       |
| 6   | `VALDOCS.NSN`    | `VALIDA-CPF-DOC`, `VALIDA-RG`, `CHECK-DOC-ESPECIAL`     | BENEFICIARIO                     | —                                | —                                | Contém backdoor de CPFs especiais — Easter Egg 2                                     |
| 7   | `VALELEG.NSN`    | `VERIF-ELEG-ESPECIFICA` (L207)                          | BENEFICIARIO, PROGRAMA-SOCIAL    | —                                | —                                | Aplica regra `COD-REGIAO = 99` — MYS-008                                             |
| 8   | `CALCBENF.NSN`   | (cálculo inline dos 5 fatores)                          | BENEFICIARIO, PROGRAMA-SOCIAL    | PAGAMENTO (L286)                 | —                                | Núcleo de cálculo; produz `PAGAMENTO` para os batches; abono natalino (L260) — MYS-004 |
| 9   | `CALCCORR.NSN`   | `CALC-INDICE-ACUM` (L149)                               | PAGAMENTO                        | PAGAMENTO (L162)                 | Tabela IPCA hardcoded            | Correção monetária; UPDATE do mesmo registro lido                                    |
| 10  | `CALCDSCT.NSN`   | `CALC-CONTRIB-SOCIAL` (L99)                             | PAGAMENTO                        | PAGAMENTO (L181)                 | —                                | Aplica os 6 tipos de desconto                                                        |
| 11  | `BATCHPGT.NSN`   | `DET-FAIXA-RENDA-BATCH` (L262)                          | BENEFICIARIO, PROGRAMA-SOCIAL    | PAGAMENTO (L335)                 | JCL noturno (1º dia útil)        | Batch principal; lê BENEFICIARIO inteiro                                             |
| 12  | `BATCHCON.NSN`   | `GRAVA-AUDITORIA-DIVERG` (L167), `GRAVA-AUDITORIA-CONC` (L201) | PAGAMENTO, AUDITORIA, `WORK FILE 1` | PAGAMENTO, AUDITORIA      | `#ARQ-RETORNO` (CNAB 240)        | Conciliação bancária; código Banco Real comentado L207-223 — Easter Egg 3            |
| 13  | `BATCHREL.NSN`   | `IMPRIME-CABECALHO` (L172)                              | PAGAMENTO, BENEFICIARIO          | —                                | JCL noturno                      | Relatórios consolidados                                                              |
| 14  | `RELPGT.NSN`     | `IMPRIME-SUBTOTAL` (L94, L174), `IMPRIME-CABECALHO` (L145) | PAGAMENTO, BENEFICIARIO       | —                                | Parâmetros via PDA               | Filtra ações `EX` sem justificativa — MYS-010                                         |
| 15  | `RELAUDIT.NSN`   | `IMPRIME-CAB-AUDIT` (L165)                              | AUDITORIA                        | —                                | —                                | Único leitor da trilha de auditoria                                                  |

## Matriz de Acoplamento por DDM (quem mexe em quê)

> **Leitura:** cada coluna mostra todos os programas que tocam a DDM. Qualquer mudança de schema afeta TODOS os programas listados.

| DDM (FNR)                 | Escritores (STORE/UPDATE)                                            | Leitores (FIND/READ)                                                         | Total |
| ------------------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ----- |
| `BENEFICIARIO` (150)      | CADBENEF, CADDEPEND                                                  | VALBENEF, VALDOCS, VALELEG, CONSBENF, CALCBENF, BATCHPGT, BATCHREL, RELPGT   | 10    |
| `PROGRAMA-SOCIAL` (151)   | CADPROG                                                              | CALCBENF, BATCHPGT, VALELEG                                                  | 4     |
| `PAGAMENTO` (152)         | CALCBENF, CALCCORR, CALCDSCT, BATCHPGT, BATCHCON                     | BATCHREL, CONSBENF, RELPGT                                                   | 8     |
| `AUDITORIA` (153)         | BATCHCON                                                              | RELAUDIT                                                                     | 2     |

**Hot spots:**

- `BENEFICIARIO` é tocado por **10 dos 15 programas** — qualquer alteração de campo é alto risco.
- `PAGAMENTO` tem **5 escritores diferentes** sem coordenação por código — só o banco arbitra a ordem (race condition latente no batch noturno).
- `AUDITORIA` tem **1 único escritor** (BATCHCON) — isto viola IN-TCU 63/2010, que exige que **toda** mudança em PAGAMENTO/BENEFICIARIO gere log.

## Dependências Circulares

- **Código (CALLNAT):** nenhuma — não existe chamada inter-programa.
- **Dados:** ciclo de auto-update em `PAGAMENTO` por `CALCCORR.NSN` (L13 lê → L162 grava o mesmo registro) e `CALCDSCT.NSN` (L13 lê → L181 grava). Não é ciclo entre programas, mas é leitura-escrita do mesmo registro Adabas no mesmo programa — requer atenção na migração para JPA (lock otimista).

## Programas Órfãos / Pontos de Entrada

Como não há `CALLNAT`, **todos os 15 programas são pontos de entrada** (online via 3270 ou batch via JCL). Não há código morto detectável por análise estática de chamadas — mas há:

- **Código morto interno:** `BATCHCON.NSN` L207-223 (integração Banco Real comentada) — Easter Egg 3.
- **Subroutine subutilizada:** `CHECK-DOC-ESPECIAL` em `VALDOCS.NSN` — backdoor de CPFs (Easter Egg 2).

## Implicação para a Migração (Estágio 2)

1. **Bounded contexts ≠ programas.** Como o acoplamento é por dados, os contextos devem ser desenhados ao redor das **DDMs** (Beneficiário, Programa, Pagamento, Auditoria), não dos `.NSN`.
2. **`PAGAMENTO` é o agregado mais disputado** — provavelmente o agregado raiz do bounded context "Pagamento" no novo design.
3. **Strangler Fig:** comece pelos programas que **só leem** (CONSBENF, RELPGT, RELAUDIT, BATCHREL, VAL*) — risco mínimo. Deixe `BATCHPGT` + `CALCBENF` (núcleo de escrita) por último.
4. **Trilha de auditoria precisa virar transversal** — todo bounded context deve emitir evento de domínio captado por um serviço de auditoria, não delegar para um único batch.

---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="business-rules-catalog.md"><strong>business-rules-catalog.md</strong></a><br/>
<sub>Catálogo de regras.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="discovery-report.md"><strong>discovery-report.md</strong></a><br/>
<sub>Síntese final.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="README.md">Voltar ao Kit PT-BR</a></sub>

