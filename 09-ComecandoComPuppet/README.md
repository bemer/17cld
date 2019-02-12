# Começando com o Puppet

O `Puppet` provê ferramentas para automatizat a gestão de sua infra estrutura, sendo um produto open source com uma grande comunidade de usuários e desenvolvedores.

Ao trabalhar com o `Puppet`, você pode escolher entre basicamente duas arquiteturas: uma arquitetura client-server, utilizando o `Puppet Master` e o `Puppet Agent` ou então uma arquitetura stand-alone, fazendo uso do `Puppet Apply`.

Neste tutorial, utilizaremos a arquitetura client-server, instalando e configurando o `Puppet Server` e instalando `Puppet Agents` em containers que irão simular máquinas virtuais.


## 1. Instalação do Puppet Master

O `Puppet Master` é uma aplicação em `Ruby` que compila configurações para diferentes nodes em sua infra estrutura.

O primeiro passo para a instalação do `Puppet Master` é adicionarmos o repositório do Puppet em nosso servidor. Para isto, na máquina virtual `puppet-server` execute os seguintes comandos:

    $ wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb
    $ sudo dpkg -i puppetlabs-release-pc1-xenial.deb
    $ sudo apt-get update -y

Agora, vamos instalar o `Puppet Master` através do seguinte comando:

    $ sudo apt-get install puppetserver -y

Após finalizarmos a instalação do Master, vamos editar as configurações de alocação de memória do Puppet Master. Para isto, edite o arquivo `/etc/default/puppetserver` alterando a linha `9` de

    JAVA_ARGS="-Xms2g -Xmx2g -XX:MaxPermSize=256m"

para:

    JAVA_ARGS="-Xms512m -Xmx512m"

Após realizar esta alteração, execute os seguintes comandos:

    $ sudo systemctl start puppetserver
    $ sudo systemctl enable puppetserver

Para validar o funcionamento de seu `Puppet Master`, execute o seguinte comando:

    $ sudo systemctl status puppetserver

A saída deverá ser semelhante a:

    ● puppetserver.service - puppetserver Service
    Loaded: loaded (/lib/systemd/system/puppetserver.service; enabled; vendor preset: enabled)
    Active: active (running) since Thu 2018-06-28 20:39:36 -03; 33s ago
    Main PID: 5159 (java)
    CGroup: /system.slice/puppetserver.service
           └─5159 /usr/bin/java -Xms512m -Xmx512m -Djava.security.egd=/dev/urandom -XX:OnOutOfMemoryError=kill -9 %p -cp /opt/pupp


Isto significa que seu `Puppet Master` está devidamente instalado e em execução.


## 2. Instalando o Puppet Agent

Para a execução deste lab, vamos utilizar a mesma máquina virtual que utilizamos nos labs anteriores, chamada `chef-client`. Desta maneira, é importante lembrar que por mais que estejamos instalando pacotes em uma máquina chamada `chef-client`, vamos utilizar o `Puppet` para este provisionamento.

Para a instalação do `Puppet Agent`, você deverá adicionar o repositório também no servidor. Para isto, execute os seguintes comandos no terminal do `chef-client`:

    $ wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb
    $ sudo dpkg -i puppetlabs-release-pc1-xenial.deb
    $ sudo apt-get update -y

Feito isto, instale o `Puppet Agent` através do seguinte comando:

    $ sudo apt-get install puppet-agent -y

Agora que o `Puppet Agent` foi instalado, vamos realizar uma configuração para que o Puppet Agent possa se comunicar com o Puppet Master. Para isto, precisamos editar o arquivo `/etc/hosts` e adicionar o endereço IP do Puppet Master. O arquivo hosts deverá ficar conforme abaixo:

    127.0.0.1       localhost
    127.0.1.1       chef-client.fiap.com.br
    <IP do Master>   puppet-server.fiap.com.br

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters

Agora, vamos editar o arquivo de configruração do Puppet Agent em nossa VM. Edite o arquivo `/etc/puppetlabs/puppet/puppet.conf` e insira o seguinte conteúdo:

    [main]
    certname = chef-client.fiap.com.br
    server = puppet-server.fiap.com.br


Inicie o Puppet Agent através dos seguintes comandos:

    $ sudo systemctl start puppet
    $ sudo systemctl enable puppet

## 3. Assinando o Certificado do Puppet Agent no Puppet Server

Quando um novo node com o `Puppet Agent` é executado pela primeira vez, o mesmo envia uma requisição para a assinatura de um certificado para o `Puppet Master`. Utilizando uma arquitetura `client-server`, o Puppet Master deve aprovar a requisição do certificado para cada um dos nodes com o Puppet Agent instalado.

Agora, acessando a VM `puppet-server`, execute o seguinte comando para listar as requisições de certificado pendentes:

    $ sudo /opt/puppetlabs/bin/puppet cert list

