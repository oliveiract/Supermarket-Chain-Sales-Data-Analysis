# Análise de Vendas em Períodos Promocionais + Excel + SQL + Dashboard no Power BI

<img src="https://i.imgur.com/Kbq3skI.png" width="100%"/>

# 1. Problema de Negócios

AtliQ Mart, uma grande empresa varejista com mais de 50 supermercados na região sul da Índia, organizou uma ampla promoção em todos os seus 50 estabelecimentos durante o Diwali 2023 e o Sankranti 2024, períodos festivos na Índia, em seus produtos da marca AtliQ. Agora, o diretor de vendas está buscando compreender quais promoções tiveram um desempenho satisfatório e quais não, a fim de tomar decisões embasadas para o próximo período promocional.

Ambas as campanhas apresentam 15 produtos em oferta, cobrindo as categorias Combo, Mercearia e Itens Básicos, Eletroeletrônicos, Cuidados Domiciliares e Cuidados Pessoais. A diferença de tempo entre uma campanha e outra é de aproximadamente 60 dias. Participam 50 lojas distribuídas em 10 cidades. Cada campanha oferece cinco tipos de promoção: 25% de desconto, 33% de desconto, 50% de desconto, Compre um e Leve Outro Grátis (BOGOF) e 500 Cashback.


## 1.1. Sobre os dados

