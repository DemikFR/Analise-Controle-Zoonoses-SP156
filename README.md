<!-- PROJECT LOGO -->
<br />
<div align="center">
  <h1 align="center">Análise do Controle de Zooneses do SP156</h1>

  <p align="center">
    Análise dos Dados do portal SP156 da Prefeitura de SP com Power BI
  </p>
  <p align="center">
    Os dados usados se encontram na base de <a href="http://dados.prefeitura.sp.gov.br/dataset/dados-do-sp156">dados abertos da Prefeitura de SP</a>.
  </p>
  <h3 align=center>🔨Projeto ainda em desenvolvimento.🔨</h3><br>
</div>


<!-- TABLE OF CONTENTS -->
<details>
  <summary>Sumário</summary>
  <ol>
    <li>
      <a href="#sobre-o-projeto">Sobre o Projeto</a>
      <ul>
        <li><a href="#ferramentas">Ferramentas</a></li>
      </ul>
    </li>
    <li><a href="#iniciar-o-projeto">Iniciar o Projeto</a></li>
    <li><a href="#requisitos-de-negócios">Requisitos de Negócios</a></li>
    <li>
      <a href="#extração-e-pré-processamento">Extração e Pré-processamento</a>
      <ul>
        <li><a href="#extração-web-scraping">Extração (Web Scraping)</a></li>
        <li><a href="#pré-processamento">Pré-processamento</a></li>
        <li><a href="#armazenamento-dos-dados">Armazenamento dos Dados</a></li>
        <li><a href="#considerações-finais-do-processamento">Considerações Finais do Processamento</a></li>
      </ul>  
    </li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>



<!-- Sobre o Projeto -->
## Sobre o Projeto

A finalidade do uso dos dados é realizar análises de dados para identificar problemas de eficiência nos serviços de controle de zooneses prestados pela Prefeitura de São Paulo aos cidadãos, com o objetivo de promover a transparência, melhorar a qualidade dos serviços e otimizar os recursos públicos. Além disso, os dados serão utilizados para fins de aprimoramento pessoal em análise de dados, visando ao desenvolvimento profissional.

Para ser feita esta análise, foi necessário um processo de web scraping para automatizar a extração dos datasets, pois eles estão separados por datas e períodos de ano em cada página diferente, depois foi feito um processo ETL para adequação aos requisitos de negócios, antes de salvar os dados.



### Ferramentas

Para realizar este projeto, foi usado as seguintes ferramenta:


* [![Python][Python.py]][Python-url]
* [![Power-BI][Power-BI.pbix]][Power-BI-url]



<!-- Iniciar o Projeto -->
## Iniciar o Projeto

1. Clone este Repositório
   ```sh
   git clone https://github.com/DemikFR/Analise-Controle-Zoonoses-SP156.git
   ```
   
2. Instale as bibliotecas que serão usadas no projeto
   ```py
   pip install requests
   pip install BeautifulSoup
   pip install pandas
   pip install StringIO
   pip install chardet
   ```

3. Busque o link de download da base dados da Prefeitura de SP para ser usado conforme o script Request do "data_scraping" de Python.



## Requisitos de Negócios

Como mencionado anteriormente, o objetivo deste projeto é identificar os principais pontos de melhoria nos serviços prestados e os problemas que afetam a população de São Paulo, no caso, com base no período de 2012 ao último mês de 2022. Com base nisso, faremos algumas perguntas:

* Quais foram os maiores problemas?

* Em relação aos serviços prestados, quais foram os que tiveram mais casos?

* Quais distritos tiveram mais denúncias?

* Qual a quantidade de denúncias recebidas por ano?

* Qual a grandeza de denúncias de "Animais que transmitem doenças" e quais foram os casos que tiveram mais denúncias?

* Quais foram os distritos que mais tiveram denúncias de "Animais que transmitem doenças"?

* Quantos serviços foram finalizados ou não?

* No geral, é possível determinar se o serviço prestado pela Prefeitura de São Paulo é eficiente?

Com essas perguntas, já será possível mapear todo o processo de análise, incluindo o tratamento dos dados.



## Extração e Pré-processamento

