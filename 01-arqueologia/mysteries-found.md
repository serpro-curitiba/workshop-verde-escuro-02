<!-- markdownlint-disable MD013 MD025 MD026 MD028 MD029 MD034 MD040 MD051 MD060 -->

# Mistérios Encontrados — SIFAP Legado

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **mysteries-found**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Este documento totalmente preenchido com os dados reais do legado SIFAP
> 2. Rastreabilidade para `01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs)
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Registre aqui toda lógica, comportamento ou código que o time não conseguiu explicar.
> "Mistérios" são trechos de código sem documentação, com lógica não-óbvia ou que parecem workarounds.
>
> **Cota mínima para passar pelo portão do Estágio 2:** 5 mistérios documentados.

## O que conta como "mistério"?

- Código que faz algo inesperado sem comentário explicando por quê
- Valores hardcoded sem explicação (números mágicos)
- Lógica condicional que parece um workaround ou gambiarra
- Campos no DDM que não são usados por nenhum programa
- Programas que existem mas não são chamados por ninguém
- Comportamento diferente entre o que a documentação diz e o que o código faz
- Easter eggs deixados pelos desenvolvedores originais

## Níveis de Confiança

| Nível     | Significado                                         |
| --------- | --------------------------------------------------- |
| **ALTA**  | Temos certeza de que há algo estranho aqui          |
| **MÉDIA** | Parece suspeito, mas pode ter explicação            |
| **BAIXA** | Pode ser intencional, mas não conseguimos confirmar |

## Mistérios Catalogados

| ID      | Descrição | Onde Encontrado | Impacto Potencial | Confiança |
| ------- | --------- | --------------- | ----------------- | --------- |
| MYS-001 | `STATUS` é mudado silenciosamente | `CADBENEF.nsn` | qualquer fluxo de onboarding no sistema moderno que não reimplementar essa regra de idade vai ativar automaticamente beneficiários que o legado manteria suspensos — pagamentos indevidos para uma faixa etária inteira. | **ALTA** |
| MYS-002 | Limite de dependentes hardcoded como `> 5` no programa, mas o DDM define capacidade de 10 ocorrências | `CADDEPEND.NSN#L63` vs `BENEFICIARIO.ddm` campo `DA` | Beneficiários com 6–10 dependentes têm dados truncados ou rejeitados silenciosamente | **ALTA** |
| MYS-003 | Constante `0.347215` usada no cálculo de `#FATOR-K` sem nenhuma documentação — origem desconhecida; o próprio campo `FATOR-K` no DDM é marcado `NAO DOCUMENTADO` | `CADPROG.NSN#L87` + `PROGRAMA-SOCIAL.ddm` campo `BG` | Reprodução incorreta do cálculo de valor-base em qualquer migração | **ALTA** |
| MYS-004 | Em dezembro (`#MES = 12`) o cálculo muda completamente: fórmula do 13º omite `#FATOR-FAM` e `#FATOR-RND`; programas tipo `A` ganham abono natalino de 15% sem documentação da regra | `CALCBENF.NSN#L253-L280` | 13º calculado com base diferente do mensal — famílias numerosas e faixas de renda recebem 13º desproporcional; risco de reprocessamento massivo | **ALTA** |
| MYS-005 | Truncação sistemática via variável inteira `(N11)`: padrão `× 100 → N11 → ÷ 100` é aplicado 4–6× por pagamento em vez de arredondamento — beneficiários sempre recebem menos do que o valor matematicamente correto | `CALCBENF.NSN#L231-L233` (e L245, L253, L271) + `CALCDSCT.NSN#L103` e `L174` | Perda sistemática de fração de centavo por pagamento; em 4,2 milhões de beneficiários o desvio acumulado é relevante e auditável | **ALTA** |
| MYS-006 | Desconto tipo `'J'` (Judicial) ignora o teto de 30% do bruto aplicado a todos os demais — comentário `JUDICIAL NAO TEM TETO` sem base legal citada; pode zerar o líquido do beneficiário sem nenhum piso | `CALCDSCT.NSN#L128-L131` e `L163-L167` | Beneficiário pode ficar com VLR-LIQUIDO = 0; sistema moderno sem essa regra aplicará o teto de 30% a judiciais, bloqueando ordens judiciais vigentes | **ALTA** |
| MYS-007 | `CPF-DEPENDENTE` aceita `00000000000` como valor intencional documentado | `BENEFICIARIO.ddm` campo `DB` | Validação de CPF no sistema moderno precisa tratar esse caso especial | **MÉDIA** |
| MYS-008 | Beneficiários com `COD-REGIAO = 99` ("INTERNACIONAL/DIPLOMATICO") pulam 100% das verificações de elegibilidade via `ESCAPE ROUTINE` imediato — status, idade, renda, documentação e tipo de programa, tudo ignorado | `VALELEG.NSN#L98-L103` | Backdoor de elegibilidade: qualquer beneficiário marcado com região 99 é aprovado incondicionalmente | **ALTA** |
| MYS-009 | Processamento batch ordena por CPF ascendente — escolha feita como "otimização de 1999" para índice Adabas; a ordem virou dependência estrutural de BATCHCON (conciliação CNAB), BATCHREL (relatórios) e da própria detecção de duplicatas por adjacência | `BATCHPGT.NSN#L178-L182` | Mudar a ordem de processamento quebra 3 programas downstream; a dependência não está documentada em nenhum deles, apenas na nota interna do BATCHPGT | **ALTA** |
| MYS-010 | Eventos de ação `'EX'` (Exclusão) são sistematicamente removidos do relatório de auditoria por filtro hardcoded; não há contador dedicado, não aparecem no sumário e nem mesmo o filtro do usuário `ACAO-FILTRO='EX'` consegue exibi-los — adicionado em 2014 como "LIMPEZA RELATORIO" | `RELAUDIT.NSN#L99-L103` | Viola IN-TCU 63/2010: exclusões não rastreavies em sistema de benefícios sociais com 4,2 milhões de registros | **ALTA** |


