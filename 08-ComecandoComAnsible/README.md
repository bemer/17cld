# Começando com o Ansible

O `Ansible` é uma plataforma de automação open source, bastante simples mas ainda assim muito poderosa. Neste tutorial utilizaremos as máquinas virtuais já existentes para instalar o Ansible e realizar configurações em hosts remotos utilizando esta ferramenta.

## 1. Instalação do Ansible

Por padrão o Ansible gerencia nodes remotos através do protocolo SSH. Isso significa que quando instalamos o Ansible, não estaremos adicionando nenhum banco de dados e nem agents em nodes remotos. Tudo o que iremos realizar é a instalação do Ansible em um servidor (que em um cenário de vida real poderia ser substituído pelo seu próprio computador) e a partir deste servidor iremos gerenciar toda a infra-estrutura.

Para realizar a instalação do Ansible, vamos utilizar a nossa máquina virtual chamada `chef-server`. A mesma onde instalamos o `Chef` nos passos anteriores.

Acesse o `chef-server` via ssh e execute os seguintes comandos:

    $ sudo apt-get update
    $ sudo apt-get install -y software-properties-common
    $ sudo apt-add-repository ppa:ansible/ansible
    $ sudo apt-get update
    $ sudo apt-get install -y ansible

Feito isto, o `Ansible` terá sido instalado em seu servidor. Você poderá verificar se a instalação ocorreu com sucesso através do comando:

    $ ansible --version

A saída deverá ser semelhante a:

    ansible 2.5.1
      config file = /etc/ansible/ansible.cfg
      configured module search path = [u'/home/chef-admin/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
      ansible python module location = /usr/lib/python2.7/dist-packages/ansible
      executable location = /usr/bin/ansible
      python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]

## 2. Criando um container

Agora que já temos o Ansible instalado em nosso server, vamos provisionar um container para funcionar como um nó a ser gerenciado pelo Ansible. Para isto, vamos utilizar a mesma arquitetura de nosso laboratório com o Chef, onde utilizamos containers sendo executados no `chef-client` simulando hosts.

Vamos então criar um novo `Dockerfile` que além de habilitar o SSH em nosso container, também realizará a instalação do `Python`, pré requisito para o funcionamento do Ansible.

No servidor `chef-client` crie um novo diretório, chamado `ansible`:

    $ mkdir ~/ansible && cd ~/ansible

Neste diretório, vamos criar o novo `Dockerfile` e inserir o seguinte conteúdo:

    FROM ubuntu:16.04

    RUN apt-get update && apt-get install -y openssh-server python
    RUN mkdir /var/run/sshd
    RUN echo 'root:123456' | chpasswd
    RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

    # SSH login fix. Otherwise user is kicked off after login
    RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

    ENV NOTVISIBLE "in users profile"
    RUN echo "export VISIBLE=now" >> /etc/profile

    EXPOSE 22 80
    CMD ["/usr/sbin/sshd", "-D"]

>Note que estamos utilizando o segundo comando para realizar a instalação do Python.

Vamos criar uma nova imagem utilizando o comando:

    $ sudo docker build -t ansible-node .

>Não se esquece de adicionar o . no final de seu comando, afim de indicar aonde se encontra o Dockerfile.

Agora, vamos iniciar um novo container, que irá simular um node a ser gerenciado pelo Ansible:

    $ sudo docker run -d -P --name ansible-client ansible-node

Feito isto, liste o status de seu comando utizando o comando:

    $ sudo docker ps

A saída deverá ser semelhante a esta abaixo, indicando que seu container está em execução:

    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                                          NAMES
    a33762878487        ansible-node        "/usr/sbin/sshd -D"   3 seconds ago       Up 3 seconds        0.0.0.0:32770->22/tcp, 0.0.0.0:32769->80/tcp   ansible-client


Novamente, obtenha a porta mapeada para seu container, através do comando:

    $ sudo docker port ansible-client 22

Anote esta porta, pois utilizaremos a mesma para nos conectarmos ao container utilizando o Ansible.


## 3. Configurando o Ansible

Agora que já temos um container sendo executado, vamos retornar ao `chef-server`, onde instalamos o Ansible, e configurar o mesmo. O primeiro passo, é editar o arquivo `/etc/ansible/hosts` e inserir os endereços IP dos nodes que vamos gerenciar. Neste caso, vamos criar um grupo de nodes chamado `webservers` e adicionar nosso container a este grupo. Para isto, edite o arquivo `/etc/ansible/hosts` utilizando o comando:

    $ sudo vi /etc/ansible/hosts

