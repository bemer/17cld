# Criando Chef Cookbooks

Neste momento, já temos nosso ambiente pronto com o Chef Server instalado, um container simulando um servidor e com o bootstrap já realizado. Vamos agora começar o processo de criação de um cookbook.


## 1. Criando um cookbook

Para realizar a criação de nosso primeiro `cookbook`, deveremos acessar o `chef-server` e no diretório `chef-repo` executar o seguinte comando:

    # chef generate cookbook cookbooks/motd

>Alguns erros serão gerados, pois não realizamos as configurações globais de usuário do Git. Em nosso laboratório, podemos ignorar estes erros.

Após criar o `cookbook`, o próximo passo é editar a nossa receita. Para isto, o primeiro passo é acessar o diretório `cookbooks` em nosso chef-repo. Utilize o comando:

    # cd cookbooks/motd

Explore o conteúdo deste diretório, afim de entender como funciona a estrutura padrão dos cookbooks do chef. Você pode inclusive explorar o conteúdo dos arquivos utilizando o comando `cat`.

Mais informações sobre a estrutura dos cookbooks podem ser encontrados [neste link](https://docs.chef.io/cookbooks.html).


## 2. Editando a receita default

Vamos editar nosso cookbook para que possamos executar algo em nosso novo node. Neste caso, vamos criar uma receita simples, que cria um arquivo chamado `motd.txt` no diretório `/tmp` de nosso container.

Para isto, edite o conteúdo do arquivo `recipes/default.rb` e insira o seguinte:

    file '/tmp/motd.txt' do
      content 'Arquivo criado utilizando o Chef Server!'
    end

Após realizar a edição, salve o arquivo.

## 3. Enviando o cookbook para o Chef Server

Como vimos anteriormente, todas as receitas são disponibilizadas em nossos nodes através do Chef Server. Sendo assim, devemos enviar a receita recém criada para o servidor. Para isto, vamos executar os seguintes comandos:

    # knife upload cookbooks/motd

> NOTA: Este arquivo deverá ser executado a partir do diretório `chef-repo`.

Após realizar o upload de seu cookbook, você poderá verificar o conteúdo do mesmo através da interface web, clicando em `Policy` na parte superior da tela, selecionando seu cookbook chamado `motd` e em seguida, na parte inferior da tela, selecionando `Content`, `Recipes` e em seguida `default.rb`, conforme a imagem abaixo:

![cookbook list](/06-CriandoCookbooks/images/cookbook_list.png)


Você também conseguirá listar os cookbooks presentes no servidor através do seguinte comando, executado no diretório `chef-repo` dentro do `Chef Server`:

    # knife cookbook list

## 4. Alterando o Run List do Node

Como nosso cookbook já está presente em nosso servidor, o próximo passo é editar o `Run list` para que o Chef Client instalado em nosso container possa realizar o download e a execução de nossa receita.

Para isto, na interface web do servidor, vamos clicar em `Nodes` na parte superior da tela, selecionar o nosso node chamado `motd_server` e clicar no campo `Edit Run List` em actions:

![node_run_list_1](/06-CriandoCookbooks/images/node_run_list_1.png)

Em seguida, devemos arrastar a receita `motd` presente em `Available Recipes` para o campo `Current Run List`:

![node_run_list_2](/06-CriandoCookbooks/images/node_run_list_2.png)

Em seguida, vamos clicar em `Save Run List`.

Você pode validar a nova Run List de seu node através do comando:

    # knife node show motd-server

Verifique se a receita `motd` está sendo exibida, conforme abaixo:

    Node Name:   motd-server
    Environment: default
    FQDN:        a32d2fdfc79e
    IP:          
    Run List:    recipe[motd]
    Roles:       
    Recipes:     
    Platform:    ubuntu 16.04
    Tags:        

## 5. Executando nossa receita

O próximo passo para isto, é executar a nossa Run List no node `motd-server` para que a receita `motd` possa criar o arquivo no diretório `/tmp`.

Vamos então acessar o nosso `Chef Client` e executar o seguinte comando:

    $ sudo docker exec -ti motd-server /bin/bash

> NOTE: Este comando irá nos mover para dentro do container `motd-server` que está simulando um servidor em nosso ambiente. A partir daí, todos os comandos são executados dentro do container, e não mais no Chef Client.

Vamos agora utilizar o chef-client que foi instalado em nosso container durante o processo de bootstrap. Para isto, utilizamos o comando:

    # chef-client

Note que o `chef-client` irá realizar o download de seu cookbook e executar os passos localmente. O output deste comando deverá ser semelhante ao seguinte:

    Starting Chef Client, version 13.8.5
    [2018-04-08T17:17:03+00:00] WARN: Plugin Network: unable to detect ipaddress
    resolving cookbooks for run list: ["motd"]
    Synchronizing Cookbooks:
      - motd (0.1.0)
    Installing Cookbook Gems:
    Compiling Cookbooks...
    Converging 1 resources
    Recipe: motd::default
      * file[/tmp/motd.txt] action create
        - create new file /tmp/motd.txt
        - update content in file /tmp/motd.txt from none to 38ac4a
        --- /tmp/motd.txt	2018-04-08 17:17:03.733992469 +0000
        +++ /tmp/.chef-motd20180408-315-1d1f2xo.txt	2018-04-08 17:17:03.733992469 +0000
        @@ -1 +1,2 @@
        +Arquivo criado utilizando o Chef Server!

    Running handlers:
    Running handlers complete
    Chef Client finished, 1/1 resources updated in 01 seconds

Para validar o funcionamento da execução desta receita, utilize o seguinte comando para listar o conteúdo do diretório `/tmp` dentro do container:

    # ls /tmp/

Note que existirá um arquivo chamado `motd.txt`. Liste o conteúdo deste arquivo através do seguinte comando:

    # cat /tmp/motd.txt;echo

Após executar estes comandos e validar o funcinamento de sua receita, utilize o comando `exit` para sair do Container e voltar para o terminal do Chef Client.

## 6. Verificando os relatórios

Outra forma de verificar o status de nossa execução é utilizando a ferramenta de `Reporting` do Chef Server. Na interface web do Chef Server, clique em `Reports` na parte superior da tela, e verifique o status de suas execuções:

![reporting](/06-CriandoCookbooks/images/reporting.png)