Os dados fornecidos pela Prefeitura estão organizados em páginas separadas para cada trimestre e ano, tanto na interface do usuário como na API. Para simplificar o processo de extração, foi desenvolvido um script em Python que será explicado detalhadamente a seguir. Esse script utiliza as bibliotecas Requests, Beautiful Soup, Pandas (incluindo StringIO) e chardet para concatenar os dados em um único conjunto de dados. Em seguida, os dados passam por um processo de transformação para atender aos requisitos de negócios, reduzindo seu tamanho e otimizando o processamento no Power BI.

### Extração (Web Scraping)

Após ter estudado a estrutura HTML do site da Prefeitura, foi criada uma função para encontrar a tag <code>&lt;a&gt;</code> a partir de um padrão de texto no atributo "title" do HTML.

   ```py
   def get_soup(url, search):
       response = requests.get(url)
    
       # Verificar se a conexão foi bem sucedida
       if response.status_code == 200:
           soup = bs(response.content, 'html.parser')

           # Retornar as páginas onde terão as de download
           return soup.find_all('a', href=True, attrs={'title': re.compile(search)}) 
       else:
           return f'Página{link["title"]} está inacessível. Código: {response.status_code}'
   ```
   
A função recebe dois parâmetros: o primeiro é a URL na qual deve-se procurar a tag e o segundo é o padrão de texto que deve ser usado para localizar a tag. É importante ressaltar que o texto fornecido deve ser uma expressão regular (regex).

Após a criação do processo anterior, agora é possível implementar o web scraping. Para isso, foi desenvolvido um loop for que chama a função criada anteriormente e percorre a lista de links retornada por ela. Para cada link encontrado, é feita uma nova busca para localizar o link de download do conjunto de dados na segunda página. Nesse caso, todos os links que possuem a extensão .csv são considerados.

   ```py
   # URL base do site da Prefeitura
   url_base = 'http://dados.prefeitura.sp.gov.br'
   
   # URL inicial onde estarão os links para os datasets
   url_inicial = 'http://dados.prefeitura.sp.gov.br/dataset/dados-do-sp156'
   
   # extensão do dataset para filtrar
   file_type = '.csv'
   
   for link in get_soup(url_inicial, r'^Dados do SP156 -.*'):
       
       # Pegar o link de download
       for page in get_soup(url_base+link['href'], r'^http://dados.prefeitura.sp.gov.br/dataset/.*'):
           link_download = page['href']
           
           # Verificar se é o arquivo a ser baixado é Excel
           if file_type in link_download:
               
               # Pegar o conteúdo do CSV
               response_csv = requests.get(page['href'])
               
               # Cada datset tem um encoding diferente, por isso foi necessário usar a biblioteca chardet para identificar 
               # automaticamente qual é.
               encoding = chardet.detect(response_csv.content)['encoding']
               
               # Colocar os dados na tabela auxiliar
               if int(link["title"][-4:]) >= 2020:
                   df3 = pd.read_csv(StringIO(response_csv.content.decode(encoding)), sep=';', encoding=encoding, low_memory=False)
               else:
                   df3 = pd.read_csv(StringIO(response_csv.content.decode(encoding)), sep=',', encoding=encoding, low_memory=False)
                   
               dfs.append(df3)  # Adicionar o DataFrame intermediário à lista
               
               # Inserir os dados na planilha original
               print(f'Os dados do {link["title"]} foram inseridos com sucesso.')
   ```
  
Para extrair os dados do arquivo CSV, foi utilizado o método "get" da biblioteca Requests. O conteúdo é armazenado em uma variável para uso posterior.

Com o conteúdo do conjunto de dados, é possível utilizá-lo no chardet para detectar a codificação (encoding) apropriada. Isso ocorre porque os conjuntos de dados podem variar em sua codificação, alguns estão em Latin1 e outros em UTF-8. Além disso, alguns conjuntos de dados podem apresentar problemas com as codificações mais comuns. É importante lembrar que, para acessar o conteúdo do arquivo CSV, é necessário utilizar o atributo ".content" do objeto retornado pela biblioteca Requests.

   ```py
   response_csv = requests.get(page['href'])
   encoding = chardet.detect(response_csv.content)['encoding']
   ```
