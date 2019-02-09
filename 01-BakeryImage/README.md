# Criando uma Bakery Image

A técnica de Bakery Image consiste em criar uma imagem de uma máquina virtual que já possua todos os pacotes necessários instalados, afim de facilitar o processo de deploy de novos servidores, economizando tempo com a instalação do mesmo. Quando trabalhamos com Bakery Images, a idéia é investir tempo afim de automatizar apenas o processo de configuração do servidor, sem levar em conta o processo de download de novas ferramentas.

Neste tutorial, vamos criar uma Bakery Image na nuvem, utilizando a nuvem da AWS.

## 1. Instalando o Apache via User data

Neste laboratório, vamos criar uma imagem base com um servidor apache instalado, sem que haja a necessidade de realizarmos acesso remoto ao servidor.

Para isto, logue-se em sua conta da AWS e em `AWS Services` digite `EC2`, Clique em `EC2`:

![acessar ec2](/01-BakeryImage/images/acessar_ec2.png)

Feito isto, você será redirecionado para a console do EC2, que é o serviço responsável por entregar máquinas virtuais na cloud da AWS. Nesta tela, clique em `Launch Instance`:

![launch instance](/01-BakeryImage/images/launch_instance.png)

Agora, vamos selecionar o sistema operacional a ser utilizado em nossa imagem. Neste caso, selecione a primeira opcão `Amazon Linux AMI`:

![select operating system](/01-BakeryImage/images/select_operating_system.png)

A próxima página permite que selecionemos o tipo de máquina virtual. Neste caso, o tipo está associado à quantidade de memória e CPU disponibilizados em nosso servidor. Vamos mater a opção `t2.micro` selecionada. Esta máquina será entregue com 1 vCPU e 1 GiB de memória RAM. Após selecionar o tipo `t2.micro`, clique em `Next: Configure Instance Details` no canto inferior direito da página:

![select instance type](/01-BakeryImage/images/instance_type.png)

Na próxima página, expanda a aba `Advanced Details`. Note que o campo `User Data` será exibido. Aqui, deveremos inserir o script para instalação do Apache Server. Insira o seguinte script no campo `User Data`:

    #!/bin/sh
    yum -y install httpd
    chkconfig httpd on
    /etc/init.d/httpd start

Em seguida, clique em `Next: Add Storage` no canto direito inferior da tela:

![instance details](/01-BakeryImage/images/instance_details.png)

Na próxima tela, clique apenas em `Next: Add Tags`:

![add storage](/01-BakeryImage/images/add_storage.png)

Agora, clique em `click to add a Name tag` no centro da página. Note que este é um like, que irá permitir a criação de uma `Tag` com um nome para nosso servidor apache. Isto irá nos ajudar a identificar nosso servidor na console da AWS posteriormente. Vamos nomear nosso servidor `apache-server`, e em seguida, clicar em `Next: Configure Security Group` no canto direito inferior da tela:

![add tags](/01-BakeryImage/images/add_tags.png)

Agora, vamos realizar a configuração de firewall necessária para que possamos acessar o nosso servidor apache através da porta 80 (HTTP). No passo 6, mantenha selecionada a opção `Create a **new** security group`. Para o nome de nosso security group, vamos utilizar `apache-server`. Insira o mesmo texto também na descrição.

Altere a regra do seu security group para que invés de `SSH`, a opção `Type` seja `HTTP`. Em seguida, clique em `Review and Launch`, no canto inferior direito da tela:

![configure sg](/01-BakeryImage/images/configure_sg.png)

Na próxima tela, revise todas as suas configurações, e clique em `Launch`, no canto inferior direito.

Note que ao clicar em `Launch`, será apresentada uma tela para que você selecione um certificado. Nesta tela, selecione `Create a new key pair` e insira um nome para o seu certificado. Para este laboratório, vamos utilizar o padrão `seu_nome-mba-fiap`. Após preencher o nome, clique em `Download Key Pair` e salve o arquivo em algum lugar seguro.

>Note que o download deste key pair só é apresentado uma vez, durante a sua criação. Caso você perca este arquivo, você não conseguirá mais acessar suas instâncias através de SSH, portanto salve o mesmo em algum lugar seguro.