Os dados utilizados neste projeto foram adquiridos por meio do desafio número 9, organizado pela Plataforma de Educação [CodeBasics](https://codebasics.io/challenge/codebasics-resume-project-challenge). O conjunto de dados compreende as seguintes informações:

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

## 1.2. Sobre as festividades

A Índia é um país cheio de diversidade cultural, onde cada comunidade possui suas próprias tradições. Ao longo do ano, são realizadas diversas festividades.

Diwali, conhecido como o Festival das Luzes, é celebrado tipicamente entre outubro e novembro para representar o triunfo do bem sobre o mal. Durante essa comemoração, casas e ruas são enfeitadas com lâmpadas, velas e lanternas. As pessoas se encontram para trocar presentes, compartilhar doces e realizar rituais religiosos em homenagem a diferentes divindades hindus.

Já o Sankranti, celebra a jornada do sol para um novo zodíaco e é festejado por toda a Índia sob diferentes nomes. Realizado em janeiro, coincide com a entrada do sol no signo de Capricórnio, marcando o término do inverno e o início da época de colheita. Durante esse período, as pessoas participam de cerimônias religiosas, fazem orações e preparam pratos típicos com os alimentos frescos da nova safra.


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

- Criar e disponibilizar Painel Power BI para os stakeholders.

## 3.1. Análise Comparativa

### 3.1.1. Tipos de Promoção

A Promoção BOGOF demonstrou um aumento significativo nas vendas e receitas incrementais, destacando sua eficácia em atrair clientes. Por outro lado, a promoção de 25% OFF resultou em redução nas vendas e receitas incrementais, enquanto a de 50% OFF teve um impacto ainda menor que a de 33% OFF. A promoção 'Ganhe 500 de Cashback' parece ser a mais eficaz em aumentar vendas e receitas incrementais

### 3.1.2. Categorias

Analisando as categorias e suas campanhas, destacamos o seguinte:

- Mercearia e alimentos básicos (Grocery and Staples): Na semana anterior à semana promocional, observou-se um aumento significativo tanto na quantidade vendida quanto na receita gerada, especialmente durante a festividade Sankranti. Os produtos pertencentes a esta categoria incluem lentilha, arroz, farinha de trigo e óleo de girassol.

- Eletrodomésticos (Home Appliances): Durante a festividade Sankranti, observou-se um aumento, embora menos expressivo, quando comparado a festividade Diwali, tanto na quantidade vendida quanto na receita gerada, com produtos como lâmpadas e aquecedores.

- Cuidados Domiciliares (Home Care): Na semana Sankranti, a quantidade vendida aumentou, porém a receita teve uma leve variação, com produtos como lençóis, cortinas, recipientes e esponjas de louça.

- Cuidado Pessoal (Personal Care): Na última campanha, esta categoria apresentou o menor desempenho em ambas as métricas, com produtos como hidratantes e sabonetes.


### 3.1.3. Produtos

Ao examinar o desempenho das promoções quanto aos produtos, é possível categorizar os resultados em duas estratégias distintas: 

- Na primeira estratégia, encontram-se os produtos que foram promovidos da mesma maneira em ambas as campanhas, ou seja, utilizando o mesmo tipo de promoção. Nessa categoria, alguns produtos mostraram um sucesso consistente, com resultados ainda melhores na última campanha. Outros mantiveram um desempenho positivo, embora possam ter sido influenciados por outros fatores. Por outro lado, alguns produtos enfrentaram desafios, registrando um desempenho negativo em ambas as campanhas.


- Na segunda estratégia, os produtos tiveram suas estratégias de venda modificadas entre as campanhas, incluindo uma mudança no tipo de promoção e um aumento no preço dos produtos. No primeiro cenário, houve um aumento nas vendas na última campanha; no segundo, houve uma queda em comparação com a anterior.


<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vRfE6KWh1k-cediMzdDAecFBiH0Z-WgyYy5FVbln4TT2v7UrQbCj0b6LmArYetqxGAGIsNHMTW3JOvL/embed?start=true&loop=true&delayms=3000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>



### 3.1.4. Cidades

Sabe-se que quanto às festividades, Diwali é amplamente celebrado em toda a Índia, sem restrições geográficas, enquanto Sankranti tem sua influência mais concentrada nos estados do sul da Índia.  No total, há registros de vendas em 10 cidades indianas. Abaixo, destacam-se as top 3:


- Em Bangalore (Bengaluru), a capital de Karnataka, localizada no sul do estado, os resultados foram notáveis durante as análises das campanhas:
    - Durante a campanha Diwali, registrou-se um incremento de 70% nas quantidades vendidas, resultando em uma receita de 23,8 milhões.
    - Na campanha Sankranti, houve um aumento de 164% nas quantidades vendidas, alcançando uma receita de 12,1 milhões.

- Chennai, a capital e a maior cidade do estado de Tamil Nadu, encontra-se na costa leste do sul, foi a segunda cidade com melhores resultados:
    - Durante a campanha Diwali, registrou-se um incremento de 66% nas quantidades vendidas, resultando em uma receita de 18,8 milhões.
    - Na campanha Sankranti, houve um aumento de 162% nas quantidades vendidas, alcançando uma receita de 9,8 milhões.
    
- Hyderabad, a capital do estado de Telangana, situada na região central-sul, foi com a terceira com melhores resultados:
    - Durante a campanha Diwali, registrou-se um incremento de 63% nas quantidades vendidas, resultando em uma receita de 14,3 milhões.
    - Na campanha Sankranti, houve um aumento de 146% nas quantidades vendidas, alcançando uma receita de 7,3 milhões.

<img src="https://imgur.com/gDw4kc8.png" width="100%"/>


### 3.1.5. Lojas

Um total de 50 lojas participaram das promoções. Abaixo estão listadas, em ordem decrescente, as três principais lojas com os melhores resultados:

- Durante a campanha do Diwali, a **loja STMYS-1**, localizada na cidade de Mysuru, registrou um incremento de 93% nas quantidades vendidas, resultando em uma receita de 3 milhões. Na campanha do Sankranti, houve um aumento significativo de 187% nas quantidades vendidas, alcançando uma receita de 1,6 milhões.

- Durante a campanha do Diwali, a **loja STCHE-4**, situada em Chennai, observou um aumento de 82% nas quantidades vendidas, gerando uma receita de 3,1 milhões. Na campanha do Sankranti, as quantidades vendidas aumentaram em 179%, resultando em uma receita de 1,4 milhões.

- Durante a campanha do Diwali, a loja **STBLR-0,** localizada em Bengaluru, viu um aumento de 79% nas quantidades vendidas, o que gerou uma receita de 3 milhões. Na campanha do Sankranti, as quantidades vendidas aumentaram significativamente em 192%, resultando em uma receita de 1,4 milhões.



# 4.0 O Produto Final do Projeto

Painel no Power BI com métricas e análises claras e autoexplicativas, facilitando a compreensão dos usuários.


# 5.0. Conclusão

- Quando substituir a oferta promocional de uma campanha por outra, priorizar a troca pela promoção BOGOF.
- A promoção de 25% OFF teve pouco efeito em motivar os clientes a comprar produtos.
- Explorar a criação de novos combos que incorporem a promoção 500 Cashback.
- Os motivos por trás da seleção dos produtos em promoção não estão explícitos, no entanto, é crucial compreender historicamente quais itens apresentam uma alta demanda durante festividades. Contudo, parece que os produtos oferecidos estão alinhados com o espírito das duas celebrações. 
    - Vale ressaltar a mudança na promoção da farinha de trigo e do óleo de girassol, que passaram de um desconto de 25% para uma oferta de "leve dois, pague um" (BOGOF), juntamente com um aumento de preço entre uma campanha e outra. Como já mencionado, durante o festival Sankranti, a demanda por esses produtos pode ser significativa. Assim, essa mudança de estratégia parece ter sido acertada, resultando em ganhos consideráveis.
