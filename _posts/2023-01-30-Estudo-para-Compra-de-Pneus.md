---
title: "Compra de Pneus"
author: "Ramon Roldan"
date: "2023-02-16"
categories: [Data Science, Tydeverse]
tags: [Tydeverse, Web Scrapping, Analitycs, Dashboard, Data Science, ETL]
---

## Principal Objetivo

Este estudo busca utilizar técnicas de Data Science juntamente com strategic sourcing e metodologias ageis para sugerir a melhor opção de compra de pneus conforme opções do mercado varejista online.

O produto que procuramos é um pneu aro 14 e será escolhido conforme melhor nota no indicador aqui criado: "Nota_Conceitual".

## Motivação do Estudo

Recentemente tive a felicidade de comprar meu primeiro carro, um lindo Ford Ka branco modelo 2017, mas como todo carro de segunda mão sempre precisamos arrumar alguma coisa.

Orientado para cultura de data driven, e muita experiencia na área de suprimentos, decidi realizar uma pesquisa de mercado.

## Organização do Trabalho

Buscando eficiência divideremos o trabalho em duas grandes partes:

1- Consultar o site do inmetro para: \* Analisar quais critérios a organização reguladora considera como essenciais. \* Esta etapa também visa entender quem são os principais fornecedores e principais produtos.

A ideia é não deixarmos de fora algum produto e/ou fornecedor de boa qualidade, mas com pouca relevancia no mercado varejista online.

2- Realizar uma consulta de mercado em todos os produtos disponíveis no site pneus store, que se enquadrem nas caractéristicas do produto que procuramos, para entender as condições comerciais como preço, prazo de entrega e valor do frete.

**Principais Etapas do Estudo**:

-   Definição do principal objetivo a ser alcançado;
-   Detalhes sobre a Motivação deste Estudo;
-   Versionar Projeto no GitHub;
-   Detalhes sobre as fontes de dados;
-   Coleta dos dados;
-   Realizar análise diagnostica;
-   Realizar analise Inferencial

### Versionar projeto no GitHub

O trabalho foi desenvolvido usando um "Projeto" dentro da IDE Rstudio, e também criamos um repositorio para ajudar no armazenamento e versionamento de todos os arquivos do projeto.

![Imagem do Repositório](/assets/img/compra_pneus/print_github.png)

Adotamos a ferramenta Kanban para organizar as atividades do projeto no formato de metodologia ágil:

![Imagem do Kanban](/assets/img/compra_pneus/kanbam.png)

## Detalhes sobre a Fonte de Dados

### Informações do Inmetro