Você deverá ver algo semelhate a:

    "chef-client" (SHA256) 60:2F:BA:93:E8:8E:98:92:E8:EE:7C:B4:47:3B:0D:D5:0C:81:C4:8C:07

Agora, ainda no `puppet-server` execute o seguinte comando para assinar o certificado:

    $ sudo /opt/puppetlabs/bin/puppet cert sign chef-client.fiap.com.br

A saída deverá ser semelhante a esta:

    Signing Certificate Request for:
      "chef-client" (SHA256) 60:2F:BA:93:E8:8E:98:92:E8:EE:7C:B4:B7:0E:C5:41:FC:C4:9B:A2:77:FC:73:47:3B:0D:D5:0C:81:C4:8C:07
    Notice: Signed certificate request for chef-client
    Notice: Removing file Puppet::SSL::CertificateRequest chef-client at '/etc/puppetlabs/puppet/ssl/ca/requests/chef-client.pem'

Agora, o Puppet Server já consegue se conectar ao node que será gerenciado, em nosso caso, chamado `chef-client`.

Para validar a comunicação, em sua VM `chef-client`, onde o `Puppet Agent` foi instalado, execute o seguinte comando:

    $ sudo /opt/puppetlabs/bin/puppet agent --test

A saída deverá ser semelhante a esta:

    Info: Using configured environment 'production'
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    Info: Caching catalog for chef-client
    Info: Applying configuration version '1530232063'
    Notice: Applied catalog in 0.02 seconds

## 4. Interagindo com o Puppet

Para gerenciarmos nossa infra estrutura utilizando o Puppet, trabalhamos com arquivos `manifest`. Manifestos são basicamente arquivos com código que o Puppet irá interpretar, e possuem a extensão `.pp`. Por padrão, os manifestos ficam no `Puppet Master` no diretório `/etc/puppetlabs/code/environments/production/manifests/`.

Vamos então criar um novo manifesto, que irá criar um arquivo no diretório `/tmp` de nosso node. Crie um arquivo chamado `motd.pp` utilizando o seguinte comando:

    $ sudo vi /etc/puppetlabs/code/environments/production/manifests/motd.pp

E insira o seguinte conteúdo:

    file { '/tmp/motd':
      ensure  => 'present',
      content => "Este arquivo foi criado pelo Puppet.\n"
    }

Agora, na VM `chef-client`, execute o seguinte comando:

    $ sudo /opt/puppetlabs/bin/puppet agent --test

O Puppet será executado, criando o arquivo `motd` no diretório `/tmp`. A saída do comando deverá ser semelhante a:

    Info: Using configured environment 'production'
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    Info: Caching catalog for chef-client
    Info: Applying configuration version '1530235205'
    Notice: /Stage[main]/Main/File[/tmp/motd]/ensure: defined content as '{md5}c83a034f3b55e6b2dceeeefd3d676435'
    Notice: Applied catalog in 0.02 seconds

Para validar o funcionamento, execute o seguinte comando:

    $ cat /tmp/motd

Você deverá visualizar o conteúdo do arquivo.

## 5. Instalando o Apache com o Puppet

Vamos agora realizar a instalação de um pacote utilizando o Puppet. Para isto, na VM `puppet-server` vamos instalar o módulo `puppetlabs-apache`.

>No Puppet, módulos são uma forma de agrupar tarefas que serão executadas. Existem diversos módulos já disponíveis criados pela comunidade e, caso queira, você mesmo pode escrever os seus próprios módulos.

Para instalar o módulo puppetlabs-apache, execute o seguinte comando:

    $ sudo /opt/puppetlabs/bin/puppet module install puppetlabs-apache

A saída de seu comando deverá ser semelhante a:

    Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
    Notice: Downloading from https://forgeapi.puppet.com ...
    Notice: Installing -- do not interrupt ...
    /etc/puppetlabs/code/environments/production/modules
    └─┬ puppetlabs-apache (v3.1.0)
    ├── puppetlabs-concat (v4.2.1)
    └── puppetlabs-stdlib (v4.25.1)

Agora, crie um novo manifesto através do seguinte comando:

    $ sudo vi /etc/puppetlabs/code/environments/production/manifests/apache.pp

E insira o seguinte conteúdo:

    node 'chef-client' {
      class { 'apache': }
      apache::vhost { 'localhost':
        port    => '8080',
        docroot => '/var/www/html'
      }
    }

Note que agora, estamos especificando qual node executará este manifesto.

Após criar o arquivo, acesse a VM `chef-client` e execute o seguinte comando:

    $ sudo /opt/puppetlabs/bin/puppet agent --test

Caso tudo esteja correto, após a execução deste comando, você poderá utilizar o browser de seu computador para acessar o `chef-client` na porta `8080` e deverá visualizar a página do Apache.