## Detalhamento dos Mistérios


### MYS-001: Status do beneficiário alterado silenciosamente por critério de idade

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN#L157-L169`
- **Trecho de código**:

```natural
* CALC IDADE BENEF
COMPUTE #IDADE = #ANO-ATUAL - #ANO-NASC
*
* DEF STATUS INICIAL
IF #OPER = 'I'
  MOVE 'A' TO #STATUS
END-IF
*
* AJUSTE P/ BENEFICIARIOS ACIMA DE 75 ANOS
IF #IDADE > 75
  MOVE 'S' TO #STATUS
END-IF
```

- **O que esperávamos**: status definido pelo operador ou mensagem de aviso antes de gravar — a tela retorna "BENEFICIARIO INCLUIDO COM SUCESSO" independentemente do resultado
- **O que o código faz**: define STATUS = 'A' para qualquer inclusão, depois sobrescreve para 'S' (Suspenso) se #IDADE > 75, sem WRITE, sem evento de auditoria, sem aviso; afeta também OPER = 'A' pela mesma variável #STATUS gravada na linha 209
- **Hipótese do time**: regra adicionada em 10/01/2011 por Jose Ferreira (AJUSTE STATUS IDOSO) — provavelmente exige revisão manual de idosos antes da ativação, mas foi implementada como override silencioso em vez de fluxo de aprovação
- **Risco se ignorarmos**: sistema moderno ativará automaticamente beneficiários com mais de 75 anos gerando pagamentos indevidos; registros com STATUS = 'S' por esse motivo não são distinguíveis na base de dados de suspensos por outras razões


### MYS-002: Limite de dependentes hardcoded contradiz capacidade do DDM

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/CADDEPEND.NSN#L63-L66` vs `01-arqueologia/legado-sifap/adabas-ddms/BENEFICIARIO.ddm#L55-L57`
- **Trecho de código**:

```natural
* LOOP DE INCLUSAO DE DEPENDENTES
REPEAT
  IF #NUM-DEP > 5
    WRITE 'LIMITE DE DEPENDENTES ATINGIDO'
    ESCAPE BOTTOM
  END-IF
```

DDM correspondente (`BENEFICIARIO.ddm`):

```
* --- DEPENDENTES (GRUPO PERIODICO - MAX 10 OCORRENCIAS) ---
  1  DA  GRP-DEPENDENTE        PE        -    10     GRUPO PERIODICO
```

- **O que esperávamos**: o programa respeitar a capacidade máxima declarada no DDM — 10 ocorrências no grupo periódico `GRP-DEPENDENTE` (DA)
- **O que o código faz**: bloqueia a inclusão de novos dependentes quando `#NUM-DEP > 5`, permitindo efetivamente no máximo 6 entradas (off-by-one: o 6º é gravado antes do bloqueio ser acionado na próxima iteração); o DDM define slot para 10
- **Hipótese do time**: limite foi definido administrativamente em algum momento (possivelmente política interna de 2008, mesma data do ajuste do PE Group no cabeçalho) e hardcoded sem atualizar o DDM — ou o DDM foi expandido de 5 para 10 após 2008 sem atualizar o programa
- **Risco se ignorarmos**: sistema moderno que mapear `GRP-DEPENDENTE` como coleção de até 10 elementos terá comportamento divergente do legado; famílias com 7–10 dependentes documentados no Adabas nunca puderam ser cadastradas — dados podem estar incompletos na base de origem