Após analisar a estrutura dos dados disponíveis, observamos que os conjuntos de dados a partir de 2020 utilizam o ponto e vírgula ";" como delimitador, enquanto os anteriores utilizam a vírgula ",". Com base nessa observação, foi implementada uma condição para verificar a data do conjunto de dados. Utilizando o final do atributo "title" de cada dataset em uma condição, é possível identificar a qual data ele pertence e, assim, determinar qual delimitador deve ser utilizado durante o processamento.

   ```py
   if int(link["title"][-4:]) >= 2020:
       df3 = pd.read_csv(StringIO(response_csv.content.decode(encoding)), sep=';', encoding=encoding, low_memory=False)
   else:
       df3 = pd.read_csv(StringIO(response_csv.content.decode(encoding)), sep=',', encoding=encoding, low_memory=False)
   ```
   
Para armazenar os dados, foi criado um dataframe auxiliar utilizando a função "read_csv" do Pandas. Dentro dessa função, é passado o conteúdo do conjunto de dados em questão. No entanto, é necessário utilizar a biblioteca "StringIO", conforme mostrado no código acima. Essa biblioteca é usada para tratar uma string (no caso, o conteúdo do arquivo CSV) como se fosse um arquivo real, permitindo que o Pandas possa reconhecê-lo corretamente.

Em seguida, o conteúdo do arquivo CSV, armazenado no dataframe auxiliar, é transferido para uma lista previamente criada. Essa abordagem visa melhorar o desempenho do Python durante o processamento dos dados.

   ```py
   dfs.append(df3)  # Adicionar o DataFrame intermediário à lista
   ```
   
Por fim, o conteúdo completo da lista é adicionado ao dataframe principal, que será utilizado em todo o processo de transformação.

   ```py
   df = pd.concat(dfs, ignore_index=True)
   ```

### Pré-processamento

Os dados disponibilizados pela Prefeitura contêm várias questões problemáticas, como a presença de 32 atributos e diversas categorias redundantes. Portanto, é necessário realizar um pré-processamento de acordo comos requisitos de negócio, já que sem realiza-lo, seria completamente inviável realizar qualquer tipo de análise. 

A seguir, serão enumeradas as ações realizadas para abordar essas questões:

1. <b>Filtrar os dados do assunto</b>:
  
    O conjunto de dados abrange diversos assuntos além da questão animal. Portanto, os dados foram previamente filtrados, o que garante um tempo de processamento mais    eficiente nos próximos passos, contribuindo para a agilidade e eficácia das etapas subsequentes.

    ```py
    df1 = df[df['Assunto'].str.contains('Animais', regex=False) | df['Tema'].str.contains(r'^Anim.*|^Pragas.*', regex=True)].reset_index(drop=True)
    ```

    É importante ressaltar que foram utilizadas as colunas "Assunto" e "Tema" para filtragem dos dados. A escolha dessas colunas se deu pelo fato de que os conjuntos de dados mais recentes não utilizam a coluna "Tema", apesar dela ser mais eficiente para esse propósito. Foi necessário empregar uma expressão regular na coluna "Assunto" para buscar qualquer ocorrência relacionada a animais ou pragas urbanas. Isso foi feito para garantir uma abrangência adequada na filtragem dos dados.
  

2. <b>Tratamento de atributos</b>:

    Após verificar todos os 32 atributos, foi mantido no dataframe apenas 9 que serão úteis na análise. Sendo assim foram filtrados conforme o código abaixo:

    ```py
    df = df[['Data de abertura', 'Tema', 'Assunto', 'Especificação do assunto', 'Serviço', 'Bairro', 'Distrito'
         , 'Status da solicitação', 'Data do parecer']]
    ```
    
    Ainda é possível reduzir a quantidade de colunas no dataframe, pois as colunas "Especificação do assunto" e "Serviço" possuem a mesma finalidade. Essa redundância ocorre devido aos conjuntos de dados mais recentes utilizarem apenas a coluna "Serviço". Portanto, foi decidido utilizar a abordagem mais recente. No caso em que a coluna "Serviço" está vazia (o que indica que os dados pertencem a conjuntos mais antigos), o seu valor será preenchido com o conteúdo da coluna "Especificação do assunto". Após esse processo, a coluna "Especificação do assunto" será removida do dataframe.
    
    ```py
    df1.loc[df1['Serviço'].isnull(), 'Serviço'] = df1.loc[df1['Serviço'].isnull(), 'Especificação do assunto']
    df1 = df1.drop('Especificação do assunto', axis=1)
    ```
    
    Não há necessidade de utilizar "Bairro" e "Distrito" juntos, pois ambos podem indicar a localização, apesar de haver essa divisão na cidade. Sendo assim, o "Bairro" será utilizado para preencher os valores vazios em "Distrito". Além disso, nos datasets de 2012 possuem códigos de distrito ao invés do nome, é possível saber qual distrito o código pertence através de uma tabela disponibilizada pela <a href="https://repositorio.seade.gov.br/dataset/codigos-nomes-dos-distritos-do-municipio-de-sao-paulo">Fundação Seade</a>.
    
    ```py
    df1.loc[df1['Distrito'].isnull() & df1['Bairro'].notnull(), 'Distrito'] = df1.loc[df1['Distrito'].isnull() & df1['Bairro'].notnull(), 'Bairro']
    df1 = df1.drop('Bairro', axis=1)
    
    # Após ler a tabela com os códigos e coloca-lo em um dataframe
    df1['Distrito'] = df1['Distrito'].replace(distrito.set_index('Nr Distrito')['Nome Distrito'])
    ```
    
