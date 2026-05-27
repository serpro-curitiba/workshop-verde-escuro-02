<!-- markdownlint-disable MD013 MD025 MD026 MD028 MD029 MD034 MD040 MD051 MD060 -->

# Glossário do SIFAP Legado

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **glossary**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Este documento totalmente preenchido com os dados reais do legado SIFAP
> 2. Rastreabilidade para `01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs)
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Preencha esta tabela com todos os termos, abreviações e siglas encontrados no código Natural/Adabas.
> **Meta: no mínimo 30 termos.**

## Por que isso importa

Sistemas legados têm vocabulário próprio que ninguém documenta em lugar nenhum — só está no nome das variáveis. Se o time do Estágio 2 não souber o que `DSCT`, `BENF`, `PE` ou `CTC` significam, vai escrever uma spec sobre o que ele _acha_ que isso significa. Glossário é o que evita esse desencontro.

## Como preencher

- **Termo**: a abreviação ou sigla exatamente como aparece no código
- **Expansão**: o significado completo do termo
- **Programa**: em qual arquivo `.NSN` ou `.ddm` o termo foi encontrado
- **Contexto**: breve explicação de como/onde o termo é usado

## Dica de extração

Prompt útil no Copilot Chat (cole o conteúdo de 2–3 arquivos `.NSN` no chat antes):

> _"Liste todas as abreviações e siglas usadas neste código Natural. Para cada uma, sugira a expansão e marque com 'CONFIRMADO' ou 'HIPÓTESE'."_

## Termos encontrados

| #   | Termo | Expansão | Programa | Contexto |
| --- | ----- | -------- | -------- | -------- |
| 1   | `SIFAP` | Sistema de Fiscalização e Administração de Pagamentos | Todos os `.NSN` e `.ddm` (cabeçalho) | Nome oficial do sistema legado de gestão de benefícios sociais; ativo desde 1997 |
| 2   | `BENF` / `BENEF` | Beneficiário | `CADBENEF.NSN`, `VALBENEF.NSN`, `CALCBENF.NSN`, `BENEFICIARIO.ddm` | Pessoa física cadastrada para receber benefício social; base principal do sistema (~4,2 milhões de registros) |
| 3   | `DSCT` | Desconto | `CALCDSCT.NSN`, `PAGAMENTO.ddm` campo `CA` | Dedução aplicada sobre valor bruto. Tipos: `J`=Judicial, `I`=Imposto, `P`=Pensão, `S`=Sindical, `A`=Administrativo, `C`=Contribuição |
| 4   | `PGTO` / `PGT` | Pagamento | `BATCHPGT.NSN`, `RELPGT.NSN`, `PAGAMENTO.ddm` | Registro mensal de valor a pagar ao beneficiário; arquivo Adabas 160 |
| 5   | `PROG` | Programa Social | `CADPROG.NSN`, `PROGRAMA-SOCIAL.ddm` | Programa social cadastrado (ex.: PBF, BPC, PETI); tabela paramétrica, arquivo Adabas 155 (~45 programas ativos) |
| 6   | `CICLO` | Ciclo de Processamento | `PAGAMENTO.ddm` campo `AF`, `AUDITORIA.ddm` campo `FA` | Identificador da rodada batch mensal de geração de pagamentos |
| 7   | `COMP` / `COMPETENCIA` | Competência (mês de referência) | `CALCBENF.NSN`, `BATCHPGT.NSN`, `PAGAMENTO.ddm` campo `AE` | Mês/ano de referência do pagamento no formato `AAAAMM` |
| 8   | `DDM` | Data Definition Module | `*.ddm` (Adabas) | Metadados Adabas equivalentes a um schema de tabela; define campos, tipos e descritores |
| 9   | `DE` | Descriptor | `*.ddm` (cabeçalhos de campo) | Marcação Adabas que indica campo indexado para busca rápida |
| 10  | `PE` | Periodic Group | `BENEFICIARIO.ddm` campo `DA` (`GRP-DEPENDENTE`), `PAGAMENTO.ddm` campo `CA` | Grupo periódico Adabas — array de sub-registros (até N ocorrências); equivale a tabela relacionada 1:N |
| 11  | `MU` | Multiple-Value Field | `PROGRAMA-SOCIAL.ddm` campo `EA`, `AUDITORIA.ddm` campo `DB` | Campo Adabas que armazena múltiplos valores escalares (array simples) |
| 12  | `FNR` | File Number | Cabeçalhos de todos os `.ddm` | Número do arquivo Adabas (ex.: 150=Beneficiário, 151=Programa, 152=Pagamento, 153=Auditoria) |
| 13  | `DBID` | Database ID | Cabeçalhos de todos os `.ddm` | Identificador da instância Adabas (`DBID: 57` em todo o SIFAP) |
| 14  | `ISN` | Internal Sequence Number | `BENEFICIARIO.ddm` campo `AA` (`NUM-INSCRICAO`) | Identificador interno único do Adabas para cada registro; usado como matrícula alternativa |
| 15  | `NIS` | Número de Identificação Social | `BENEFICIARIO.ddm` campo `NIS`, `CALCBENF.NSN` | Identificador do CadÚnico vinculado ao beneficiário |
| 16  | `FATOR-K` | Fator de Correção Especial | `CADPROG.NSN#L87`, `PROGRAMA-SOCIAL.ddm` campo `BG` | Constante misteriosa (`1.00 + #FATOR-REAJ * 0.347215`) aplicada ao valor-base sem documentação — ver MYS-003 |
| 17  | `FATOR-REG` | Fator Regional | `CALCBENF.NSN`, `BATCHPGT.NSN`, tabela `#TAB-REG(27)` | Multiplicador do valor-base por UF; tabela com 27 entradas hardcoded (~1.00 a 1.40) |
| 18  | `FATOR-FAM` | Fator Familiar | `CALCBENF.NSN` | Multiplicador progressivo por número de dependentes (1.0 a ~1.20) |
| 19  | `FATOR-RND` | Fator de Renda | `CALCBENF.NSN`, tabela `#FATOR-FAIXA(5)` | Multiplicador inverso por faixa de renda familiar; menor renda = maior fator |
| 20  | `FATOR-IDADE` | Fator de Idade | `CALCBENF.NSN` | Multiplicador por faixa etária: <18→1.05, 18-59→1.00, 60-64→1.10, ≥65→1.15 |
| 21  | `STATUS` | Situação do Beneficiário | `CADBENEF.NSN`, `BENEFICIARIO.ddm` campo `CE` | Estados: `A`=Ativo, `S`=Suspenso, `C`=Cancelado, `I`=Inativo, `D`=Desligado |
| 22  | `TIPO-PGTO` | Tipo de Pagamento | `CALCBENF.NSN`, `PAGAMENTO.ddm` | `N`=Normal, `D`=Décimo (13º), `T`=Terceiro (terceiros) |
| 23  | `TIPO-PROG` | Tipo de Programa | `CADPROG.NSN`, `PROGRAMA-SOCIAL.ddm` campo `AD` | `A`=Assistencial, `P`=Previdenciário, `T`=Trabalho |
| 24  | `COD-REGIAO` | Código da Região | `BENEFICIARIO.ddm` campo `BJ`, `VALELEG.NSN` | Regiões 01–25 (UFs); valor `99` = "ESPECIAL/INTERNACIONAL/DIPLOMATICO" — ver MYS-008 |
| 25  | `DOCS-OK` | Documentação Validada | `BENEFICIARIO.ddm` campo `DOCUMENTOS-OK`, `VALELEG.NSN` | Flag `S`/`N` indicando se documentação do beneficiário está completa |
| 26  | `CNAB 240` | Centro Nacional de Automação Bancária — layout 240 | `BATCHCON.NSN#L105-L130` | Layout de arquivo bancário de 240 posições; usado para envio/retorno de pagamentos ao Banco do Brasil |
| 27  | `CPF` | Cadastro de Pessoas Físicas | Todos os programas; `BENEFICIARIO.ddm` campo `AB` | Identificador único do beneficiário (11 dígitos); validado por MOD-11 em `VALDOCS.NSN` |
| 28  | `MOD-11` | Algoritmo Módulo 11 | `VALDOCS.NSN#L107-L141` | Algoritmo de verificação de dígito do CPF; computa DV1 e DV2 |
| 29  | `DOC-ESPECIAL` | Documento Especial (backdoor) | `VALDOCS.NSN` subroutine `CHECK-DOC-ESPECIAL` | Subroutine que aceita CPFs com prefixos `000`, `001`, `002`, `010`, `011`, `099`, `100`, `999` sem validação — Easter Egg 2 |
| 30  | `FATOR-REAJ` | Fator de Reajuste Anual | `CADPROG.NSN`, `PROGRAMA-SOCIAL.ddm` campo `BE` | Percentual de reajuste anual aplicado ao valor-base do programa (ex.: 5.75) |
| 31  | `VLR-BASE` | Valor Base | `PROGRAMA-SOCIAL.ddm` campo `BA`, `CALCBENF.NSN` | Valor mensal de referência do programa antes da aplicação dos fatores |
| 32  | `VLR-BRUTO` / `VLR-LIQ` | Valor Bruto / Valor Líquido | `PAGAMENTO.ddm`, `CALCBENF.NSN` | Bruto = antes de descontos; Líquido = após descontos (`VLR-BRUTO - VLR-DESCONTO`) |
| 33  | `ABONO NATALINO` | Abono de fim de ano (15%) | `CALCBENF.NSN#L260-L268` | Adicional de 15% sobre o valor mensal aplicado em dezembro apenas para programas tipo `A` — ver MYS-004 |
| 34  | `BATCH` | Processamento em Lote | `BATCHPGT.NSN`, `BATCHCON.NSN`, `BATCHREL.NSN` | Jobs noturnos executados no 1º dia útil do mês; geração de pagamentos, conciliação bancária e relatórios |
| 35  | `WORK FILE` | Arquivo de Trabalho | `BATCHCON.NSN#L105` (`DEFINE WORK FILE 1`) | Construção Natural para leitura sequencial de arquivos externos (ex.: retorno CNAB) |
| 36  | `FIND` / `READ` / `STORE` / `UPDATE` | Comandos Adabas via Natural | Todos os `.NSN` | DML do Natural: `FIND` (busca indexada), `READ` (varredura), `STORE` (insert), `UPDATE` (update do registro corrente) |
| 37  | `IN-TCU 63/2010` | Instrução Normativa TCU 63/2010 | `AUDITORIA.ddm` cabeçalho | Base legal que exige trilha de auditoria imutável com retenção mínima de 10 anos |
| 38  | `SENARC` | Secretaria Nacional de Renda de Cidadania | `PROGRAMA-SOCIAL.ddm` (campo `BG` `FATOR-K`) | Órgão do MDS citado como solicitante da inclusão do FATOR-K (sem chamado válido) |
| 39  | `MDS/MDAS` | Ministério do Desenvolvimento Social (e Agrário) | `BENEFICIARIO.ddm` (Port. 847/2003), `PROGRAMA-SOCIAL.ddm` | Órgão controlador do sistema; mudou de nome em reorganizações ministeriais |
| 40  | `BANCO REAL` | Banco Real (extinto) | `BATCHCON.NSN#L207-L223` (código comentado) | Banco código `356`; integração descontinuada — adquirido pelo Santander em 2007 — Easter Egg 3 |