### MYS-003: Constante `0.347215` em `#FATOR-K` — origem completamente desconhecida

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/CADPROG.NSN#L86-L88` + `01-arqueologia/legado-sifap/adabas-ddms/PROGRAMA-SOCIAL.ddm#L40-L44`
- **Trecho de código**:

```natural
* CALC VLR BASE AJUSTADO C/ FATOR K
COMPUTE #FATOR-K = 1.00 + (#FATOR-REAJ * 0.347215)
COMPUTE #VLR-CALC = #VLR-BASE * #FATOR-K
```

Campo correspondente no DDM (`PROGRAMA-SOCIAL.ddm`):

```
  1  BG  FATOR-K                N        5.4   -     FATOR CORRECAO ESPECIAL
*                                                     >>> NAO DOCUMENTADO <<<
*                                                     INSERIDO AGO/2008 POR ADILSON
*                                                     "ATENDE SOLICITACAO SENARC"
*                                                     SEM MAIS DETALHES NO CHAMADO
```

- **O que esperávamos**: qualquer fator de correção financeira ter origem rastreável — portaria, índice oficial, resolução interna
- **O que o código faz**: multiplica o valor-base pelo resultado de `1.00 + (fator_reajuste × 0.347215)` antes de gravar; o `0.347215` não tem comentário, não consta em nenhuma tabela, portaria ou registro encontrado; o campo `FATOR-K` no DDM foi adicionado em ago/2008 — 5 anos _depois_ do código (alterado em 2003) — citando apenas "ATENDE SOLICITACAO SENARC" sem número de chamado válido
- **Hipótese do time**: pode ser um índice de correção vinculado a alguma medida provisória de 2003 (período de frequentes reajustes de benefícios sociais pós-Plano Real), ou um fator empírico ajustado manualmente e nunca formalizado; a divergência de datas sugere que o campo do DDM foi acrescentado em 2008 para _registrar_ o fator já em uso há 5 anos, não para defini-lo
- **Risco se ignorarmos**: sistema moderno que recalcular `VLR-BASE` sem aplicar esse fator produzirá valores divergentes dos atualmente pagos; aplicar o fator sem entender sua origem legal pode gerar passivo jurídico em auditoria

### MYS-004: Dezembro — fórmula do 13º silenciosamente omite fatores usados no mensal

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/CALCBENF.NSN#L253-L280`
- **Trecho de código**:

```natural
* CALC 13O SALARIO - DEZEMBRO
* FORMULA DIFERENCIADA: VLR_13 = VLR_BASE * FATOR_REG * (MESES_ATIVOS/12)
IF #MES = 12
  MOVE 'D' TO #TIPO-PGTO
  COMPUTE #VLR-13 = #VLR-BASE * #FATOR-REG * #FATOR-IDADE
  COMPUTE #VLR-BRUTO = #VLR-BENF + #VLR-13
*
* ABONO NATALINO - 15% ADICIONAL PARA PROGRAMAS TIPO 'A'
  IF #TIPO-PROG = 'A'
    COMPUTE #VLR-ABONO = #VLR-BENF * 0.15
    COMPUTE #VLR-BRUTO = #VLR-BRUTO + #VLR-ABONO
  ELSE
    MOVE 0 TO #VLR-ABONO
  END-IF
END-IF
```

