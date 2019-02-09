# Instalando o Apache através do Chef Server

Anteriormente, realizamos a instalação do Apache Server em nosso `Chef Client` utilizando o modo de execução local do Chef. Agora, o próximo passo é criar um cookbook e disponibilizar a instalação do mesmo através de nosso Chef Server, para um novo container, que irá simular mais um servidor.

## 1. Criando um novo Node para instalação do Apache

Vamos criar um novo Node para que possamos simular a instalação do Apache. Para isto, vamos precisar gerar um novo container, expondo além da porta 22 para o ssh, também a porta 80 para o nosso web server.

Para isto, vamos voltar à nossa instância `Chef Client` e editar o arquivo `Dockerfile`, presente no diretório `docker`. Edite a linha que contém a intrução `EXPOSE` adicionando também a porta `80`. Esta linha deverá ficar da seguinte forma:

    EXPOSE 22 80

Agora, vamos gerar uma nova imagem a partir de nosso Dockerfile. Esta imagem irá se chamar `ssh-apache_image`. Para isto, utilizaremos o seguinte comando:

    $ sudo docker build -t ssh-apache_image .

Após a geração desta imagem, vamos executar um novo container, mapeando a porta `8080` de nosso host para a porta `80` do container. Utilize o comando:

    $ sudo docker run -d -p 8080:80 -P --name apache-server ssh-apache_image

Vamos agora listar as portas utilizadas em nosso novo container utilizando o comando:

    $ sudo docker port apache-server 22 | awk '{split($0,a,":"); print a[2]}'

Novamente, anote a porta apresentada para que possamos utilizar no processo de bootstrap.

Após a criação de uma nova imagem e a execução do container, realize o bootstrap deste novo node através do knife, em seu Chef Server. Execute os mesmos passos descritos no tutorial [05-BootstrapUsandoKnife](/05-BootstrapUsandoKnife/), porém desta vez utilizando o nome `apache-server`, com o seguinte comando:

    $ knife bootstrap <IP do Chef Client>:<Porta utilizada pelo container> -x root -P 123456 -N apache-server

## 2. Criação de Cookbook para o Apache

Agora, ainda em seu Chef Server, realize a criação de um novo Cookbook, desta vez chamado `apache`. Utilize o comando:

    $ chef generate cookbook cookbooks/apache

>Lembre-se de executar o comando a partir do diretório chef-repo/.

Edite o arquivo `cookbooks/apache/recipes/default.rb` e insira o seguinte conteúdo:

    package 'apache2' do
      action :install
    end

    service 'apache2' do
      action [:enable,:start]
      supports :reload => true
    end


Envie seu novo cookbook para o servidor através do comando:

    $ knife upload cookbooks/apache

## 3. Editando o Run List do Novo node

Siga o mesmo procedimento descrito no tutorial [05-CriandoCookbooks](/05-CriandoCookbooks/), porém desta vez para o novo node utilizando a receita `apache`:

![apache_run_list](/07-InstalandoApache/images/apache_run_list.png)

## 4. Executando a o Run List no novo node

Em sua instância `Chef Client`, execute o seguinte comando para ter acesso ao container onde será instalado o Apache:

    $ sudo docker exec -ti apache-server /bin/bash

E em seguida, já dentro do container, execute o comando:

    # chef-client

Note que o Apache será instalado.

A primeira maneira para validar o funcionamento do Apache em nosso novo node, é verificando se o serviço foi devidamente instalado e está ativo. Faça isto através do comando:

    # service apache2 status

A saída deste comando deverá ser:

    * apache2 is running

## 5. Acessando através de seu browser

Como estamos realizando a instalação de um servidor web, nada melhor do que utilizarmos o nosso browser para validação do funcionamento. Para isto, abra uma nova aba em seu navegador e insira o endereço IP do seu `Chef Client` com a porta `8080`, Esta entrada deverá ser semelhante a:

    http://192.168.0.13:8080

Você deverá visualizar a interface do Apache Server:

![apache server](/07-InstalandoApache/images/apache_server.png)
