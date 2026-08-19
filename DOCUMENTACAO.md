# Dicionário de Dados e Documentação do Modelo

Este documento descreve a estrutura do modelo de dados (*Star Schema*), as tabelas, o fluxo de ETL (Power Query) e as principais medidas em DAX utilizadas no painel de Varejo de Calçados.

---

## 🏗️ 1. Arquitetura do Modelo (Star Schema)
O modelo foi otimizado utilizando o conceito de *Star Schema*, garantindo alta performance de leitura e cruzamento eficiente de filtros.
- **Tabelas Fato (Fact Tables):** Armazenam os eventos de negócio (histórico e transações).
- **Tabelas Dimensão (Dimension Tables):** Armazenam os atributos qualitativos (filtros e categorias).

---

## 🗂️ 2. Tabelas do Modelo

### Tabelas Fato
* **`Fatos_Vendas`**: Registra cada item vendido na loja. Contém as chaves de ligação e os valores unitários, totais, custos e descontos.
* **`Fatos_Pagamentos`** *(Financeiro)*: Registra os métodos de pagamento (A vista, Prazo, Pix, Cartão) de cada transação.
* **`Fatos_Estoque`**: Registra o histórico diário/mensal de mercadorias em estoque, permitindo análises de custo imobilizado e tempo de prateleira (idade do estoque).

### Tabelas Dimensão
* **`Dim_Clientes`**: Cadastro único de clientes. Inclui faixa etária, data de nascimento (tratada para remoção de anomalias), endereço e contato. *(Nota: Contém o agrupamento "Consumidor Final" para vendas não identificadas).*
* **`Dim_Funcionarios`** *(Vendedoras)*: Cadastro da equipe de vendas. Filtra ativamente logins de sistema (E-commerce, Caixas genéricos) mapeando-os para "Vendedor Desconhecido" nas Fatos.
* **`Dim_Produtos`**: Hierarquia de mercadorias contendo Marca, Categoria (Sapato, Tênis, Sandália) e Tamanho.
* **`Dim_Lojas`**: Identificação das filiais/matriz físicas e virtuais.
* **`Dim_Calendario`**: Tabela de inteligência de tempo contendo Ano, Mês, Dia, Trimestre e Dia da Semana, essencial para funções de inteligência temporal (Sazonalidade).

---

## ⚙️ 3. Etapas de ETL e Tratamento (Power Query)
As principais transformações realizadas no Power Query (Linguagem M) para garantir a governança e limpeza dos dados:
1. **Conexão e Empilhamento:** Leitura e combinação (*Append*) de múltiplos arquivos `.csv` e consolidação em tabelas únicas.
2. **Remoção de Colunas Inúteis:** Exclusão de colunas residuais geradas por falhas na exportação do ERP (ex: `Unnamed: 17`).
3. **Tratamento de Tipagem:** Conversão rigorosa de textos para Moeda (`Currency.Type`), Números Inteiros (`Int64.Type`) e Datas.
4. **Limpeza de Nomes e Prefixos:** Remoção de sujeiras nos textos (ex: "DR.", "SRA.") para padronização.
5. **Tratamento de Exceções (Erros de Idade):** Identificação e substituição de datas de nascimento nulas ou implausíveis para manter a coerência dos gráficos demográficos.

---

## 🧮 4. Principais Medidas e Cálculos (DAX)
Todas as métricas do painel foram centralizadas em uma tabela virtual oculta (`_Medidas`) para facilitar a manutenção.

* **`Faturamento`**: Soma total da receita bruta de vendas.
* **`Ticket Médio`**: `Faturamento` dividido pelo número de cupons/vendas únicas.
* **`Itens por Cupom`**: Média da quantidade de produtos levados em cada transação.
* **`Clientes Únicos`**: Contagem distinta de `ID_CLIENTE` na tabela de vendas num dado período.
* **`Retenção (%)`**: Percentual de clientes que compraram mais de uma vez na janela de tempo analisada.
* **`Lucro Bruto`**: `Faturamento` menos o `Custo da Mercadoria Vendida (CMV)`.
* **`Markup Real`**: Margem aplicada sobre o custo base dos produtos vendidos.
* **`Custo Total do Estoque`**: Valoração atual de todos os produtos parados no almoxarifado.
* **`GMROI (Gross Margin Return on Investment)`**: Cálculo avançado que mede o retorno financeiro gerado para cada R$1,00 investido em estoque. Obtido dividindo o `Lucro Bruto` pelo `Estoque Médio Monetário`.