- **O que esperávamos**: o 13º salário seguir a mesma fórmula do benefício mensal (`VLR-BASE × FATOR-REG × FATOR-FAM × FATOR-RND × FATOR-IDADE`), aplicando os mesmos fatores de forma consistente
- **O que o código faz**: três mudanças simultâneas ocorrem em dezembro — (1) `#FATOR-FAM` e `#FATOR-RND` são **silenciosamente removidos** da fórmula do 13º (`#VLR-13 = #VLR-BASE * #FATOR-REG * #FATOR-IDADE`); (2) programas do tipo `'A'` recebem um abono natalino de 15% sobre o valor mensal, sem qualquer comentário explicando a regra ou a base legal; (3) `TIPO-PGTO` muda para `'D'` afetando o fluxo de descontos logo abaixo; o comentário no cabeçalho do bloco menciona "MESES_ATIVOS/12" mas nenhum cálculo proporcional foi implementado
- **Hipótese do time**: a omissão de `#FATOR-FAM` parece intencional (o 13º seria "por pessoa", não "por família") mas nunca está documentada; o abono natalino de 15% exclusivo para tipo `'A'` (Assistencial) foi adicionado em 22/12/2009 por Jose Ferreira (comentário no cabeçalho: "ABONO NATALINO") — provavelmente atende algum decreto de fim de ano, mas a lei/portaria não está referenciada; o comentário "MESES_ATIVOS/12" sugere uma lógica de proporcionalidade que foi planejada mas nunca codificada
- **Risco se ignorarmos**: sistema moderno que replicar a fórmula mensal para o 13º pagará valores diferentes dos que o legado paga; famílias com muitos dependentes (alto `#FATOR-FAM`) e beneficiários de alta renda (baixo `#FATOR-RND`) serão os mais impactados; o abono natalino de 15% não implementado para tipo `'P'` e `'T'` pode ser erro ou regra — sem documentação, não é possível distinguir

### MYS-005: Truncação via variável inteira `(N11)` — perda sistemática de centavos em todo cálculo financeiro

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/CALCBENF.NSN#L231-L233` (padrão repetido em L245–247, L253–255, L271–272) + `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN#L103–105` e `L174–176`
- **Trecho de código** (padrão idêntico repetido 6 vezes nos dois programas):

```natural
* TRUNCAR P/ 2 CASAS DECIMAIS - PADRAO MAINFRAME
COMPUTE #VLR-TEMP = #VLR-BENF * 100
COMPUTE #VLR-BENF = #VLR-TEMP / 100
```

Declaração da variável usada como "truncador":

```natural
  1 #VLR-TEMP            (N11)   /* inteiro - sem casas decimais */
```

- **O que esperávamos**: arredondamento bancário padrão (half-up) — `INT(valor × 100 + 0.5) ÷ 100`
- **O que o código faz**: `#VLR-TEMP` é declarado como `(N11)` — inteiro sem casas decimais; ao fazer `COMPUTE #VLR-TEMP = #VLR-BENF * 100`, o Natural/Adabas descarta a parte fracionária por atribuição a inteiro (floor truncation); dividir por 100 devolve 2 casas decimais mas sempre arredondado para baixo; o padrão é aplicado **4 vezes** num pagamento normal (VLR-BENF, VLR-LIQ em CALCBENF + VLR-MAX-DSCT, VLR-TOTAL-DSCT em CALCDSCT) e **6 vezes** em dezembro (+ VLR-13 e VLR-ABONO); o comentário `PADRAO MAINFRAME` indica que é herança consciente do sistema original, mas não justifica a ausência de arredondamento
- **Hipótese do time**: a implementação replica o comportamento de um campo `PACKED DECIMAL` de mainframe IBM que também trunca, e foi portada literalmente para Natural sem avaliar se a semântica financeira deveria ser preservada ou corrigida; a perda por truncação é sempre a favor do pagador (governo), nunca do beneficiário
- **Risco se ignorarmos**: (1) sistema moderno que usar `BigDecimal` com `HALF_UP` produzirá valores diferentes dos históricos gravados no Adabas — divergência em reconciliação financeira; (2) com ~4,2 milhões de beneficiários, uma perda média de R$ 0,003 por truncação × 4 truncações × 12 meses = ~R$ 600 mil/ano em valores não pagos que podem ser exigidos retroativamente em auditoria; (3) a truncação em CALCDSCT é aplicada ao desconto (beneficiários pagam ligeiramente menos), criando um duplo desvio assimétrico impossível de explicar sem conhecer a técnica

### MYS-006: Desconto judicial (`'J'`) ignora o teto de 30% aplicado a todos os demais tipos

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN#L128-L131` (cálculo sem teto) e `L163-L167` (verificação de teto que exclui `'J'`)
- **Trecho de código**:

```natural
* CALC TETO MAXIMO DESCONTO - 30% DO BRUTO
COMPUTE #VLR-MAX-DSCT = #VLR-BRUTO * 0.30

    VALUE 'J'
* DESCONTO JUDICIAL - VALOR FIXO OU PERCENTUAL
        IF BENEFICIARIO-V.VLR-DSCT(#IDX) > 0
          MOVE BENEFICIARIO-V.VLR-DSCT(#IDX) TO #VLR-DSCT-ITEM
        ELSE
          COMPUTE #VLR-DSCT-ITEM = #VLR-BRUTO *
              (BENEFICIARIO-V.PCT-DSCT(#IDX) / 100)
        END-IF
* JUDICIAL NAO TEM TETO
        ADD #VLR-DSCT-ITEM TO #VLR-TOTAL-DSCT

* APLICAR TETO 30% - EXCETO JUDICIAL
    IF #TIPO-DSCT NE 'J'
      IF #VLR-TOTAL-DSCT > #VLR-MAX-DSCT
        MOVE #VLR-MAX-DSCT TO #VLR-TOTAL-DSCT
      END-IF
    END-IF
```