> Adicione mais linhas conforme necessário. Não se limite a 30!

## Exemplo de linha bem preenchida

| #   | Termo  | Expansão | Programa                        | Contexto                                                                                                         |
| --- | ------ | -------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 1   | `DSCT` | Desconto | `CALCDSCT.NSN`, `PAGAMENTO.ddm` | Tipo de dedução aplicada sobre valor bruto do pagamento. Tipos: 'J' (judicial), 'I' (imposto), 'T' (trabalhista) |

## Observações

- **Convenção de prefixo Adabas (DDM)**: pares de letras (AA, AB, BA, CA…) são códigos curtos de campo (legado da limitação de 2 caracteres do Adabas clássico); o nome longo (`NUM-CPF`, `NOME-COMPLETO`) é apenas alias para legibilidade.
- **Convenção Natural**: variáveis locais começam com `#` (ex.: `#CPF`, `#VLR-BENF`); views de DDM têm sufixo `-V` (ex.: `BENEFICIARIO-V`).
- **Convenção de nomes de programa**: 8 caracteres, sem separador — `CALC`+`BENF`, `BATCH`+`PGT`, `VAL`+`ELEG`. Restrição do mainframe original.
- **Termos ambíguos que precisam de validação com especialista**:
  - `FATOR-K` (origem da constante `0.347215` desconhecida — MYS-003)
  - `COD-REGIAO = 99` (significado de "ESPECIAL" não documentado — MYS-008)
  - `EX` (Exclusão) — filtrada dos relatórios sem justificativa formal (MYS-010)

---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="GUIDE.md"><strong>GUIDE do Estágio 1</strong></a><br/>
<sub>Passo a passo do estágio.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="business-rules-catalog.md"><strong>business-rules-catalog.md</strong></a><br/>
<sub>Catálogo de regras.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="README.md">Voltar ao Kit PT-BR</a></sub>

