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

Após o processo anterior criado, já é possível criar o web scraping. Assim, foi criado um loop for que chama a função que foi criada anteriormente e percorre a lista que ela retorna 

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