- **O que esperávamos**: todos os descontos obedecerem ao teto de 30% do valor bruto, protegendo um mínimo de 70% para o beneficiário
- **O que o código faz**: o tipo `'J'` (Judicial — penhora ou ordem de pagamento por decisão judicial) é adicionado ao total sem nenhuma verificação de limite; a guarda `IF #TIPO-DSCT NE 'J'` ao final do loop exclui explicitamente o judicial da aplicação do teto; se o valor ou percentual da ordem judicial superar o bruto inteiro, `#VLR-TOTAL-DSCT` excede `#VLR-BRUTO`; o único protetor restante está em `CALCBENF.NSN` (`IF #VLR-LIQ < 0 → MOVE 0`), que zera o líquido mas não reverte o desconto gravado
- **Hipótese do time**: a exceção é juridicamente justificável — no Brasil, ordens judiciais de alimentos ou penhora podem, em tese, não ter teto sobre benefícios assistenciais dependendo da decisão; o problema é que o código não documenta **qual** base legal autoriza isso, não valida o tipo de ação judicial (`NUM-PROCESSO` existe no DDM mas nunca é verificado), e não implementa piso mínimo de subsistência que a legislação posterior (Lei 13.105/2015) exige
- **Risco se ignorarmos**: sistema moderno que aplicar o teto de 30% a todos os tipos bloqueará ordens judiciais ativas — descumprimento de decisão judicial; sistema que replicar a exceção sem validação continuará permitindo que ordens judiciais zerem o líquido, expondo o órgão a ações por violação do mínimo existencial

### MYS-007: `CPF-DEPENDENTE` aceita `00000000000` como valor válido — deduplica­ção ignorada e tipo inconsistente com o DDM

- **Arquivo**: `01-arqueologia/legado-sifap/adabas-ddms/BENEFICIARIO.ddm#L62` + `01-arqueologia/legado-sifap/natural-programs/CADDEPEND.NSN#L97` + `01-arqueologia/legado-sifap/natural-programs/VALDOCS.NSN#L102`
- **Trecho de código**:

```
* BENEFICIARIO.ddm — campo DB
  2  DB  CPF-DEPENDENTE         A       11     -     CPF OU 00000000000
```

```natural
* CADDEPEND.NSN — verificação de duplicata ignora CPF zero
  IF BENEFICIARIO-V.CPF-DEP(#IDX) = #CPF-DEP AND #CPF-DEP NE 0
    WRITE 'DEPENDENTE JA CADASTRADO (CPF DUPLICADO)'
```

```natural
* VALDOCS.NSN — validação CPF curto-circuita para zero
DEFINE SUBROUTINE VALIDA-CPF-DOC
  IF #CPF = 0
    MOVE FALSE TO #CPF-OK
    ESCAPE ROUTINE
  END-IF
```

- **O que esperávamos**: `CPF-DEPENDENTE = 0` ser tratado uniformemente — ou rejeitado como inválido (como ocorre com o titular em `CADBENEF.NSN`), ou aceito com uma regra de unicidade alternativa (nome + data de nascimento)
- **O que o código faz**: três comportamentos inconsistentes simultâneos — (1) o DDM declara o campo como `A 11` (alfanumérico), mas a VIEW em `CADDEPEND.NSN` o acessa como `(N11)` (numérico), forçando conversão implícita de `"00000000000"` → `0`; (2) a verificação de duplicata tem a cláusula `AND #CPF-DEP NE 0`, o que significa que múltiplos dependentes sem CPF podem ser cadastrados para o mesmo titular sem qualquer controle de unicidade; (3) `VALDOCS.NSN` rejeita CPF zero como inválido (`#CPF-OK = FALSE`) mas esse programa não é chamado durante o cadastro de dependentes — apenas para o titular
- **Hipótese do time**: `00000000000` foi introduzido para acomodar dependentes sem CPF (crianças pequenas, trabalhadores informais antes da obrigatoriedade em 2003); a solução foi pragmática e a exclusão do check de duplicata parece intencional para esse caso; a inconsistência de tipo `A` vs `N` no DDM vs VIEW é mais grave — sugere que o campo foi originalmente `A` para suportar a string com zeros à esquerda, mas o programa o leu como numérico e "funcionou" porque `"00000000000"` como número é `0`
- **Risco se ignorarmos**: sistema moderno que mapear `CPF-DEPENDENTE` como `VARCHAR(11)` ou `CHAR(11)` precisará tratar `"00000000000"` como sentinela especial e **nunca** aplicar validação de CPF sobre ele; um mapeamento como `BIGINT` ou `NUMERIC(11)` perde a semântica — `0` e `"00000000000"` são diferentes para validação; a ausência de deduplicação por nome/nascimento para CPF zero significa que a base pode ter dependentes duplicados não detectáveis

