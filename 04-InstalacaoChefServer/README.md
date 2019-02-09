# Instalação do Chef Server

O Chef Server é um componente que age como um "hub" para os dados de configuração, sendo responsável por armazenar os cookbooks e suas receitas, as políticas que deverão ser aplicadas a cada um dos servidores em nossa infra estrutura e também os metadados que descrevem cada um destes servidores.

Neste tutorial, vamos realizar o processo de instalação do Chef Server em nosso servidor.

## 1. Instalando o Chef Server

Para realizarmos a instalação do Chef Server, vamos utilizar a máquina virtual com o nome `chef-server`. Inicie a máquina virtual em seu computador e siga os passos descritos no capítulo [2. Configuração da Rede das VMs](/02-ConfiguracaoRedeVirtualBox/) para realizar acesso ao novo servidor utilizando os seguinte dados:

    Usuário: chef-admin
    Senha: chefserver

> Caso queira, salve a no Putty, utilizando desta vez o nome `chef-server`.

Agora que estamos conectados ao servidor através do Putty, vamos iniciar o processo de instalação do `Chef Server`. Para isto, vamos primeiro realizar o download da última versão do Chef Server. O link para download da versão mais recente do Chef Server para instalação no Ubuntu poderá ser encontrado [nesta url](https://downloads.chef.io/chef-server#ubuntu).

Neste caso, vamos utilizar a versão `12.17.33`.

Em seu Chef Server, execute o seguinte comando para realizar o download do pacote:

    $ wget https://packages.chef.io/files/stable/chef-server/12.17.33/ubuntu/16.04/chef-server-core_12.17.33-1_amd64.deb

Aguarde o processo de download do arquivo, e quando o mesmo for completado, instale o Chef Server utilizando o seguinte comando:

    $ sudo dpkg -i chef-server-core_12.17.33-1_amd64.deb

>Por se tratar da instalação de um pacote, precisaremos utilizar um usuário com permissões de `root`. Por isso o uso do comando `sudo`.

Agora, execute o seguinte comando para atualizar o endereço IP do Chef Server utilizado para a geração dos certificados:

    echo api_fqdn "\""$(ifconfig enp0s3 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}')"\"" | \
    sudo tee --append /etc/opscode/chef-server.rb > /dev/null

E em seguida execute execute o seguinte comando para configurar o servidor:

    $ sudo chef-server-ctl reconfigure

Este processo deverá demorar alguns minutos.

Ao final, você deverá ver o seguinte output:

    Chef Client finished, 495/1084 resources updated in 04 minutes 46 seconds
    Chef Server Reconfigured!

A partir deste momento, o Chef Server já está instalado em nosso servidor, sendo necessário agora instalarmos alguns módulos adicionais para interação com o mesmo.

## 2. Instalação dos módulos adicionais

O primeiro módulo que iremos instalar é a interface web do servidor. Para isto, vamos executar os seguintes comandos:

    $ sudo chef-server-ctl install chef-manage
    $ sudo chef-server-ctl reconfigure
    $ sudo chef-manage-ctl reconfigure

>Durante a execução do último comando, será necessário aceitarmos os termos de uso do Chef Manage. Será apresetado um arquivo com os termos em sua tela. Para sair deste arquivo, digite `:q` e em seguida digite `yes` e pressione `Enter`.

E em seguida, instalaremos o módulo de relatórios:

    $ sudo chef-server-ctl install opscode-reporting
    $ sudo chef-server-ctl reconfigure
    $ sudo opscode-reporting-ctl reconfigure

Agora, vamos criar um novo usuário para acessarmos a interface de administração do Chef Server utilizando o seguinte comando:

    $ sudo chef-server-ctl user-create brunoemer Bruno Emer brunoemer@gmail.com '123456' --filename ~/brunoemer.pem

> Lembre-se de substituir o comando, inserindo seu nome e endereço de email.

Uma vez executados os passos para a instalação do Chef Server e dos módulos adicionais, acesse o servidor através de seu browser. Você deverá ser direcionado para a tela de login do Chef Server. Utilize o usuário e senha que acabou de criar para realizar o primeiro acesso:

![chef server login](/04-InstalacaoChefServer/images/chef_server_login.png)

## 3. Criação de uma organização

No Chef Server, uma `organização` é uma entidade utilizada para controlar os acessos. cada organização contém grupos padrão, pelo menos um usuário e um nó onde o chef-client é instalado.

Quando trabalhamos com o Chef Server, podemos ter diversas organizações, sendo que uma destas organizações é criada justamente no processo de instalação.

Ao executarmos o login pela primeira vez no Chef Server, nos será apresentada uma tela para a criação de nossa organização padrão. Nesta tela, clique em `Create New Organization`:

![create organization](/04-InstalacaoChefServer/images/create_organization.png)

Em seguida preencha o `Full Name` e o `Short Name` desta organização como `mba-fiap` e clique em `Create Organization`:

![mba fiap organization](/04-InstalacaoChefServer/images/mba_fiap_organization.png)

Você será redirecionado para a tela principal do Chef Server:

![chef main screen](/04-InstalacaoChefServer/images/chef_main_screen.png)

## 4. Instalação do Chef DK e configuração do Workstation

O Chef DK (Chef Development Kit) é um pacote que contem basicamente tudo o que precisamos para trabalhar com o Chef. Ele irá instalar o chef-client, knife, ferramentas de teste, entre outros.

Em nosso laboratório, vamos instalar o Chef DK no mesmo servidor em que instalamos o Chef Server. O primeiro passo é realizarmos o download do pacote do Chef DK. Para isto vamos utilizar o comando:

    $ wget https://packages.chef.io/files/stable/chefdk/2.4.17/ubuntu/16.04/chefdk_2.4.17-1_amd64.deb

Em seguida, vamos realizar o processo de instalação:

    $ sudo dpkg –i chefdk_2.4.17-1_amd64.deb

Feito isto, vamos realizar a configuração do Workstation. O workstation é basicamente uma máquina que contém todos os pacotes necessários para interagirmos com o Chef e realizar a criação de cookbooks, receitas, testar e validar o funcionamento do Chef e interagir com o Chef Server.

O primeiro passo para a configuração de nosso Workstation é a instalação do Ruby, linguagem de programação utilizada pelo Chef. Execute os seguintes comandos:

    $ sudo apt-get update
    $ sudo apt-get install -y ruby

Após a instalação do Ruby, vamos realizar o download do Starter Kit diretamente da interface web do Chef Server. O Starter Kit é basicamente um pacote com todos os certificados e arquivos de configuração necessários para que seu workstation possa interagir com o Chef Server.

>Os arquivos de configuração necessários para que o workstation interaja com o Chef Server podem ser gerados manualmente, porém utilizando a interface web, o processo fica bem mais fácil, poupando bastante tempo. Caso você queira mais informações sobre o processo de geração do Starter Kit, acesse [este link](https://docs.chef.io/workstation.html).

Ao acessar o Chef Server através da interface web, navegue até `Aministration`, selecione a organização `mba-fiap` criada anteriormente, e no menu lateral esquerdo, clique em `Starter Kit`. Você irá se deparar com a seguinte tela:

![download starter kit](/04-InstalacaoChefServer/images/download_starter_kit.png)

Nesta tela, clique em `Download Starter Kit`, e em seguida clique em `Proceed`.

O arquivo será salvo em seu computador. O próximo passo é mover o mesmo para a máquina virtual. Este é um processo que pode ser realizado através de recursos do próprio Hypervisor, porém para simular um ambiente de produção, vamos utilizar um aplicativo chamado `Filezilla`, que é basicamente um client SFTP.

Instale o `Filezilla` em seu computador a partir deste link:

    https://filezilla-project.org/

Ao terminar o processo de instalação, abra o aplicativo e conecte-se ao servidor preenchendo os seguintes dados na parte superior da tela:

    Host: Endereço IP do Chef server
    Username: chef-admin
    Password: chefserver
    Port: 22

E em seguida clique em `Quickconnect`. Será apresentada uma tela para que você aceite o certificado do servidor. Aceite o mesmo e você estará conectado ao servidor através de um client SFTP.

Agora, vamos transferir o arquivo `chef-starter` para dentro de nosso servidor. Através da interface do Filezilla, navegue ao diretório em que você realizou o download do Starter Kit. Dê um duplo clique no arquivo, e ele será transferido para dentro do Servidor:

![filezilla screen](/04-InstalacaoChefServer/images/filezilla_screen.png)

Volte ao terminal de seu `Chef Server` e verifique que o comando `chef-starter.zip` está presente em seu diretório através do comando:

    $ ls chef-starter.zip

Vamos agora extrair o Starter Kit utilizando o seguinte comando:

    $ unzip chef-starter.zip

Uma pasta chamada `chef-repo` será criada. Acesse esta pasta utilizando o comando:

    $ cd chef-repo/

Agora, vamos realizar o download dos certificados do servidor. Para isto, utilizaremos o utilitário do Chef chamado `Knife.`
O Knife é uma ferramenta de linha de comando que provê uma interface entre um repositório local e o Chef Server. Utilizaremos o Knife para algumas tarefas como por exemplo o bootstrap de novos hosts, o upload de novas receitas para o Chef Server, entre outros.

Para realizar o download dos certificados do servidor, execute o seguinte comando:

    $ knife ssl fetch

A saída de seu comando deverá ser semelhante a:

    WARNING: Certificates from 192.168.0.13 will be fetched and placed in your trusted_cert
    directory (/home/chef-admin/chef-repo/.chef/trusted_certs).

    Knife has no means to verify these are the correct certificates. You should
    verify the authenticity of these certificates after downloading.

    Adding certificate for 192_168_0_13 in /home/chef-admin/chef-repo/.chef/trusted_certs/192_168_0_13.crt

## 5. Validando a instalação do Chef Server

Neste ponto, já temos o Chef Server devidamente instalado, realizamos a configuração de nossa Workstation, instalamos todas as ferramentas necessárias para a administração dos Cookbooks e baixamos os certificados apresentados pelo nosso servidor.

Vamos agora executar um comando através do Knife afim de interagir com o Chef Server. Este é um comando simples, que irá apenas listas os nodes registrados em nosso servidor. Execute o comando:

    $ knife client list

A sua saída deverá ser:

    mba-fiap-validator

Com isto, finalizamos o processo de instalação e configuração do Chef Server, e já podemos iniciar o bootstrap dos servidores em nossa rede.
