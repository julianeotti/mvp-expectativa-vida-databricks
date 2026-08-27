# Catálogo de Dados

## 1. Objetivo

Este catálogo documenta os atributos utilizados no modelo dimensional da camada Gold do MVP, incluindo suas definições, tipos, fontes, domínios, valores esperados, valores observados no recorte analítico de 2013 e principais regras de qualidade.

Os dados foram obtidos a partir de duas fontes distintas, posteriormente integradas pela chave composta país-ano. Os atributos originais foram preservados na camada Bronze, tratados e padronizados na camada Silver e organizados em esquema estrela na camada Gold.

## 2. Fontes dos dados

| Fonte | Arquivo | Conteúdo | URL | Licença/condições de uso | Data de acesso |
|---|---|---|---|---|---|
| WHO / Base de expectativa de vida | `WHO_life_expectancy.csv` | Indicadores de expectativa de vida, mortalidade, saúde e aspectos socioeconômicos | https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who/data | INSERIR | INSERIR |
| Base de desenvolvimento humano | `IDH.csv` | Indicadores de escolaridade, educação e IDH | INSERIR URL | INSERIR | INSERIR |

> Os valores mínimos e máximos observados apresentados neste catálogo correspondem à base analítica final de 2013. Para atributos sem limite superior formal informado pela fonte, não foi criado um valor máximo arbitrário.

---

# 3. Dimensão País — `workspace.gold.dim_pais`

| Atributo | Descrição | Tipo | Origem | Mínimo esperado | Máximo esperado | Domínio/Categorias | NULL | Regra de qualidade |
|---|---|---|---|---|---|---|---|---|
| `id_pais` | Chave numérica gerada para identificar unicamente cada país na dimensão | INTEGER | Gerado no pipeline | 1 | Número de países da dimensão | Inteiros positivos | Não | Deve ser único e não nulo |
| `pais` | Nome do país | STRING | WHO / IDH | — | — | Países presentes na base integrada | Não | Deve identificar unicamente um país na dimensão |
| `status` | Classificação econômica do país disponibilizada pela fonte | STRING | WHO | — | — | `Developed`, `Developing` | Não | Apenas categorias existentes no domínio da fonte |

No recorte de 2013, a dimensão contém 180 países.

---

# 4. Dimensão Tempo — `workspace.gold.dim_tempo`

| Atributo | Descrição | Tipo | Origem | Mínimo esperado | Máximo esperado | Domínio | NULL | Regra de qualidade |
|---|---|---|---|---|---|---|---|---|
| `id_tempo` | Chave temporal utilizada na tabela fato | INTEGER | Gerado a partir de `ano` | 2013 | 2013 | Ano selecionado para o MVP | Não | Deve existir na dimensão Tempo |
| `ano` | Ano de referência dos indicadores | INTEGER | WHO / IDH | 2013 | 2013 | 2013 | Não | MVP restrito ao recorte analítico de 2013 |

---

# 5. Tabela Fato — `workspace.gold.fato_saude_desenvolvimento`

