# Simulador de Investimentos em Fundos Imobiliários (FIIs)

Ferramenta em Excel que simula um investimento mensal em FIIs: a partir do aporte, calcula o patrimônio acumulado, os dividendos mensais estimados, projeta cenários de 2 a 30 anos e divide o aporte conforme o perfil do investidor.

Projeto desenvolvido no Desafio de Projeto do curso **Reclame AQUI – Dados e IA na Prática** (DIO).

## Como usar

1. Abra `simulador-fii.xlsx`, na aba **Simulador**.
2. Preencha as células amarelas (texto azul):
   - **Aporte mensal**: quanto você investe por mês.
   - **Dividend yield mensal**: rendimento médio mensal dos FIIs (padrão 0,8% a.m.).
   - **Prazo (anos)**: tempo de investimento.
   - **Perfil do investidor**: escolha na lista suspensa (Conservador, Moderado ou Arrojado).
3. Todo o resto é recalculado automaticamente.

## Estrutura da planilha

| Aba | Conteúdo |
|---|---|
| **Simulador** | Entradas, resultados, divisão do aporte por perfil, tabela de cenários e gráfico |
| **Perfis** | Tabela de apoio com o percentual de cada classe de FII por perfil |

## Intervalos nomeados

Em vez de referências como `$B$4`, as fórmulas usam nomes, o que as deixa mais legíveis e evita o uso de `$` ao arrastar.

| Nome | Referência | O que guarda |
|---|---|---|
| `Aporte` | Simulador!B4 | Aporte mensal |
| `TaxaMensal` | Simulador!B5 | Dividend yield mensal |
| `PrazoAnos` | Simulador!B6 | Prazo em anos |
| `Perfil` | Simulador!B7 | Perfil escolhido |
| `Patrimonio` | Simulador!B11 | Patrimônio final calculado |
| `TabPerfis` | Perfis!A2:D4 | Tabela de apoio usada no PROCV |
| `ListaPerfis` | Perfis!A2:A4 | Origem da lista suspensa de perfis |

Para criar um nome: selecione a célula, digite o nome na **Caixa de Nome** (à esquerda da barra de fórmulas) e pressione Enter. Para conferir todos: **Fórmulas > Gerenciador de Nomes**.

## Explicação dos cálculos

### 1. Total investido
```
=Aporte*PrazoAnos*12
```
Soma de todos os aportes: valor mensal × número de meses.

### 2. Patrimônio final (função VF)
```
=VF(TaxaMensal; PrazoAnos*12; -Aporte)
```
`VF` (valor futuro) calcula quanto uma série de pagamentos iguais vale ao final do período, com juros compostos.

- **taxa** = `TaxaMensal`: taxa e período precisam estar na mesma unidade. Como a taxa é mensal, o prazo é convertido em meses.
- **nper** = `PrazoAnos*12`: número de aportes.
- **pgto** = `-Aporte`: negativo porque, na convenção financeira do Excel, é dinheiro saindo do bolso do investidor. Sem o sinal, o resultado aparece negativo.

A fórmula assume que os dividendos recebidos são reinvestidos todo mês, por isso o patrimônio cresce de forma exponencial.

### 3. Ganho com dividendos reinvestidos
```
=B11-B10
```
Diferença entre o patrimônio final e o total investido: é o que os dividendos reinvestidos geraram.

### 4. Dividendos mensais estimados
```
=Patrimonio*TaxaMensal
```
Quanto o patrimônio acumulado passa a render por mês ao fim do prazo, mantida a mesma taxa.

### 5. Cenários de 2 a 30 anos
Tabela com uma linha para cada prazo, de 2 a 30 anos:

| Coluna | Fórmula (linha 5) | Significado |
|---|---|---|
| Anos | `2, 3, … 30` | Prazo do cenário |
| Meses | `=E5*12` | Prazo convertido em meses |
| Total investido | `=Aporte*F5` | Aportes acumulados |
| Patrimônio | `=VF(TaxaMensal; F5; -Aporte)` | Valor futuro naquele prazo |
| Dividendos/mês | `=H5*TaxaMensal` | Renda mensal estimada |

Como `Aporte` e `TaxaMensal` são nomes, a fórmula é arrastada para baixo sem precisar de `$`. Só a referência à linha (E5, F5, H5) muda. O gráfico **Patrimônio x Total investido** usa essa tabela e mostra o efeito dos juros compostos: a distância entre as duas linhas aumenta com o tempo.

### 6. Divisão do aporte por perfil (PROCV)
```
=PROCV(Perfil; TabPerfis; 2; FALSO)   → % em Tijolo
=PROCV(Perfil; TabPerfis; 3; FALSO)   → % em Papel
=PROCV(Perfil; TabPerfis; 4; FALSO)   → % em FOF
```
`PROCV` procura o perfil escolhido na primeira coluna de `TabPerfis` e retorna o valor da coluna indicada (2 = Tijolo, 3 = Papel, 4 = FOF). O `FALSO` exige correspondência exata do nome do perfil.

O valor mensal de cada classe é o percentual × aporte (`=B17*Aporte`). A linha **Total** soma tudo e serve de conferência: deve dar 100% e o valor do aporte.

**Tabela de apoio (aba Perfis):**

| Perfil | Tijolo | Papel | FOF |
|---|---|---|---|
| Conservador | 30% | 60% | 10% |
| Moderado | 50% | 40% | 10% |
| Arrojado | 60% | 20% | 20% |

- **Tijolo**: fundos donos de imóveis físicos (lajes, galpões, shoppings).
- **Papel**: fundos que investem em títulos de crédito imobiliário (CRI).
- **FOF**: fundos que investem em cotas de outros FIIs.

## Validações

- **Perfil**: lista suspensa (Dados > Validação de Dados > Lista, com origem `=ListaPerfis`), o que impede erro de digitação no PROCV.
- **Prazo**: aceita apenas números inteiros de 1 a 50.

## Prints da ferramenta

> Adicione os prints na pasta `images/` e eles aparecem aqui.

![Simulador](images/simulador.png)
![Cenários e gráfico](images/cenarios.png)
![Tabela de perfis](images/perfis.png)

## Premissas e limitações

- O dividend yield é considerado constante durante todo o prazo, o que não acontece na prática.
- Não considera valorização ou desvalorização das cotas, inflação, custos de corretagem ou impostos.
- Os percentuais por perfil são exemplos didáticos, não recomendação de investimento.