3. <b>Tratamento do nome de atributos</b>:

    Para facilitar a compreensão de cada tabela, foi decidido mudar o nome da coluna "Assunto" para "Tipo de serviço", isso facilitará na compreensão dos dados, já que existe outro atributo como "Serviço" que é a descrição do "Assunto".
    
    ```py
    df2 = df1.rename({'Assunto': 'Tipo de Serviço'}, axis=1)
    ```
    
4. <b>Remover redundâncias no "Tipo de Serviço"</b>:

    Existe o "Tipo de Serviço" chamado "Animais que transmitem doenças ou risco à saúde", que abrange uma variedade de serviços relacionados a problemas como pernilongos, ratos e outros. No entanto, há outros tipos de serviço que poderiam se enquadrar nessa mesma categoria, mas estão separados, como "Animais/barata", "Animais/Aranha" e outros. Para solucionar isso, todos os "Tipos de Serviço" que contenham a expressão "Animais /" serão convertidos para "Animais que transmitem doenças ou risco à saúde". É importante ressaltar que existem outros tipos de serviço que também possuem a expressão "Animais /", mas não se enquadram na categoria de "Animais que transmitem doenças ou risco à saúde". Esses casos serão ignorados no código.
    
    ```py
    df2.loc[df2['Tipo de Serviço'].str.contains('Animais[ |/]') & df2['Tipo de Serviço']
        .str.match('^(?!.*(Cão|Gato|Cavalo|silvestres|RGA|agressor|via))'), 'Tipo de Serviço'
       ] = 'Animais que transmitem doenças ou risco à saúde'
    ```
    

    Para aprimorar a forma como os tipos de serviço estão sendo expressos, alguns deles passarão por um processo de simplificação. Por exemplo, o tipo de serviço "Animais / Cão" será apresentado apenas como "Cão" e "Vistoria / Animais" será mostrado somente como "Vistoria". Para realizar essa modificação, o seguinte código foi implementado:
    
    ```py
    df2['Tipo de Serviço'] = df2['Tipo de Serviço'].str.replace('Animais / ', '')
    df2['Tipo de Serviço'] = df2['Tipo de Serviço'].str.replace(' / Animais', '')
    ```


    O tipo de serviço "Dengue/chikungunya/zika (mosquito Aedes aegypti)" é classificado como um serviço de vistoria, de acordo com a descrição do serviço. Para lidar com esse caso específico, foi implementado o seguinte código:

    ```py
    df2['Tipo de Serviço'] = df2['Tipo de Serviço'].str.replace('Dengue/chikungunya/zika (mosquito aedes aegypti)', 'Vistoria')
    ```
    
