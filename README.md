
# Introdução

O respositório tem como finalidade armazenar os scripts, dados, figuras
e tabelas criados para a pesquisa de Jonathan Morais Barcellos Ferreira
e Gabriel Delias de Sousa Simões para o [II Congresso Internacional
“Dignidade Humana em tempos de (pós) pandemia: direito e democracia no
Brasil
contemporâneo”](https://www.even3.com.br/ii-congresso-internacional-dignidade-humana-em-tempos-de-pandemia-direito-e-democracia-no-brasil-contemporaneo-316015/).

O artigo-produto da pesquisa foi submetido para publicação e o preprint
pode ser obtido em: [link]()

Os dados dessa pesquisa podem ser reutilizados desde que citada a fonte.

# Coleta dos dados

Os dados foram extraídos a partir da raspagem do site do Supremo
Tribunal Federal utilizando das funções do pacote
[decJ](https://github.com/johnmbf/decJ). O pacote foi desenvolvido para
facilitar as pesquisas envolvendo o Supremo Tribunal Federal. Foram
utilizadas das seguintes funções.

|         Função          |                                Descrição                                |
|:-----------------------:|:-----------------------------------------------------------------------:|
|   `extrairSTF.info()`   | Extrai as informações do processo, como data do ajuizamento e o assunto |
|  `extrairSTF.partes()`  |  Extrai as partes do processo, como o requerente, intimado, advogados   |
| `extrairSTF.relator()`  |                      Extrai o relator do processo                       |
| `extrairSTF.decisoes()` |                     Extrai as decisões do processo                      |

As funções apresentadas na tabela acima recebem três argumentos:
`lista`, `classe` e `n`. O argumento `lista` deve ser um objeto do tipo
`list` onde serão armazenados os dados que retornarem da função. A
escolha por incluir esse argumento é para caso o pesquisador já tenha
construído uma lista antes com outras ações, pode complementar essa
lista aplicando na função. O argumento `classe` deve ser um `character`
e se refere à classe processual e o `n` ao número do processo que se
pretende buscar. O argumento `n` pode ser um número apenas como
`n = 800` ou pode receber vários números, como `n = 800:900`.

O objeto dessa pesquisa é as arguições de descumprimento de preceito
fundamental ajuízadas entre 2019 e 2022. Portanto, `classe = 'ADPF'` e
`n = 561:1039`. O código abaixo demonstra a raspagem de dados utilizando
das funções e aplicando os parâmetros.

``` r
# Coleta da aba informações ----
lista_info <- list()
lista_info <- decJ::extrairSTF.info(lista_info, 'ADPF', 561:1039)
tabela_info <- dplyr::bind_rows(lista_info)

# Coleta na aba partes ----
lista_partes <- list()
lista_partes <- decJ::extrairSTF.partes(lista_partes, 'ADPF', 561:1039)
tabela_partes <- dplyr::bind_rows(lista_partes)

# Coleta na aba relator ----
lista_relator <- list()
lista_relator <- decJ::extrairSTF.relator(lista_relator, 'ADPF', 561:1039)
tabela_relator <- dplyr::bind_rows(lista_relator)

# Coleta aba decisões ----
lista_decisoes <- list()
lista_decisoes <- decJ::extrairSTF.decisao(lista_decisoes, 'ADPF', 561:1039)
tabela_decisoes <- dplyr::bind_rows(lista_decisoes)
```

As listas com os dados obtidos foram transformadas em tabelas. Essas
tabelas depois foram salvas em um arquivo de backup a fim de evitar que
o código precise ser rodado toda vez que fosse necessário buscá-los[^1].

``` r
# Salvar os arquivos ----
saveRDS(tabela_info, 'DATA/raw_info.rds')
saveRDS(tabela_partes, 'DATA/raw_partes.rds')
saveRDS(tabela_relator, 'DATA/raw_relator.rds')
saveRDS(tabela_decisoes, 'DATA/raw_decisoes.rds')
```

Com isso, ao trabalhar com os próximos scripts, os dados brutos são
carregados, sem a necessidade de executar a raspagem novamente.

``` r
# Carrega os dados ----
tabela_info      <- readRDS('DATA/raw_info.rds')
tabela_partes    <- readRDS('DATA/raw_partes.rds')
tabela_relator   <- readRDS('DATA/raw_relator.rds')
tabela_decisoes  <- readRDS('DATA/raw_decisoes.rds')
```

Os dados brutos contém as seguintes informações:

|     Objeto      | Variáveis   | Descrição                                                                |
|:---------------:|-------------|--------------------------------------------------------------------------|
|   tabela_info   | ADPF        | Número da ADPF                                                           |
|   tabela_info   | Ajuizamento | Data do ajuizamento da ADPF                                              |
|   tabela_info   | Assunto     | O(s) assunto(s) da ADPF agrupado(s)                                      |
|  tabela_partes  | ADPF        | Número da ADPF                                                           |
|  tabela_partes  | Tipo        | O tipo (natureza) da parte (requerente, advogado, intimado, por exemplo) |
|  tabela_partes  | Parte       | O nome da parte                                                          |
| tabela_relator  | ADPF        | Número da ADPF                                                           |
| tabela_relator  | Relator     | O relator do processo                                                    |
| tabela_decisoes | ADPF        | O número da ADPF                                                         |
| tabela_decisoes | Data        | A data de julgamento da decisão                                          |
| tabela_decisoes | Nome        | O nome(tipo) decisao                                                     |
| tabela_decisoes | Julgador    | O órgão/ministro que julgou a decisao                                    |
| tabela_decisoes | Decisão     | Um resumo do dispositivo da decisão                                      |

Os dados extraídos ainda não podem ser totalmente aproveitados e
precisaram passar por um processo de tratamento. No entanto, antes do
tratamento, três ADPFs foram removidas das análises:

| ADPF | Motivo                                                                      |
|------|-----------------------------------------------------------------------------|
| 859  | O processo possui segredo de justiça                                        |
| 920  | A ação foi protocolada como ADPF equivocadamente e o processo foi reautuado |
| 943  | A ação foi protocolada como ADPF equivocadamente e o processo foi reautuado |
| 1022 | A ação foi protocolada como ADPF equivocadamente e o processo foi reautuado |

``` r
# Remover as ADPFs ----
tabela_info <- tabela_info |>
    dplyr::filter(              # ADPF não pode ser:
        ADPF != '859'  &        # 859 e
        ADPF != '920'  &        # 920 e
        ADPF != '943'  &        # 943 e
        ADPF != '1022'          # 1022
    )
tabela_partes <- tabela_partes |>
    dplyr::filter(              # ADPF não pode ser:
        ADPF != '859'  &        # 859 e
        ADPF != '920'  &        # 920 e
        ADPF != '943'  &        # 943 e
        ADPF != '1022'          # 1022
    )
tabela_relator <- tabela_relator |>
    dplyr::filter(              # ADPF não pode ser:
        ADPF != '859'  &        # 859 e
        ADPF != '920'  &        # 920 e
        ADPF != '943'  &        # 943 e
        ADPF != '1022'          # 1022
    )
tabela_decisoes <- tabela_decisoes |>
    dplyr::filter(              # ADPF não pode ser:
        ADPF != '859'  &        # 859 e
        ADPF != '920'  &        # 920 e
        ADPF != '943'  &        # 943 e
        ADPF != '1022'          # 1022
    )
```

[^1]: Cada função levou cerca de 50 minutos para completar a raspagem de
    dados, por essa razão decidimos salvar esses arquivos em `.rds`.