O Instituto Nacional de Metrologia, Qualidade e Tecnologia [Inmetro](https://dados.gov.br/dados/conjuntos-dados/programa-brasileiro-de-etiquetagem-pbe) é uma autarquia federal vinculada ao Ministério da Economia. Sua missão institucional é prover confiança à sociedade brasileira nas medições e na qualidade dos produtos, por meio da Metrologia e da Avaliação da Conformidade, promovendo a harmonização das relações de consumo, a inovação e a competitividade do País.

Neste trabalho utilizaremos os dados abertos do Programa Brasileiro de Etiquetagem (PBE), disponibilizados pelo governo federal, para auxilar na escolha pautada em dados:

![Imagem da Etiqueta](/assets/img/compra_pneus/pneu.png)

Este [Video](https://www.youtube.com/watch?v=867SbL5RulU) didatico explica com maior riqueza de detalhes o significado dos códigos da etiqueta e como podemos interpreta-as para ajudar no esclarecimento.

### Loja Pneu Store

A [Pneu Store](https://www.pneustore.com.br/) é uma loja online especializada na comercialização de pneus para veiculos em todo o territorio Brasileiro.

Este site é referência de mercado por apresentar detalhes técnicos dos produtos, bem como preços competitivos de marcas renomeadas, e boas avaliações no reclame aqui.

Vale a pena destacar que tive indicações tanto de alguns profissinais da área, mecânicos, quanto de amigos meus, consumidores finais, que compram e ainda não tiveram problema.

No [video](https://www.youtube.com/watch?v=QIWJH8xKxcM) tem todo um estudo detalhado sobre confiabilidade do site.

![Imagem do site Pneu Store](/assets/img/compra_pneus/print_pneustore.png)

## Coleta dos Dados

Para realizar nosso estudo iremos aplicar uma técnica de data science chamada web scrapping que consiste em pegar as informações online da url.

Utilizaremos a linguagem R como backend e framework proposto pela metodologia do tidyverse dando preferência para a biblioteca rvest.

### Dados do site Pneus Store


```r
#Primeiro vamos começar limpando o global environment
rm(list = ls())
#Utilizaremos a biblioteca tidyverse para o tratamento, manipulação e vizualização de dados.
library(tidyverse)
#Utilizaremos a biblioteca Janitor para nos ajudar no saneamento e tratamento dos dados
library(janitor)
#A biblioteca rvest será utilizada para realizar web scrapping das informações disponíveis.
library(rvest)
#Para realizar a análise exploratória e apresentação gráfica também usaremos o pacote DataExplorer
library(DataExplorer)

#Dentro do site pneu store o caminho dos pneus aro 14 estão salvo em 4 páginas
urls <- paste0("https://www.pneustore.com.br/categorias/pneus-de-carro/175-65-r14?q=%3Arelevance&page=", 0:4)

#A variavel "Links" vai realizar um looping entre as 4 páginas e salvar a url de cada pneu em uma lista
links <-urls %>% 
  map(read_html) %>% 
  map( ~tibble(link = html_attr(html_nodes(.x, "a"), "href")) %>%
  filter(str_detect(link, "/categorias/pneus-de-carro/pneus-175-65r14/")) %>%
  distinct() %>%
  mutate(link = str_c("https://www.pneustore.com.br", link))) %>% bind_rows()

#A variavel bases_pneus_store irá armazenar os dados de todos os produtos disponíveis por página
bases_pneus_store <- links %>%
  mutate(base = map(link, ~ read_html(.x) %>% html_node("table") %>% html_table() %>% as_tibble())) %>%
  unnest(cols = base) %>% rename('variavel' = 'X1', 'resultado' = 'X2') %>% 
  pivot_wider(names_from = variavel,values_from = resultado) %>% janitor::clean_names() %>%
  mutate(nome = map(link, ~ read_html(.x) %>% html_nodes('h1') %>% html_text2()),
         resistencia_ao_rolamento = map(link, ~ read_html(.x) %>% html_node('.energy-efficiency') %>% html_attr('data-value')),
         aderencia_em_pista_molhada = map(link, ~ read_html(.x) %>% html_node('.water-adhesion') %>% html_attr('data-value')),
         ruido_externo = map(link, ~ read_html(.x) %>% html_node('.noise-level') %>% html_attr('data-value')),
         preco_a_vista = map(link, ~ read_html(.x) %>% html_nodes('.price.my-2.font-black') %>% html_text2() %>% parse_number(locale = locale(decimal_mark = ","))),
         preco_parcelado = map(link, ~ read_html(.x) %>% html_node(xpath = '//b') %>% html_text2() %>% parse_number(locale = locale(decimal_mark = ","))),
indice_de_peso = str_extract(indice_de_peso, "(?<=- )[^ ]*") %>% parse_number(locale = locale(decimal_mark = ",")),
indice_de_velocidade = str_extract(indice_de_velocidade, "(?<=- )[^ ]+(?= km)") %>% parse_number(locale = locale(decimal_mark = ",")))
```

### Análise Exploratória (EDA)

Analisando a base descobrimos que contém 36 variáveis e 69 modelos de pneus. Também é possível notar que nossa base contém diferentes tipos primitivos de dados:


```r
bases_pneus_store %>% glimpse()
```

```
## Rows: 69
## Columns: 36
## $ link                       <chr> "https://www.pneustore.com.br/categor…
## $ marca                      <chr> "PIRELLI", "CONTINENTAL", "FORMULA", …
## $ modelo                     <chr> "CINTURATO P1", "CONTIPOWERCONTACT", …
## $ medida                     <chr> "175/65R14", "175/65R14", "175/65R14"…
## $ largura                    <chr> "175mm", "175mm", "175mm", "175mm", "…
## $ perfil                     <chr> "65%", "65%", "65%", "65%", "65%", "6…
## $ aro                        <chr> "14", "14", "14", "14", "14", "14", "…
## $ diametro_total_em_mm       <chr> "583.1", "583.1", "583.1", "583.1", "…
## $ indice_de_peso             <dbl> 475, 475, 475, 475, 475, 475, 475, 47…
## $ indice_de_velocidade       <dbl> 190, 190, 190, 190, 190, 190, 210, 19…
## $ rft_run_flat               <chr> "NÃO", "NÃO", "NÃO", "NÃO", "NÃO", "N…
## $ tipo_de_construcao         <chr> "RADIAL", "RADIAL", "RADIAL", "RADIAL…
## $ peso                       <chr> "6.78", "6.795", "6.572", "6.351", "6…
## $ extra_load                 <chr> "NÃO", "NÃO", "NÃO", "NÃO", "NÃO", "N…
## $ protetor_de_bordas         <chr> "NÃO", "NÃO", "NÃO", "NÃO", "NÃO", "N…
## $ sidewall                   <chr> "BSW LETRAS PRETAS", "BSW LETRAS PRET…
## $ tipo_de_terreno            <chr> "HT", "HT", "HT", "HT", "HT", "HT", "…
## $ desenho                    <chr> "Assimétrico", "Assimétrico", "Assimé…
## $ tala_da_roda               <chr> "5", "5", NA, NA, NA, NA, "5.0", NA, …
## $ tala_possiveis_da_roda     <chr> "5-6", "5-6", NA, NA, NA, NA, "6", NA…
## $ utqg                       <chr> "420AA", "480AB", "180AB", "200BB", "…
## $ treadwear                  <chr> "420", "480", "180", "200", "440", "4…
## $ tracao                     <chr> "A", "A", "A", "B", "B", "A", "A", NA…
## $ temperatura                <chr> "A", "B", "B", "B", "B", "A", "B", NA…
## $ registro_inmetro           <chr> "001387/2012", "001152/2015", "001387…
## $ garantia                   <chr> "5 anos Contra Defeito de Fabricação"…
## $ observacoes                <chr> "Produto novo,Imagem meramente ilustr…
## $ fabricante                 <chr> NA, NA, "PIRELLI", "BRIDGESTONE", "GO…
## $ tipo_de_montagem           <chr> NA, NA, NA, NA, NA, "SEM CÂMARA", NA,…
## $ profundidade_do_sulco      <chr> NA, NA, NA, NA, NA, NA, "7.5", NA, NA…
## $ nome                       <list> "Pneu Pirelli Aro 14 Cinturato P1 17…
## $ resistencia_ao_rolamento   <list> "C", "E", "E", "E", "E", "F", "E", "…
## $ aderencia_em_pista_molhada <list> "E", "C", "E", "E", "F", "E", "E", "…
## $ ruido_externo              <list> "MEDIUM", "MEDIUM", "HIGH", "MEDIUM"…
## $ preco_a_vista              <list> 359.9, 394.9, 329.9, 319.9, 349.9, 3…
## $ preco_parcelado            <list> 408.98, 11, 374.89, 363.52, 397.61, …
```

Analisando os dados descobrimos que existem 30 fornecedores diferentes e que alguns tem um portifolio de produtos mais variado que outros fornecedores.

![plot of chunk r](figure/r-1.png)

![](/assets/img/compra_pneus/qtd_pneus_por_marca.png)

Analisando os dados descobrimos que existem 49 modelos de pneus diferentes porque as variantes servem para atender necessidades diferentes, como desempenho em terrenos distintos e consumo de conbustivel.

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

![](/assets/img/compra_pneus/qtd_pneus_por_modelo.png)

### Seleção das Principais Variáveis

Avaliando qualidade dos dados descobrimos que algumas das 36 variaveis tem muitos dados faltantes:

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png)

![](/assets/img/compra_pneus/qtd_valores_vazios.png)

Avaliando maiores detalhes das variaveis com mais dados faltantes, vermelhas e roxas, entendemos que não são relevantes para o trabalho e por isso iremos desconsiderar as 6 variáveis: preco_parcelado, tala_possiveis_da_roda, talas_da_roda, fabricante, tipo_de_montagem, profundidade_do_sulco

Seguindo a sugestão do INMETRO entendemos que as variavéis resistencia_ao_rolamento, aderencia_em_pista_molhada, e ruido_externo são fundamentais:

![Programa Brasileiro de Etiquetagem (PBE)](/assets/img/compra_pneus/inmetro.png)

Com base em em pesquisa concluimos que 10 as principais variáveis principais para compor a nota_conceitual são:

1.  **Resistência ao Rolamento (Obrigatório)**: Está diretamente relacionada à eficiência energética, uma vez que mede a energia absorvida quando o pneu está rodando. Com isso, quanto menor for a resistência ao rodar, menor será o consumo de combustível e, consequentemente, menor será o impacto ao meio ambiente (emissão de CO 2 ). Na etiqueta, os pneus serão classificados em seis níveis, sendo A o mais eficiente e até F. [Fonte](https://www.anip.org.br/etiquetagem/);

2.  **Aderência em Pista Molhada (Obrigatório)**: É um indicador do desempenho que informa ao consumidor sobre a aderência do pneu em pistas molhadas. As escalas vão de A (melhor desempenho) até E, e abrange pneus para veículos de passeio e pesados. Essa classificação mede a distância percorrida pelo veículo após a frenagem quando a pista está molhada. [Fonte](https://www.anip.org.br/etiquetagem/);

3.  **Ruído Externo (Obrigatório)**: Indica o nível do ruído produzido pelos pneus em decibéis (dB) e, consequentemente, o impacto no meio ambiente. Este critério deve ter como limite máximo 75 dB para pneus de veículos de passeio, 77 dB para pneus de veículos comerciais leves e 78 dB para pneus de caminhões e ônibus. [Fonte](https://www.anip.org.br/etiquetagem/);

4.  **Tração (Obrigatório)**: É um índice baseado na capacidade do pneu parar um carro no asfalto ou concreto molhado. Não tem nada que ver com a habilidade do pneu fazer curvas. Há quatro categorias de tração, AA , A , B e C , AA sendo o mais alto e C o mais baixo. [Fonte](https://www.pneustore.com.br/informacao-tecnica-pneus);

5.  **Temperatura (Obrigatório)**: O índice de temperatura escrita no pneu indica a capacidade do pneu dissipar o calor e como lida com o acúmulo dele. Há três possíveis índices: A, B e C, A sendo o melhor e C o pior. O índice só se aplica a pneus inflados corretamente de acordo com o valor de pressão descrito no manual do carro. Inflação, excesso de velocidade ou excesso de peso, faz com que o pneu aqueça mais rapidamente, causando seu desgaste precoce e, possivelmente ocasionando falhas no desempenho. [Fonte](https://www.pneustore.com.br/informacao-tecnica-pneus);

6.  **Treadwear (Obrigatório)**: Este número pode variar de 60 a 700, e quanto maior o número, maior rendimento quilométrico terá o pneu. Por exemplo, um pneu Treadwear 400 deveria render duas vezes mais que um pneu com Treadwear 200. Vale lembrar, que nem sempre um pneu que dura mais é um pneu melhor, pois um pneu macio que tem maior aderência, poderá durar menos, mas oferecer um melhor desempenho em curvas e frenagens. [Fonte](https://www.pneustore.com.br/informacao-tecnica-pneus);

7.  **Índice de peso (Obrigatório)**: Para saber o peso máximo o seu pneu suporta, você também pode conferir na tabela de índices de carga. Você encontrará um destes números estampados no seu pneu depois da medida. Exemplo: na medida 205/70R16 112S, 112 é o número que designa o peso máximo por pneu, neste caso 112 representa 1120kg. [Fonte](https://www.pneustore.com.br/informacao-tecnica-pneus);

8.  **Preço (Opcional):**: Visando maior econômia quanto menor melhor.;

9.  **Registro Inmetro (Opcional):** Os pneus novos radiais de passeio, comerciais leves, caminhões e ônibus comercializados no mercado brasileiro, produzidos no Brasil ou importados, devem conter a etiqueta. [Fonte](https://www.anip.org.br/etiquetagem/);

10. **Índice de Velocidade (Opcional)**: Para encontrar a velocidade máxima que você pode dirigir com seu pneu, você pode consultar a tabela onde encontra todos os índices e velocidades respectivas. Você encontrará uma destas letras estampada no seu pneu depois da medida. Exemplo: na medida 215/45R17 100Y, Y é a letra que designa a velocidade máxima do pneu, neste caso Y representa 300 km/h na tabela. [Fonte](https://www.pneustore.com.br/informacao-tecnica-pneus);

11. **Extra Load (Opcional)**: Os pneus reforçados ou EXTRA LOAD (XL) destinam-se aos veículos pesados ou equipados com uma motorização potente. Os flancos dos pneus reforçados são mais rígidos do que os dos pneus clássicos designados por "SL" para "Standard Load". A rigidez dos flancos permite suportar uma carga, uma pressão e tensões mais elevadas. [Fonte](https://www.pneuslider.pt/pneus-reforcados).

### Avaliação Final

Para conseguirmos realizar a avaliação de trade-off comparando as melhores caracteristicas com o melhor preço, vamos utilizar a uma nota conceitual onde as variáveis tem pesos distintos e a nota final irá ponderar:


```r
  base_para_nota_conceitual<- bases_pneus_store %>% 
      select(nome,
             marca,
             resistencia_ao_rolamento,
             aderencia_em_pista_molhada,
             ruido_externo,
             tracao,
             temperatura,
             treadwear,
             indice_de_peso,
             registro_inmetro,
             indice_de_velocidade,
             preco_a_vista,
             preco_parcelado,
             link) %>% 
    mutate(
      nota_resistencia_ao_rolamento = case_when(
      resistencia_ao_rolamento == "A" ~ 7,
      resistencia_ao_rolamento == "B" ~ 6,
      resistencia_ao_rolamento == "C" ~ 5,
      resistencia_ao_rolamento == "D" ~ 4,
      resistencia_ao_rolamento == "E" ~ 3,
      resistencia_ao_rolamento == "F" ~ 2,
      TRUE ~ 1),
      
      nota_aderencia_em_pista_molhada = case_when(
      aderencia_em_pista_molhada == "A" ~ 7,
      aderencia_em_pista_molhada == "B" ~ 6,
      aderencia_em_pista_molhada == "C" ~ 5,
      aderencia_em_pista_molhada == "D" ~ 4,
      aderencia_em_pista_molhada == "E" ~ 3,
      aderencia_em_pista_molhada == "F" ~ 2,
      TRUE ~ 1),
      
      nota_ruido_externo = case_when(
      ruido_externo == "A" ~ 7,
      ruido_externo == "B" ~ 6,
      ruido_externo == "C" ~ 5,
      ruido_externo == "D" ~ 4,
      ruido_externo == "E" ~ 3,
      ruido_externo == "F" ~ 2,
      TRUE ~ 1),
      
      nota_tracao = case_when(
      tracao == "AA" ~ 4,
      tracao == "A" ~ 3,
      tracao == "B" ~ 2,
      TRUE ~ 1),
      
    nota_temperatura = case_when(
      temperatura == "A" ~ 3,
      temperatura == "B" ~ 2,
      TRUE ~ 1),

    nota_treadwear = case_when(
      treadwear >= 60 & treadwear <= 200 ~ 1,
      treadwear >= 201 & treadwear <= 400 ~ 2,
      TRUE ~ 3),

    nota_indice_de_peso = case_when((indice_de_peso*4) <1007 ~ 0,
                                indice_de_peso <= 462 ~ 1,
                                indice_de_peso > 462 & indice_de_peso <= 475 ~ 2,
                                TRUE ~ 3),
    nota_registro_inmetro = case_when( is.na(registro_inmetro) ~ 0,
                                   TRUE ~1),

    nota_indice_de_velocidade = case_when(indice_de_velocidade <= 190 ~ 0,
                                      TRUE ~ 1),
    nota_extra_load = case_when(indice_de_velocidade == "SIM" ~ 1,
                                      TRUE ~ 0),
    nota_conceitual = (nota_resistencia_ao_rolamento * .7) + (nota_aderencia_em_pista_molhada * .7) + (nota_ruido_externo * .7) + (nota_tracao * .7) + (nota_temperatura * .7) + (nota_treadwear *.7) +
      (nota_indice_de_peso * .7) + (nota_registro_inmetro * .3) + (nota_indice_de_velocidade * .3) + (nota_extra_load * .3)
      
    ) %>% arrange(desc(nota_conceitual))

#Printa as 10 primeiros produtos com melhor desempenho na nota conceitual
base_para_nota_conceitual  %>% unnest()
```

```
## # A tibble: 22 × 25
##    nome       marca resis…¹ adere…² ruido…³ tracao tempe…⁴ tread…⁵ indic…⁶
##    <chr>      <chr> <chr>   <chr>   <chr>   <chr>  <chr>   <chr>     <dbl>
##  1 Pneu Dyna… DYNA… E       C       MEDIUM  A      A       420         530
##  2 Pneu Mich… MICH… E       C       MEDIUM  A      A       420         475
##  3 Pneu Pire… PIRE… C       E       MEDIUM  A      A       420         475
##  4 Pneu Pire… PIRE… C       E       MEDIUM  A      A       420         475
##  5 Jogo 4 Pn… PIRE… C       E       MEDIUM  A      A       420         475
##  6 Pneu Cont… CONT… E       C       MEDIUM  A      B       480         475
##  7 Pneu Form… FORM… E       E       HIGH    A      B       180         475
##  8 Jogo 4 pn… KUMHO E       E       HIGH    A      B       460         475
##  9 Pneu Kumh… KUMHO E       E       HIGH    A      B       460         475
## 10 Jogo 4 Pn… FORM… E       E       HIGH    A      B       180         475
## # … with 12 more rows, 16 more variables: registro_inmetro <chr>,
## #   indice_de_velocidade <dbl>, preco_a_vista <dbl>,
## #   preco_parcelado <dbl>, link <chr>,
## #   nota_resistencia_ao_rolamento <dbl>,
## #   nota_aderencia_em_pista_molhada <dbl>, nota_ruido_externo <dbl>,
## #   nota_tracao <dbl>, nota_temperatura <dbl>, nota_treadwear <dbl>,
## #   nota_indice_de_peso <dbl>, nota_registro_inmetro <dbl>, …
```

# Considerações Finais

Com base nas caractéristicas técnicas elencadas o produto que melhor atenderia nossas necessidades é "Pneu Dynamo Aro 14 MH01 175/65R14 86T".

Sobre a parte financeira escolhemos pagar à vista porque se fossemos parcelar em 11 vezes o preço sairia mais caro, o que não seria compensado deixando num investimento rendendo 100% o CDI, e pagando por meio de PIX conseguimos um cupom de desconto. Com isso conseguimos o preço unitário final de R$ 200 trazendo um saving de 60% em relação ao encontrado se não tivessemos realizado o estudo.

Estou muito satisfeito com a compra e inclusive os pneus chegaram em casa.

![Imagem dos Pneus que chegaram em Casa](/assets/img/compra_pneus/pneus_chegaram.jpg)

**Premissas**: Este trabalho foi realizado com a linguagem R, IDE Rstudio, com Quarto, e sistema operacional Linux Mint. Foram utilizados conhecimentos de data science e metodologias ágeis. Seguindo as boas práticas do mercado demos preferencia para bibliotecas do tidyverse.

![Rstudio](https://cdn.jsdelivr.net/gh/devicons/devicon/icons/rstudio/rstudio-original.svg){alt="Ramon-Rstudio" align="center" width="40" height="30"} ![Quarto](https://quarto.org/quarto.png){alt="Ramon-quarto" align="center" width="60" height="30"}
![Linux](https://linuxmint.com/web/img/logo-mono.svg){alt="Ramon-Mint" align="center" width="80" height="70"}

![Tidyverse](https://raw.githubusercontent.com/rstudio/hex-stickers/master/SVG/tidyverse.svg){alt="Ramon-tidyverse" align="center" width="40" height="30"}

![Metodologias Agéis](https://tse1.mm.bing.net/th?id=OIP.YQHMHRrHb3almjchEGIknQHaE8){alt="Ramon-metodos" align="center" width="40" height="30"}

# Observações

Este artigo tem finalidade de estudo empirico pessoal e não é recomendação de compra e/ou venda, caso tenha alguma dúvida técnica procure um mecânico de sua confiança. Um ponto importante em se destacar é que os preços e disponibiidade de estoque estão sujeitos à mudança dinâmica do mercado varejista, e que as especificações técnicas também podem sofrer alterações conforme novos decretos da agência regulamentadora.