### MYS-008: Região `99` (Internacional/Diplomático) pula todas as verificações de elegibilidade

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN#L98-L103`
- **Trecho de código**:

```natural
* ============================================
* REGIAO 99 - INTERNACIONAL/DIPLOMATICO
* ============================================
IF #COD-REG = 99
  MOVE TRUE TO #ELEGIVEL
  WRITE 'BENEFICIARIO ELEGIVEL - REGIAO ESPECIAL'
  ESCAPE ROUTINE
END-IF
*
* ============================================
* VERIF STATUS BENEFICIARIO     ← nunca alcançado para região 99
* ============================================
```

Campo correspondente no DDM (`BENEFICIARIO.ddm`):

```
  2  BJ  COD-REGIAO             A        2     -     01-05 OU 99 (ESPECIAL)
```

- **O que esperávamos**: uma região especial ter regras de elegibilidade próprias (mais restritivas ou distintas), não a ausência total de verificação
- **O que o código faz**: quando `COD-REGIAO = 99`, o programa seta `#ELEGIVEL = TRUE`, imprime `BENEFICIARIO ELEGIVEL - REGIAO ESPECIAL` e executa `ESCAPE ROUTINE` — saindo imediatamente; todas as verificações subsequentes são completamente ignoradas: status do beneficiário (suspenso/cancelado/inativo), faixa etária mínima e máxima, teto de renda familiar, completude de documentação (`DOCS-OK`) e regras específicas por tipo de programa (`A`/`P`/`T`); um beneficiário com status `'C'` (cancelado) e renda acima do teto, sem documentação, na região 99, é aprovado sem ressalvas
- **Hipótese do time**: a alteração foi feita em `05/04/2013` por Anderson Lima (`INC REGIAO 99`) — provavelmente para atender um caso de diplomatas brasileiros no exterior ou beneficiários em situação especial reconhecida por tratado; a implementação como `ESCAPE ROUTINE` total, no entanto, é uma decisão de design perigosa: não há log de auditoria específico para aprovações via região 99, não há lista de CPFs autorizados, e qualquer operador com acesso ao cadastro pode atribuir `COD-REGIAO = 99` a qualquer beneficiário
- **Risco se ignorarmos**: (1) ao migrar, se a região 99 não for tratada como caso especial explícito, esses beneficiários serão reavaliados pela engine de elegibilidade e possivelmente bloqueados — interrupção de pagamentos; (2) se a exceção for replicada sem auditoria, o sistema moderno herda uma porta de entrada sem controle de acesso; (3) o campo `COD-REGIAO` no DDM diz apenas `01-05 OU 99 (ESPECIAL)` — não há documentação do que "especial" significa, quantos beneficiários têm esse código, ou quem pode atribuí-lo

### MYS-009: Ordem de processamento batch por CPF — "otimização de 1999" virou dependência estrutural não documentada

- **Arquivo principal**: `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN#L178-L182` (dependêntes: `BATCHCON.NSN`, `BATCHREL.NSN`)
- **Trecho de código**:

```natural
* PROCESSAMENTO PRINCIPAL
* LEITURA EM ORDEM ALFABETICA POR CPF (OTIMIZACAO 1999)
* NOTA: SISTEMAS DOWNSTREAM DEPENDEM DESTA ORDENACAO
MOVE 0 TO #CPF-ANT
READ BENEFICIARIO-V BY CPF

  IF BENEFICIARIO-V.CPF = #CPF-ANT   ← duplicata detectada apenas por adjacência
    ADD 1 TO #QTD-IGNORADOS
    ESCAPE TOP
  END-IF
  MOVE BENEFICIARIO-V.CPF TO #CPF-ANT
```