5. <b>Remover redundâncias nos serviços de "Animais que transmitem doenças ou risco à saúde"</b>:

    Com o código a seguir, é possível visualizar todos os serviços classificados como "Animais que transmitem doenças ou risco à saúde" e identificar possíveis melhorias na forma como estão expressos:
    
    ```py
    df2.loc[df2['Tipo de Serviço'] == 'Animais que transmitem doenças ou risco à saúde', 'Serviço'].drop_duplicates().sort_values()
    ```
    
    Após análise, foi identificado que existem diversos serviços que estão separados, mas tratam do mesmo tema, como por exemplo: "Abelhas e Vespas", "Colmeia/Vespeiro instalado" e "Remoção de Abelhas, Vespas ou Marimbondos". Todos eles estão relacionados ao tema de abelhas, vespas e marimbondos. Diante disso, foi decidido manter o serviço "Remoção de Abelhas, Vespas ou Marimbondos" e unificar os outros dois. Essa abordagem será aplicada para outras inconsistências, conforme demonstrado no código abaixo:
    
    ```py
    df3.loc[(df3['Serviço'].str.contains('Pernilongo/Mosquito', regex=True)) | (df3['Serviço'] == 'Reclamação de Pernilongo'), 'Serviço'] = 'Reclamação de Pernilongos e Mosquitos'
    df3.loc[(df3['Serviço'] == 'Abelhas e Vespas') | (df3['Serviço'] == 'Colméia/Vespeiro instalado'), 'Serviço'] = 'Remoção de Abelhas, Vespas ou Marimbondos'
    df3.loc[df3['Serviço'].str.contains('Escorpião', regex=True), 'Serviço'] = 'Escorpiões'
    df3.loc[df3['Serviço'] == 'Reclamação de Escorpião', 'Serviço'] = 'Reclamação de Escorpiões'
    df3.loc[df3['Serviço'] == 'Ocorrências com morcego', 'Serviço'] = 'Reclamação de Morcegos'
    ```
    
    Existem alguns serviços que contêm palavras desnecessárias, as quais serão removidas para fins de padronização. Por exemplo, têm registros como "Pernilongo/Mosquito - Solicitar vistoria em local infestado" e "Pombos - Solicitar vistoria em local infestado", enquanto já existe "Reclamação de mosquitos" e "Reclamação de pombos", que são mais adequados para descrever os serviços oferecidos pela Prefeitura. Para solucionar essa questão, foi desenvolvido um código para substituir todos os serviços que possuem um hífen ("-") por "Reclamação de...", assim padronizando a nomenclatura dos serviços.
    
    ```py
    df3.loc[df3['Tipo de Serviço'] == 'Animais que transmitem doenças ou risco à saúde', 'Serviço'] = df3.loc[df3[
        'Tipo de Serviço'] == 'Animais que transmitem doenças ou risco à saúde', 'Serviço'].str.replace(r' -.*', '', regex=True)
        
    df3.loc[df3['Serviço'].str.split().str.len() == 1, 'Serviço'] = 'Reclamação de ' + df3['Serviço']
    ```
    
