# Trabalho final - 17CLD

Você trabalha em uma grande empresa e tem a necessidade de realizar a instalação de um servidor apache em sua infra estrutura. O processo de instalação deverá levar em consideração os seguintes requerimentos:

* O apache deverá possuir 3 virtual hosts, sendo um para o ambiente de Desenvolvimento, um para o ambiente de QA e um para o ambiente de produção.
* Cada um dos ambientes deverá apontar para uma página HTML que apresente o nome do ambiente, bem como o nome de cada integrante de sua equipe (grupo) junto com o ID que identifica cada um (RM).
* O acesso via SSH deverá estar habilitado, porém a porta utilizada não poderá ser a 22, devendo esta ser substituída para uma porta alta de sua escolha.
* Este servidor deverá possuir também um usuário com permissões totais, que será utilizado pelo CTO de sua companhia (professor).

Para atender a esta demanda, você deverá utilizar `Chef`, criando um ou mais cookbooks, uma vez que esta demanda tem se repetido constantemente e o número de servidores em seu datacenter tende a aumentar devido a campanhas de marketing.

Você deverá fornecer um arquivo .zip contendo os cookbooks que serão executados para provisionar o ambiente atendendo a todos os requisitos, além de uma documentação identificando os processos para execução das receitas Chef e os seguintes itens:

* Qual foi a porta utilizada para o acesso via SSH;
* Dados de usuário e senha para que seu CTO possa validar o funcionamento do ambiente.
* Quais os virtual hosts configurado no servidor para que o CTO possa efetuar as alterações no arquivo hosts local e validar o funcionamento.


>Note que o processo de instalação deverá levar em consideração apenas um servidor com 3 virtual hosts configurados no Apache, e não 3 servidores virtuais cada um com um ambiente.

Este processo de instalação será a base em seu processo de avaliação sendo que cada um dos itens irá valer 2 pontos e a documentação irá valer mais 2 pontos.