- **O que esperávamos**: o batch processar por programa ou por região — ordens que fazem sentido para rotear pagamentos a bancos e gerar relatórios regionais; ou ao menos que a dependência de ordem estivesse documentada em todos os programas afetados
- **O que o código faz**: `BATCHPGT` lê todos os beneficiários em ordem crescente de CPF (`READ BENEFICIARIO-V BY CPF`), que é o índice primario Adabas (`AB`, campo `(DE)`); a ordem de leitura determina a sequência em que `#SEQ-PGTO` é incrementado e atribuído como `NUM-PAGTO` — ou seja, `NUM-PAGTO` reflete indiretamente a ordem de CPF; três programas dependem disso de formas diferentes: (1) o próprio `BATCHPGT` usa `#CPF-ANT` para dedup por adjacência — só funciona porque os registros chegam ordenados; (2) `BATCHCON` reconcilia o arquivo CNAB retornado pelo banco, que também vem em ordem de CPF porque o arquivo enviado ao banco foi gerado em ordem de CPF; (3) `BATCHREL` lê pagamentos `BY COMPETENCIA` e acumula subtotais regionais em ordem que não é explicada, mas os testes de consistência foram homologados com a ordem CPF
- **Hipótese do time**: a alteração de janeiro/2000 (`OTIMIZ ORD CPF`) foi feita porque `BY CPF` usa o descritor Adabas (`AB`) e é um full-sequential read sem sort externo — muito mais rápido do que ler por programa ou região; em 1999/2000 isso provavelmente representou uma redução significativa de tempo de batch com 4,2 milhões de registros; o efeito colateral é que o banco (Banco do Brasil) passou a receber e a devolver arquivos CNAB na mesma sequência, e `BATCHCON` foi escrito assumindo isso; a nota `SISTEMAS DOWNSTREAM DEPENDEM DESTA ORDENACAO` é o único aviso — e não aparece em `BATCHCON` nem em `BATCHREL`
- **Risco se ignorarmos**: (1) qualquer refatoração que mude a ordem de processamento (ex.: processar por região para paralelismo) quebra a detecção de duplicatas em `BATCHPGT`, a conciliação em `BATCHCON` e potencialmente os totais em `BATCHREL`; (2) o sistema moderno com JPA/Hibernate não garante ordem de `findAll()` sem `ORDER BY` explícito — se a migração não reproduzir `ORDER BY cpf` o dedup por adjacência falha silenciosamente; (3) se o banco parceiro processar o CNAB em ordem diferente em alguma competência, `BATCHCON` gerará falsos `NAO ENCONTRADOS`

### MYS-010: Eventos `'EX'` (Exclusão) são sistematicamente ocultos do relatório de auditoria — intencional ou bug?

- **Arquivo**: `01-arqueologia/legado-sifap/natural-programs/RELAUDIT.NSN#L99-L103` + `01-arqueologia/legado-sifap/adabas-ddms/AUDITORIA.ddm#L26-L28`
- **Trecho de código**:

```natural
* ============================================
* FILTRO ACAO - EXCLUSOES NAO SAO EXIBIDAS
* ============================================
  IF AUDITORIA-V.ACAO = 'EX'
    ADD 1 TO #QTD-FILTRADOS
    ESCAPE TOP
  END-IF
*
* FILTRO ACAO (SE INFORMADO)      ← nunca alcançado para 'EX'
  IF #ACAO-FILTRO NE ' '
    IF AUDITORIA-V.ACAO NE #ACAO-FILTRO
```

Sumário ao final do relatório (nenhuma linha para exclusões):

```natural
  '  INCLUSOES........:' #QTD-INCLUSAO /
  '  ALTERACOES.......:' #QTD-ALTERACAO /
  '  CONSULTAS........:' #QTD-CONSULTA /
  '  CONCILIACOES.....:' #QTD-CONCILIACAO /
  '  DIVERGENCIAS.....:' #QTD-DIVERGENCIA /
  '  OUTRAS...........:' #QTD-OUTROS
  /* SEM #QTD-EXCLUSAO */
```

DDM correspondente (`AUDITORIA.ddm`):

```
  1  BA  COD-ACAO               A        2     -     TIPO ACAO (DE)
*                                                     EX=EXCLUSAO
* DESCRICAO: LOG DE AUDITORIA - TRILHA DE ALTERACOES DO SIFAP
*            REGISTRO IMUTAVEL - NAO PERMITE UPDATE/DELETE
*            OBRIGATORIEDADE LEGAL: IN-TCU 63/2010
```