| Atributo | Descrição | Tipo | Fonte | Mínimo esperado | Máximo esperado | Mínimo observado 2013 | Máximo observado 2013 | Nulos em 2013 | Regra de qualidade |
|---|---|---|---|---|---|---:|---:|---:|---|
| `id_pais` | Chave estrangeira da dimensão País | INTEGER | Pipeline | 1 | Nº de países | 1 | 180 | 0 | Deve existir em `dim_pais` |
| `id_tempo` | Chave estrangeira da dimensão Tempo | INTEGER | Pipeline | 2013 | 2013 | 2013 | 2013 | 0 | Deve existir em `dim_tempo` |
| `exp_vida` | Expectativa de vida do país no ano analisado, em anos | DOUBLE | WHO | > 0 | 120 | 49,9 | 87,0 | 4 | Valores não positivos são considerados inválidos |
| `mortalidade_adulta` | Mortalidade de adultos entre 15 e 60 anos por 1.000 pessoas | DOUBLE | WHO | 0 | 1.000 | 5,0 | 518,0 | 4 | Não pode assumir valores negativos |
| `mortalidade_infantil` | Mortalidade infantil por 1.000 pessoas, conforme definição da fonte | DOUBLE | WHO | 0 | 1.000 | 0,0 | 1.000,0 | 0 | Não pode assumir valores negativos |
| `mortalidade_hiv` | Mortalidade de crianças de 0 a 4 anos relacionada ao HIV/AIDS por 1.000 habitantes, conforme definição da fonte | DOUBLE | WHO | 0 | 1.000 | 0,1 | 9,6 | 0 | Não pode assumir valores negativos |
| `consumo_alcool` | Consumo registrado de álcool puro, em litros per capita, entre pessoas com mais de 15 anos | DOUBLE | WHO | 0 | Não definido pela fonte | 0,01 | 15,04 | 2 | Não pode assumir valores negativos |
| `imc` | IMC médio da população segundo a variável disponibilizada na fonte | DOUBLE | WHO | > 0 | Não definido pela fonte | 2,1 | 83,3 | 2 | Valores não positivos são inválidos; extremos devem ser avaliados com cautela |
| `pib_dolar` | PIB per capita em dólares | DOUBLE | WHO | 0 | Não existe máximo teórico | 14,21 | 113.751,85 | 25 | Não pode assumir valores negativos |
| `anos_escolaridade_media` | Anos médios de escolaridade do país | DOUBLE | IDH | 0 | Não definido pela fonte | 4,9 | 20,4 | 4 | Não pode assumir valores negativos |
| `indice_educacao` | Índice de educação utilizado na composição dos indicadores de desenvolvimento humano | DOUBLE | IDH | 0 | 1 | 0,224 | 0,981 | 6 | Valores fora do intervalo 0–1 são convertidos para NULL antes da integração |
| `idh` | Índice de Desenvolvimento Humano do país | DOUBLE | IDH | 0 | 1 | 0,345 | 0,946 | 1 | Valores válidos devem permanecer entre 0 e 1 |

---

# 6. Indicadores de qualidade da camada Silver

Os indicadores abaixo são utilizados durante o tratamento dos dados e não integram diretamente a tabela fato analítica.

| Atributo | Tabela | Descrição | Domínio |
|---|---|---|---|
| `flag_registro_invalido` | WHO Silver | Identifica registros que violam regras básicas relacionadas a status, ano ou expectativa de vida | `0 = válido`; `1 = inválido` |
| `flag_chave_duplicada` | IDH Silver | Identifica registros em que a combinação país-ano não é única | `0 = única`; `1 = duplicada` |
| `flag_indice_educacao_invalido` | IDH Silver | Identifica valores do índice de educação fora do domínio esperado de 0 a 1 | `0 = válido`; `1 = inválido` |

---

# 7. Linhagem dos dados

| Fonte original | Camada Bronze | Transformação principal | Camada Silver | Destino na Gold |
|---|---|---|---|---|
| `WHO_life_expectancy.csv` | `workspace.bronze.who_life_expectancy` | Conversão de tipos, tratamento de `NA`, padronização de nomes e aplicação de regras de qualidade | `workspace.silver.who_life_expectancy` | `dim_pais`, `dim_tempo` e `fato_saude_desenvolvimento` |
| `IDH.csv` | `workspace.bronze.idh` | Conversão de tipos, tratamento de `NA`, padronização de `paises` para `pais` e identificação de duplicidades e valores inválidos | `workspace.silver.idh` | `dim_tempo` e `fato_saude_desenvolvimento` |

A integração entre as duas fontes é realizada na construção da camada Gold por meio de `INNER JOIN` utilizando a chave composta `pais + ano`. O recorte analítico para 2013 é aplicado somente após o tratamento realizado na camada Silver.

Registros associados a chaves país-ano ambíguas são excluídos da integração. Valores do índice de educação fora do domínio esperado são convertidos para NULL, preservando os demais atributos do registro.

---

# 8. Modelo de dados

O modelo final utiliza esquema estrela:

- `dim_pais`: informações descritivas dos países;
- `dim_tempo`: período de referência;
- `fato_saude_desenvolvimento`: indicadores quantitativos de saúde, educação e desenvolvimento.

A granularidade da tabela fato corresponde a um registro por país no ano de 2013.