E insira as seguintes linhas ao final do mesmo:

    [webservers]
    ansible-container ansible_port=<porta do docker> ansible_host=<IP do chef client> ansible_user=root

Agora, vamos criar uma variável de ambiente para que o Ansible ignore a chave SSH do host. Faremos isto em nosso laboratório pois por padrão o Ansible sempre verifica a chave de cada servidor antes de realizar acesso ao mesmo, assim, se a chave de um determinado host muda, ele irá retornar um erro. Como vamos utilizar o mesmo servidor com diferentes containers, vamos alterar o nosso ambiente para que esta checagem não ocorra. Para isto, execute o seguinte comando, afim de criar uma variável de ambiente que fará com que o Ansible ignore esta checagem:

    $ export ANSIBLE_HOST_KEY_CHECKING=False

Após salvar as alterações no arquivo e criar a variável de ambiente, execute o seguinte comando para validar o funcionamento:

    $ ansible all -m ping --ask-pass

O Ansible irá solicitar a senha do usuário `root` criado junto com nosso container. Insira a senha `123456` e a saída deverá ser igual a seguinte:

    ansible-container | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }

Isso significa que o `Ansible` conseguiu se conectar ao nosso container devidamente.

## 4. Criando um Playbook

`Playbooks` são arquivos utilizados pelo `Ansible` para realizar alterações de forma programática em nossa infra estrutura. Estes arquivos são escritos em YAML e possuem uma sintaxe que deve ser seguida. Cada playbook é composto por um ou mais `plays`.

O primeiro passo, é criarmos uma nova pasta onde vamos manter os arquivos que vamos utilizar. No `chef-server` execute o seguinte comando:

    $ mkdir ~/ansible && cd ~/ansible

Agora crie um arquivo chamado `motd-playbook.yml` e insira o seguinte conteúdo:

    - hosts: webservers
      tasks:
      - name: Create new file
        shell: |
          echo 'Arquivo criado pelo Ansible!' > /tmp/motd

Após criarmos o nosso primeiro playbook, vamos executar o mesmo utilizando o Ansible. Para isto, ainda no `chef-server`, execute o seguinte comando:

    $ ansible-playbook motd-playbook.yml --ask-pass

O Ansible irá solicitar novamente a senha de SSH, e em seguida irá executar o playbook. O output da execução do playbook deverá ser semelhante a este:

    PLAY [webservers] *******************************************************************************************************

    TASK [Gathering Facts] **************************************************************************************************
    ok: [ansible-container]

    TASK [Create new file] **************************************************************************************************
    changed: [ansible-container]

    PLAY RECAP
    ansible-container          : ok=2    changed=1    unreachable=0    failed=0   

Agora, vamos verificar se o nosso playbook foi executado com sucesso. Acesse o `chef-client` e em seguida execute o seguinte comando para acessar o container em execução:

    $ sudo docker exec -ti ansible-client /bin/bash

Agora, dentro do container, acesse o diretório `/tmp` através do comando:

    # cd /tmp

E verifique o conteúdo do arquivo motd:

    # cat motd

## 5. Instalando o apache através do Ansible

Agora que já vimos como criar um novo arquivo utilizando um playbook do Ansible, vamos criar um novo playbook que irá instalar o Apache em nosso container.

Para isto, no `chef-server`, crie um arquivo chamado `apache-playbook.yml` com o seguinte conteúdo:

    - hosts: webservers
      sudo: yes
      tasks:
        - name: install apache2
          apt: name=apache2 update_cache=yes state=latest

        - name: enabled mod_rewrite
          apache2_module: name=rewrite state=present
          notify:
            - restart apache2

      handlers:
        - name: restart apache2
          service: name=apache2 state=restarted

Após criar o arquivo, vamos então executar o playbook utilizando o Ansible através do comando:

    $ ansible-playbook apache-playbook.yml --ask-pass

Agora vamos validar a execução do playbook. Acesse o servidor `chef-client` e execute o seguinte comando para verificar qual a porta do host mapeada para a porta 80 do container:

    $ sudo docker port ansible-client 80

Acesse então em seu browser o endereço IP do `chef-client`:`Porta do container` para verificar a sua instalação do apache server:

![apache server](/08-ComecandoComAnsible/images/apache-server.png)
