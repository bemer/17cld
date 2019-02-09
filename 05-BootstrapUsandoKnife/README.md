# Bootstrap de Nodes utilizando o Knife

Como vimos no laboratório anterior, o Knife é uma ferramenta que nos permite interagir com o Chef Server para várias tarefas. Neste laboratório, vamos ver na prática como funciona o processo de bootstrap de um novo servidor, ou seja, o processo de instalação do Chef Client em servidores de nossa rede de remotamente, através do Knife.

Antes de começarmos a execução deste laboratório, é importante no entanto entender qual é a arquitetura que vamos utilizar para provisionar nossos nodes. A idéia aqui, é possuir diversos servidores diferentes, para que possamos interagir com todos eles, no entanto como não temos uma quantidade grande de máquinas virtuais, vamos utilizar containers para esta tarefa.

O que vamos fazer inicialmente é:

* Criar uma imagem Docker com um SSH server para que possamos acessar a mesma remotamente;
* Iniciar containers a partir desta imagem;
* Utilizar o Knife para realizar o bootstrap destas imagens, que estarão simulando nossos servidores.

Estes containers serão executados dentro de nosso `Chef Client`, e a partir do `Chef Server` vamos utilizar o Knife para realizar o bootstrap.

Este desenho demonstra a arquitetura utilizada neste laboratório:

![lab architecture](/05-BootstrapUsandoKnife/images/lab_architecture.png)

## 1. Criando a imagem Docker

O primeiro passo a ser executado neste caso, é a criação da imagem Docker que vamos utilizar para simular os servidores. Acesse o servidor `chef-client` e crie uma pasta chamada `docker` e acesse a mesma utilizando o seguinte comando:

    $ mkdir docker && cd docker

Crie um arquivo chamado `Dockefile`:

    $ touch Dockerfile

E utilizando um editor de texto de sua preferência, insira o seguinte conteúdo a este arquivo:

    FROM ubuntu:16.04

    RUN apt-get update && apt-get install -y openssh-server
    RUN mkdir /var/run/sshd
    RUN echo 'root:123456' | chpasswd
    RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

    # SSH login fix. Otherwise user is kicked off after login
    RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

    ENV NOTVISIBLE "in users profile"
    RUN echo "export VISIBLE=now" >> /etc/profile

    EXPOSE 22
    CMD ["/usr/sbin/sshd", "-D"]

Note que este este Dockerfile está utilizando a imagem base `ubuntu:16.04` e realizando a instalação e configuração de um `ssh server`.

Após criar o Dockerfile, execute o comando a seguir para gerar sua imagem:

    $ sudo docker build -t ssh-server_image .

Após gerar sua imagem, você poderá verificar se a mesma está devidamente criada e disponível em seu `Chef Client` através do seguinte comando:

    $ sudo docker images

No output deste comando, você poderá visualizar as duas imagens disponíves: a imagem `ubuntu` utilizada como base image para a geração de nossa imagem com o ssh e a própria imagem `ssh-server_image`:

    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ssh-server_image    latest              07e7baf0bbb1        55 seconds ago      207MB
    ubuntu              16.04               f975c5035748        4 weeks ago         112MB

## 2. Executando um container

Agora que já temos nossa imagem criada, podemos executar a mesma através do seguinte comando:

    $ sudo docker run -d -P --name motd-server ssh-server_image
    
Caso você receba uma mensagem de erro dizendo que já existe um container em execução com este nome, utilize o comando a seguir para remover o container anterior e poder utilizar o mesmo nome em sua execução:
    
    $ sudo docker rm -f $(sudo docker ps -a -q) 

Note que estamos criando um container chamado `motd-server` a partir da imagem `ssh-server_image`. Adicionalmente estamos utilizando a opção `-d` para executar o container em `daemon mode`, ou seja, em background, e a opção `-P` para realizar o mapeamento de uma porta aleatória para acesso ao SSH.

Verifique a execução de seu container através do comando:

    $ sudo docker ps

O output deverá ser semalhante a:

    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                   NAMES
    a32d2fdfc79e        ssh-server_image    "/usr/sbin/sshd -D"   5 seconds ago       Up 4 seconds        0.0.0.0:32768->22/tcp   motd-server

Como informado, a opção `-P` irá mapear uma porta aleatória para o container, desta forma, precisamos descobrir qual é a porta utilizada por este container para realizarmos o processo de bootstrap. Vamos utilizar o seguinte comando para obter os dados da porta utilizada:

    $ sudo docker port motd-server 22

A saída deste comando deverá ser semelhante a:

    0.0.0.0:32768

Anote esta porta, por vamos precisar da mesma no próximo passo.

## 3. Bootstrap com o Knife

Neste momento, já temos um container em execução, simulando um servidor chamado `motd-server`. Vamos então realizar o bootstrap, ou seja, o processo de instalação do `chef-client` e configuração para que ocorra a comunicação com o `chef-server`.

Para isto, acesse o `Chef Server`, em seguida navegue até o diretório `chef-repo/` e execute o seguinte comando:

    $ knife bootstrap <IP do Chef Client>:<Porta utilizada pelo container> -x root -P 123456 -N motd-server

>O endereço IP do servidor Chef Client é o endereço provido pelo DHCP do VirtualBox - mesmo que utilizamos para realizar as configurações de encaminhamento de porta - enquanto a porta utilizada pelo container é a mesma exposta no passo 2 deste tutorial. Lembre-se de remover os sinais `<>` do comando.

Caso você precise obter o endereço IP do chef-client novamente, utilize o comando a seguir no `chef-client`:

    $ ifconfig enp0s3 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}'

Ao executarmos este comando, o knife irá realizar uma conexão via ssh ao nosso container, instalar os pacotes necessários e configurar o mesmo para que ocorra a conexão com o Chef Server.

Para validar o funcionamento do comando, acesse seu Chef Server através da interface web, navegue até `Nodes`, e você deverá ver seu container listado como um servidor chamado `motd-server`:

![motd server](/05-BootstrapUsandoKnife/images/motd-server.png)

Você também poderá validar através do terminal do `Chef Server`, executando o seguinte comando:

    $ knife client list

O servidor chamado `motd-server` deverá ser listado.
