# Análise de Vendas em Períodos Promocionais + SQL + Dashboard no Power BI

<img src="https://i.imgur.com/Kbq3skI.png" alt="drawing" width="100%"/>

# 1. Problema de Negócios

AtliQ Mart, uma grande empresa varejista com mais de 50 supermercados na região sul da Índia, organizou uma ampla promoção em todos os seus 50 estabelecimentos durante o Diwali 2023 e o Sankranti 2024, períodos festivos na Índia, em seus produtos da marca AtliQ. Agora, o diretor de vendas está buscando compreender quais promoções tiveram um desempenho satisfatório e quais não, a fim de tomar decisões embasadas para o próximo período promocional.

Ambas as campanhas apresentam 15 produtos em oferta, cobrindo as categorias Combo, Mercearia e Itens Básicos, Eletrodomésticos, Cuidados Domiciliares e Cuidados Pessoais. A diferença de tempo entre uma campanha e outra é de aproximadamente 60 dias. Participam 50 lojas distribuídas em 10 cidades. Cada campanha oferece cinco tipos de promoção: 25% de desconto, 33% de desconto, 50% de desconto, Compre um e Leve Outro Grátis (BOGOF) e 500 Cashback.


## 1.1. Sobre os dados

Os dados utilizados neste projeto foram adquiridos por meio do desafio número 9, organizado pela Plataforma de Educação [CodeBasic](https://codebasics.io/challenge/codebasics-resume-project-challenge). O conjunto de dados compreende as seguintes informações:

### dim_campaigns:

Variável | Descrição
---------|------------
campaign_id | Identificador único para cada campanha promocional.
campaign_name | Nome descritivo da campanha (por exemplo, Diwali, Sankranti).
start_date | A data em que a campanha começa, formatada como DD-MM-AAAA.
end_date | A data em que a campanha termina, formatada como DD-MM-AAAA.

### dim_products:

Variável | Descrição
---------|------------
product_code | Código único atribuído a cada produto para identificação.
product_name | O nome completo do produto, incluindo marca e especificidades (por exemplo, quantidade, tamanho).
category | A classificação do produto em categorias mais amplas, como Mercearia & Mantimentos, Cuidados Domésticos, Cuidados Pessoais, Eletrodomésticos, etc.


### dim_stores:

Variável | Descrição
---------|------------
store_id | Código único que identifica cada localização da loja.
city | A cidade onde a loja está localizada, indicando o mercado geográfico.


### fact_events:

Variável | Descrição
---------|------------
event_id | Identificador único para cada evento de venda.
store_id | Refere-se à loja onde o evento ocorreu, vinculado à tabela dim_stores.
campaign_id | Indica a campanha sob a qual o evento foi registrado, vinculado à tabela dim_campaigns.
product_code | O código do produto envolvido no evento de venda, vinculado à tabela dim_products.
base_price | O preço padrão do produto antes de qualquer desconto promocional.
promo_type | O tipo de promoção aplicada (por exemplo, desconto percentual, BOGOF (Compre Um e Ganhe Outro), cashback).
quantity_sold(before_promo) | O número de unidades vendidas na semana imediatamente anterior ao início da campanha, servindo como base de comparação com as vendas promocionais.
quantity_sold(after_promo) | A quantidade do produto vendida após a aplicação da promoção.


# 2. Premissas de Negócio

## 2.1. Definição de Métricas

Serão avaliadas a receita incremental e a quantidade vendida incremental. A receita incremental refere-se à diferença entre a receita total durante os seis dias de promoção e a receita da semana anterior, que consiste em um período de seis dias sem promoção. A quantidade vendida incremental pode ser definida como a diferença entre as unidades vendidas durante a promoção e as unidades vendidas na semana anterior à promoção, período sem promoção.

## 2.2.  Estratégias Promocionais e Considerações sobre os Produtos

### 2.2.1 Categorização dos Produtos Oferecidos nas Promoções

- Os Bens de Consumo de Rápido Movimento (FMCG) são produtos que se destacam por sua alta rotatividade e baixo custo. São itens consumidos regularmente, caracterizados por sua natureza não durável, o que implica em rápida utilização e reposição frequente. Essa categoria abrange produtos essenciais do dia a dia, como alimentos, bebidas, artigos de higiene pessoal, produtos de limpeza e outros itens domésticos acessíveis. Itens que são oferecidos em ambas as campanhas e se enquadram na categoria de Bens de Consumo de Rápido Movimento (FMCG) incluem loção hidratante corporal, sabonete, farinha de trigo, lentilha, arroz e óleo de girassol.

- Os produtos oferecidos em ambas as campanhas, como cortinas, lençóis, recipientes, lâmpadas, esponjas e aquecedores, pertencem à categoria de bens duráveis ou semi-duráveis. Isso se deve ao fato de apresentarem uma vida útil mais longa em comparação com os Bens de Consumo de Rápido Movimento (FMCG). 

- O combo essencial de 8 produtos para casa pode abranger uma variedade de itens que pertencem a diferentes categorias. Se ele for composto principalmente por produtos de alta rotatividade e baixo custo, como produtos de limpeza, será classificado como Bens de Consumo de Rápido Movimento (FMCG). No entanto, se também incluir itens duráveis ou semi-duráveis, como utensílios domésticos ou de cozinha, poderá ser uma combinação das duas categorias.

    **Existem várias razões para incluir esses produtos na campanha. As promoções podem ser feitas para incentivar a compra deles, evitar perdas devido à proximidade da data de vencimento, reduzir o estoque ou atrair clientes que normalmente não comprariam por causa do preço mais alto. Além disso, produtos relacionados a celebrações ou consumidos durante períodos festivos podem estar recebendo destaque.**

# 3. Planejamento da Solução

- Verificar o documento "Recommended Insights.pdf" para revisar algumas recomendações do gerente.

- Elaborar um relatório que englobe análises e respostas às questões comerciais críticas delineadas pelos executivos seniores, conforme descrito no documento "ad-hoc-requests.pdf".

- Apresentar a análise para o diretor de vendas.

## 3.1. Análise Comparativa

### 3.1.1 Produtos