Após isto, clique em `Launch Instances`:

![create key pair](/01-BakeryImage/images/create_key_pair.png)

Neste ponto, seu servidor será provisionado. Clique então em `View Instances` para verificar o status de seu servidor:

![launch status](/01-BakeryImage/images/launch_status.png)

Você será redirecionado novamente para a página do EC2, no entanto agora existirá uma instância em execução. Para testarmos a instalação do Apache Server nesta instância, vamos copiar o hostname público da mesma e utilizar uma nova aba de nosso browser para validar o funcionamento.

O hostname está disponível em `Public DNS (IPv4)`:

![get public dns](/01-BakeryImage/images/get_public_dns.png)

Ao acessar este endereço em uma nova aba do browser, deveremos visualizar a página do Apache Server:

![apache server](/01-BakeryImage/images/apache_server.png)


## 2. Criando uma imagem

Agora que já temos nosso servidor apache funcionando, vamos gerar uma nova imagem a partir deste servidor. Para isto, na tela do EC2, clique eu sua instância chamada `apache-server` com o botão direito do mouse e em `Image`, clique em `Create Image`:

![create image](/01-BakeryImage/images/create_image.png)

Na próxima tela, complete com o nome `apache-server` e adicione uma descrição para o mesmo. Clique então em `Create Image`:

![image information](/01-BakeryImage/images/image_information.png)

Note que o processo de criação de uma nova imagem será iniciado. Para acompanhar este processo, clique em `View pending image ami-0a2d560eb597bb` (note que este ID será diferente para cada conta):

![view image](/01-BakeryImage/images/view_image.png)

Este processo deverá demorar alguns minutos. Quando o mesmo for completado, o status de sua imagem será exibido como `available`:

![image status](/01-BakeryImage/images/image_status.png)


## 3. Iniciando um servidor a partir da sua imagem

Quando o processo de criação de sua imagem estiver completo, você poderá então iniciar novas instâncias a partir dela. Para isto, clique em `Launch` na parte superior esquerda da tela:

![launch image](/01-BakeryImage/images/launch_image.png)

> Caso você não visualize sua imagem na console, clique em `AMIs`, no menu lateral esquerdo da tela, dentro de `IMAGES`.

Você será redirecionado então para o mesmo processo de criação de uma instância EC2. Siga os mesmos procedimentos de anteriormente, no entanto **não adicione o script de user data** e nomeie esta nova instância como `apache-server-2`.

Desta vez, para a configuração do security group, utilize a opção `Select an existing security group` e selecione o security group criado anteriormete, `apache-server`.

Note que desta vez, ao clicar em `Launch`, o seu certificado já estará selecionado. Marque a opção logo abaixo do mesmo para garantir que você tem ciência de que se não tiver acesso ao arquivo você não conseguirá acessar sua instância e em seguida clique em `Launch Instances`.

Você deverá ter agora dois servidores em execução, sendo que um deles foi configurado manualmente e o outro foi criado através de uma imagem previamente gerada. Acesse o hostname de seu novo servidor em uma nova aba do browser e valide o funcionamento.

## 4. Limpando o ambiente

Após finalizarmos o laboratório, é importante se lembrar de excluir todos os servidores e imagem previamente criados para que não ocorram cobranças em seu cartão de crédito.

Para isto, na tela principal do serviço EC2, selecione os dois servidores criados, clique em `Actions`, `Instance State` e em `Terminate` para excluir as máquinas virtuais provisionadas. Na tela de confirmação, clique em `Yes, Terminate`:

![terminate instances](/01-BakeryImage/images/terminate_instances.png)

Agora, no menu lateral esquerdo, navegue até `AMIs`, selecione a imagem criada e em `Actions`, clique em `Deregister`:

![deregister ami](/01-BakeryImage/images/deregister_ami.png)

Por fim, clique em `Snapshots`no menu lateral, selecione o snapshot existente, clique em `Actions` e em seguida em `Delete`:

![delete snapshot](/01-BakeryImage/images/delete_snapshot.png)