6. <b>Remover redundâncias nos serviços que não são "Animais que transmitem doenças ou risco à saúde"</b>:
    
    Diferente da última etapa, esta será para padronizar os serviços que não são "Animais que transmitem doenças ou risco à saúde", portanto o seguinte código foi executado:

    ```py
    pd.set_option('display.max_colwidth', None) # Poder ler todos os textos
    df3.loc[df3['Tipo de Serviço'] != 'Animais que transmitem doenças ou risco à saúde', 'Serviço'].drop_duplicates().sort_values()
    ```
    
    Foram identificados alguns caracteres incorretos, como espaços em branco ou pontos de interrogação que foram inseridos por engano. Para corrigir essa questão, o seguinte código foi executado:
    
    ```py
    df3['Serviço'] = df3['Serviço'].str.replace('?', '-')
    df3['Serviço'] = df3['Serviço'].str.replace('–', '-')
    df3['Serviço'] = df3['Serviço'].str.replace('Invadiu o local ', 'Invadiu o local')
    ```
    
    Durante o processo de tratamento dos caracteres incoerentes, foi realizada uma busca por redundâncias no conjunto de dados. Foram identificadas três categorias relacionadas a animais acidentados ou atropelados, todas contendo a mesma informação. Para padronizar esses registros, optou-se por alterar todas as ocorrências para "Atropelado ou Acidentado vivo e sem proprietário".

    Outra redundância encontrada foi em relação às categorias que tratam das condições de criação. As categorias "Condições de criação", "Denunciar condições inadequadas de criação" e "Condições de criação / maus tratos" representam o mesmo problema. Foi decidido manter a categoria "Condições de criação / maus tratos" por ser mais eficiente na expressão da situação.

    A terceira redundância diz respeito à remoção de animais mortos em vias públicas. A categoria "Remoção de animal morto em via pública" apresenta variações, como palavras no plural ou singular, ou até mesmo acréscimos de palavras que não fazem diferença no resultado final.

    A seguir, será apresentado o código responsável por realizar essas alterações:
    
    ```py
    df3.loc[df3['Serviço'].str.contains('Atropelado') | df3['Serviço'].str.contains('Acidentado'), 'Serviço'
           ] = 'Atropelado ou Acidentado vivo e sem proprietário'
    df3.loc[df3['Serviço'].str.match('.*ondições'), 'Serviço'
       ] = 'Condições de criação / maus tratos'
    df3.loc[df3['Serviço'].str.match('.*morto em via.*'), 'Serviço'
       ] = 'Remoção de animal morto em via pública'
    ```
    
    Com a execução desses códigos, as redundâncias foram eliminadas e as categorias foram padronizadas, resultando em dados mais consistentes.
    
    Durante a análise dos serviços, identificou-se que alguns deles não estão descritos de forma clara, como é o caso de "Solto em via pública". Para melhorar a compreensão desses serviços, foi desenvolvido um código que utiliza o "Tipo de Serviço" para complementar o texto. Dessa forma, o serviço será apresentado de maneira mais explicativa, como por exemplo "Cão Solto em via pública" no caso mencionado anteriormente.

    A seguir, é apresentado o código utilizado para realizar essa melhoria na descrição dos serviços:
    
    ```py
    condicoes = (df3['Serviço'] == 'Em parques') | (df3['Serviço'] == 'Invadiu o local') | (df3['Serviço'] == 'Solto em via pública')
    df3.loc[condicoes, 'Serviço'] = df3.loc[condicoes, 'Tipo de Serviço'].str.cat(df3.loc[condicoes, 'Serviço'], sep=' ')
    
    condicoes = (df3['Serviço'] == 'Diversas ocorrências') | (df3['Serviço'] == 'Ocorrências rotineiras')
    df3.loc[condicoes, 'Serviço'] = df3.loc[condicoes, 'Tipo de Serviço'].str.cat(df3.loc[condicoes, 'Serviço'], sep=' - ')
    ```
    
    É importante observar que, durante o processo, foram utilizados dois separadores distintos para diferentes casos. No primeiro caso, onde o objetivo era formar uma frase contínua, foi utilizado um único espaço como separador. Isso contribui para a legibilidade e fluidez do texto, seguindo as convenções gramaticais de espaçamento entre palavras.

    No segundo caso, em que os termos "diversos casos" e "Ocorrências rotineiras" atuam como agregadores, optou-se pelo uso de um hífen como separador. Essa escolha se baseia em uma consideração gramatical, onde o hífen é utilizado para unir termos que formam uma unidade semântica e representam uma única ideia.

    Dessa forma, ao utilizar o hífen como separador, está sendo indicado que "diversos casos" e "Ocorrências rotineiras" são elementos que estão relacionados e constituem um conjunto coeso.

7. <b>Capitalização dos Serviços (Capitalize)</b>:


    Diversos serviços apresentam inconsistências na capitalização das palavras, com algumas iniciando em maiúsculas e outras não. Além disso, há redundâncias devido à etapa 5. Para corrigir esses problemas, é possível utilizar o processo de capitalize. No entanto, é importante observar algumas regras, uma vez que alguns serviços utilizam "-" e "/" para separar termos, e também contêm nomes próprios que requerem todas as palavras em maiúsculas. Devido a essas particularidades, não foi possível utilizar diretamente o método <code>capitalize()</code> do Python. Portanto, foi necessário criar a seguinte função e aplicá-la:

    ```py
    # Função para realizar o capitalize com a regra
    def capitalize(servico):
        # Descobrir o sinal que a frase usa
        sinal = re.findall(r" - | / ", servico)
        
        if sinal:
            # Dividindo a frase em duas partes: antes e depois do traço ou da barra
            partes = re.split(sinal[0], servico)
            
            # Realizando o capitalize na primeira parte (antes do traço)
            parte1 = partes[0].capitalize()
    
            # Realizando o capitalize na segunda parte (depois do traço)
            parte2 = partes[1].capitalize()
            
            return "".join([parte1, sinal[0], parte2])
        else:
            return servico.capitalize()

    # Aplicação da função
    df3['Serviço'] = df3['Serviço'].apply(lambda x: capitalize(x) if not re.search(r'RGA|CMCA', x) else x)
    ```

    A função recebe o parâmetro "serviço" e realiza uma busca do "-" ou da "/" com uma expressão regex. Caso seja encontrado, o método "split" é aplicado para dividir o serviço com base no sinal encontrado. Em seguida, o método capitalize é executado em cada uma das partes resultantes. Por fim, o método join é utilizado para concatenar as duas partes capitalizadas, inserindo o sinal entre elas.

    O método apply do Pandas foi utilizado na coluna "Serviço", invocando a função por meio de uma função lambda. A função lambda só aplicará a função se a coluna não contiver nenhuma sigla, uma vez que siglas não devem ser modificadas.
  
     
