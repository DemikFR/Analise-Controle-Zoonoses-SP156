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
      <a href="#extração-e-tratamento-dos-dados">Extração e Tratamento dos Dados</a>
      <ul>
        <li><a href="#extração-e-tratamento-dos-dados">Extração e Tratamento dos Dados</a></li>
      </ul>  
    </li>
    <li>
      <a href="#análise-do-negócio">Análise do Negócio</a>
      <ul>
        <li><a href="#quais-foram-os-5-gêneros-mais-alugados">Quais foram os 5 gêneros mais alugados?</a></li>
        <li><a href="#existe-alguma-possibilidade-desses-gêneros-serem-os-mais-alugados-por-terem-mais-filmes">Existe alguma possibilidade desses gêneros serem os mais alugados por terem mais filmes?</a></li>
        <li><a href="#quais-foram-os-5-filmes-mais-alugados">Quais foram os 5 filmes mais alugados?</a></li>
        <li><a href="#quais-foram-os-5-filmes-menos-alugados">Quais foram os 5 filmes menos alugados?</a></li>
        <li><a href="#existe-algum-filme-que-não-foi-alugado">Existe algum filme que não foi alugado?</a></li>
        <li><a href="#por-ordem-decrescente-qual-foi-o-lucro-que-cada-loja-recebeu">Por ordem decrescente, qual foi o lucro que cada loja recebeu?</a></li>
        <li><a href="#quem-são-os-10-maiores-clientes">Quem são os 10 maiores clientes?</a></li>
        <li><a href="#quais-são-as-cidades-onde-residem-os-10-maiores-clientes">Quais são as cidades onde residem os 10 maiores clientes?</a></li>
	<li><a href="#quais-são-as-cinco-cidades-com-o-maior-número-de-clientes-exceto-as-que-já-possuem-lojas">Quais são as cinco cidades com o maior número de clientes, exceto as que já possuem lojas?</a></li>
        <li><a href="#quem-é-o-ator-que-tem-mais-filmes-alugados">Quem é o ator que tem mais filmes alugados?</a></li>
        <li><a href="#qual-foi-o-lucro-médio-de-cada-ano">Existe algum filme que não foi alugado?</a></li>
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
2. Busque o link de download da base dados da Prefeitura de SP para ser usado conforme o script Request do "data_scraping" de Python.



## Requisitos de Negócios

Como mencionado anteriormente, o objetivo deste projeto é identificar os principais pontos de melhoria nos serviços prestados e os problemas que afetam a população de São Paulo, no caso, com base no período de 2012 ao último mês de 2022. Com base nisso, faremos algumas perguntas:

1. Quais foram os maiores problemas?

2. Em relação aos serviços prestados, quais foram os que tiveram mais casos?

4. Quais distritos tiveram mais denúncias?

5. Qual a quantidade de denúncias recebidas por ano?

6. Qual a grandeza de denúncias de "Animais que transmitem doenças" e quais foram os casos que tiveram mais denúncias?

7. Quais foram os distritos que mais tiveram denúncias de "Animais que transmitem doenças"?

8. Quantos serviços foram finalizados ou não?

9. No geral, é possível determinar se o serviço prestado pela Prefeitura de São Paulo é eficiente?

Com essas perguntas, já será possível mapear todo o processo de análise, incluindo o tratamento dos dados.



## Extração e Tratamento dos Dados

Os dados disponibilizados pela Prefeitura estão organizados em páginas separadas para cada trimestre e ano, inclusive por meio de sua API que também apresenta essa mesma divisão. Para otimizar a forma de extração, foi desenvolvido um script em Python que será explicado detalhadamente a seguir, utilizando as bibliotecas Requests, Beautiful Soup e Pandas (StringIO inclusa), a fim de concatenar os dados em um único dataset.

Após a importação das bibliotecas, o código 


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
