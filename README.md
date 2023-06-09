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
</div>


<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
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
      <a href="#web-scraping-e-tratamento-dos-dados">Web Scraping e Tratamento dos Dados</a>
      <ul>
        <li><a href="#web-scraping">Web Scraping</a></li>
      </ul>  
    </li>
    <li><a href="#agradecimentos">Agradecimentos</a></li>
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



## Web Scraping e Tratamento dos Dados

Os dados fornecidos pela Prefeitura estão organizados em páginas separadas para cada trimestre e ano, tanto na interface do usuário como na API. Para simplificar o processo de extração, foi desenvolvido um script em Python que será explicado detalhadamente a seguir. Esse script utiliza as bibliotecas Requests, Beautiful Soup, Pandas (incluindo StringIO) e chardet para concatenar os dados em um único conjunto de dados. Em seguida, os dados passam por um processo de tratamento para atender aos requisitos de negócios, reduzindo seu tamanho e otimizando o processamento no Power BI.

### Web Scraping

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
               if int(link["title"][-4:]) >= 2021:
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
Após analisar a estrutura dos dados disponíveis, observamos que os conjuntos de dados a partir de 2021 utilizam o ponto e vírgula ";" como delimitador, enquanto os anteriores utilizam a vírgula ",". Com base nessa observação, foi implementada uma condição para verificar a data do conjunto de dados. Utilizando o final do atributo "title" de cada dataset em uma condição, é possível identificar a qual data ele pertence e, assim, determinar qual delimitador deve ser utilizado durante o processamento.

   ```py
   if int(link["title"][-4:]) >= 2021:
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

### Transformação dos Dados

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
    
3. <b>Remover redundâncias no "Tipo de Serviço"</b>:

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