9. <b>Formatação de datas</b>:

    Durante o processo de extração dos datasets, foi identificado que eles possuem três formatos diferentes para representar as datas. Esses formatos incluem apenas a data no formato "yyyy-mm-dd", uma combinação de data e hora no formato datetime com código e outra representação apenas como datetime.

    Com o objetivo de padronizar a representação das datas e facilitar a manipulação dos dados, foi tomada a decisão de realizar um processo de extração para manter somente o formato "yyyy-mm-dd" como data. Isso permitirá uma uniformidade nos registros e facilitará a análise e comparação dos dados ao longo do tempo.

    Ao adotar essa abordagem, será possível garantir consistência e coerência nos datasets, tornando-os mais acessíveis e compatíveis com as operações de processamento e análise que serão realizadas posteriormente no Power BI.
    
    ```py
    df4['Data de abertura'] = df4['Data de abertura'].str.slice(0,10)
    
    df4.loc[df4['Data do parecer'].notnull(), 'Data do parecer'] = df4.loc[
    df4['Data do parecer'].notnull(), 'Data do parecer'].str.slice(0,10)
    ```
    
    Por ter valores nulos (no caso os serviços que ainda não foram atendidos) a extração de data foi realizada apenas para os valores não nulos, conforme o código anterior.
    
Após a conclusão desse processo, torna-se evidente uma notável melhoria na consistência dos dados. Além disso, houve uma significativa redução no tamanho do conjunto de dados e na quantidade de registros. Essa redução é de extrema importância, uma vez que um volume excessivo de registros poderia dificultar ou até mesmo inviabilizar a análise dos dados.

Portanto, com o processo de redução e otimização finalizado, os dados agora estão prontos para serem explorados de forma mais eficiente e eficaz, possibilitando uma análise mais robusta e precisa.


### Armazenamento dos Dados

Para que os dados possam ser analisados no Power BI, é necessário armazená-los em um local apropriado. Neste caso, optamos por salvar os dados em formato Parquet compactado. Essa escolha foi baseada em um estudo prévio realizado com os mesmos dados em formato CSV, no qual foi constatado que o Parquet ocuparia 80.000 KB a menos em espaço de armazenamento em comparação ao CSV. Além disso, sem o pré-processamento realizado posteriormente, o tamanho ocupado em disco seria superior a 1 GB.

Portanto, para salvar os dados no formato Parquet compactado com compressão Gzip, foi utilizado o seguinte código:

  ```py
  df4.to_parquet('sp156_all_time.gzip',
              compression='gzip',
              index=False)
  ```


### Considerações Finais do Processamento

Embora tenha obtido sucesso em todo o processo de garantir a inclusão dos distritos na análise, é importante ressaltar que os distritos de denúncias anteriores a 2015 não devem ser utilizados. Isso ocorre porque os dados fornecidos pela Prefeitura de São Paulo para esses distritos não estão coerentes com os respectivos logradouros ou com os códigos de distrito da Fundação Seade que foram utilizados para a descrição. Portanto, recomenda-se excluir os dados dos distritos de denúncias anteriores a 2015 para evitar inconsistências na análise.
    
    
<!-- LICENSE -->
## License

Distributed under the BSD 2-Clause License. See `LICENSE.txt` for more information.



<!-- CONTACT -->
## Contact

Demik Freitas - [Linkedin](https://www.linkedin.com/in/demik-freitas/) - demik.freitast2d18@gmail.com

Project Link: [https://github.com/DemikFR/Analise-Controle-Zoonoses-SP156](https://github.com/DemikFR/Analise-Controle-Zoonoses-SP156)



<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[Python.py]: https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54
[Python-url]: https://www.python.org/
[Power-BI.pbix]: https://img.shields.io/badge/power_bi-F2C811?style=for-the-badge&logo=powerbi&logoColor=black
[Power-BI-url]: https://www.python.org/