- **O que esperávamos**: todos os tipos de ação definidos no DDM — inclusive `EX=EXCLUSAO` — aparecerem na trilha de auditoria, especialmente por obrigatoriedade legal (IN-TCU 63/2010)
- **O que o código faz**: o filtro `IF AUDITORIA-V.ACAO = 'EX'` ocorre **antes** do filtro de usuário, tornando impossível exibir eventos de exclusão mesmo que o operador informe `ACAO-FILTRO = 'EX'` explicitamente; os eventos são contabilizados em `#QTD-FILTRADOS` junto com os filtros opcionais do usuário, mascarando quantos eventos `EX` existem; a variável `#QTD-EXCLUSAO` nunca foi declarada; o bloco `DECIDE ON FIRST VALUE` que conta por tipo não tem `VALUE 'EX'` — logo, mesmo que o filtro fosse removido, os eventos `EX` cairiam em `#QTD-OUTROS` sem identificação; a alteração `15/09/2014 - FERNANDA COSTA - LIMPEZA RELATORIO` é o único rastro da introdução desse comportamento
- **Intencional ou bug?** O comentário `EXCLUSOES NAO SAO EXIBIDAS` é deliberado e escrito no imperativo — não parece descuido; possíveis motivos intencionais: (a) decisão operacional de não expor exclusões em relatórios de rotina por ser informação sensível; (b) cobertura de exclusões irregulares realizadas diretamente no Adabas por DBAs; (c) simplificação de relatório que se tornou permanente; o mais provável é que seja **intencional mas juridicamente indefensável**: a IN-TCU 63/2010 exige rastreabilidade completa de alterações, e o próprio DDM declara `OBRIGATORIEDADE LEGAL` e `RETENCAO MINIMA: 10 ANOS`
- **Risco se ignorarmos**: (1) sistema moderno que replicar o filtro herda uma violação de compliance ativa; (2) sistema que remover o filtro pode revelar um volume de eventos `EX` acumulado desde 2014 que nunca foi auditado — possível evidência de irregularidades; (3) a migração deve decidir explicitamente sobre a política de exibição de exclusões e documentar a decisão em ADR antes de codificar

## Easter Eggs

> Dica: existem **3 easter eggs** escondidos no código legado. Registre aqui os que encontrar:

1. [x] Easter Egg 1: No arquivo CALCCCORR.NSN há referência ao Plano Verão, mudança de cruzado para cruzeiro
2. [x] Easter Egg 2: Em `VALDOCS.NSN` a subroutine `CHECK-DOC-ESPECIAL` (adicionada em 07/06/2011 por Roberto Mendes, comentário `PREFIXOS GOVERNO/TESTE`) aceita sem validação qualquer CPF cujos 3 primeiros dígitos sejam `000`, `001`, `002`, `010`, `011`, `099`, `100` ou `999` — ao detectar um desses prefixos, zera `#QTD-ERROS`, força `#CPF-OK = TRUE` e `#RESULTADO = 'V'`, anulando inclusive falhas já registradas pelo algoritmo MOD-11; o comentário em `VALBENEF.NSN` confirma: `EXCECAO: CPFs INICIADOS COM 000 SAO VALIDOS (TESTE GOVERNO)` — backdoor de ambiente de testes que nunca foi removido da produção
3. [x] Easter Egg 3: Em `BATCHCON.NSN` existe um bloco inteiro de código morto comentado (linhas 207–223) para integração com o **Banco Real** (código bancário `356`), adicionado em 18/09/2005 por Marcos Ribeiro; o próprio código se explica: `BANCO REAL FOI ADQUIRIDO PELO SANTANDER EM 2007` e `MANTER CODIGO PARA REFERENCIA HISTORICA`; o layout CNAB do Banco Real era diferente do Banco do Brasil (CPF na posição 30, valor na posição 100 — vs posição 44 e 120 no BB), e a subroutine `CONCILIA-REAL` referenciada no bloco nunca chegou a ser escrita — a integração foi descontinuada antes de ser concluída ou o código foi apagado junto com a desativação

## Resumo

- Total de mistérios encontrados: \_\_\_
- Confiança alta: \_\_\_
- Confiança média: \_\_\_
- Confiança baixa: \_\_\_
- Easter eggs encontrados: \_\_\_ / 3

---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="mysteries-checklist.md"><strong>mysteries-checklist.md</strong></a><br/>
<sub>Lista do que procurar.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="discovery-report.md"><strong>discovery-report.md</strong></a><br/>
<sub>Síntese final.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="README.md">Voltar ao Kit PT-BR</a></sub>

