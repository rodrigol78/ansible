**Esse é um projeto que criei para minha aula sobre Ansible na Alura.**

# Comando inicial <h10>
```
ansible -vvv wordpress -u vagrant --private-key .vagrant/machines/wordpress/virtualbox/private_key -i hosts -m shell -a 'echo Hello, World'
```
**Explicação:**  
*ansible -vvv* = roda o ansible no modo verbose.  
*wordpress* = grupo criado no arquivo hosts  
*-u vagrant* = usuário do vagrant  
*--private-key .vagrant/machines/wordpress/virtualbox/private_key* = arquivo de chave primária já criado pelo vagrant  
*-i hosts* =  indica o arquivo hosts que será usado  
*-m shell* = indica o módulo do ansible a ser usado (nesse caso o shell)  
*-a 'echo Hello, World'* = argumento que foi passado pro sheel (nesse caso o comando echo 'Hello, World'  

# Criando o primeiro Playbook - Módulo shell<h10>
Playbook tem a extensão .yml, sua função é passar uma série de comandos para o ansible.  
**Ex:**  
```
---
- hosts: all
    tasks: 
     - shell: 'echo hello > /vagrant/world.txt'
```

**Explicação**
*Começa-se com uma sequência de ---  
*- hosts: all* = indicamos quais hosts receberação os comandos
*tasks: listas de comandos serão executados nos hosts indicados
*- shell: 'echo hello > world.txt'* = comando usando shell

# Comando para usar o playbook <h10>
```
ansible-playbook provisioning.yml -u vagrant -i hosts --private-key .vagrant/machines/wordpress/virtualbox/private_key
```

**Explicação**
*ansible-playbook*= comando para rodar o playbook  
*provisioning.yml*= nome do arquivo .yml que contem os comandos a serem executados  
*-u vagrant* = usuário do vagrant  
*-i hosts* =  indica o arquivo hosts que será usado  
*--private-key .vagrant/machines/wordpress/virtualbox/private_key* = arquivo de chave primária já criado pelo vagrant  

# Instalando dependências - Módulo apt <h10>
```
- hosts: all
  tasks: 
    - name: 'Instala o PHP7.4'
      apt:
        name: php7.4
        state: latest
      become: yes

    - name: 'Instala Apache2'
      apt:
        name: apache2
        state: latest
      become: yes
    
    - name: 'Instala o modphp'
      apt:
        name: libapache2-mod-php7.4
        state: latest
      become: yes
```
**Explicação**  

*- name: 'Instala PHP7.4'*= Descreve o que a task irá realizar  
*apt:*= administra pacotes do debian e suas derivações  
*name: php7.4*= nome do pacote que desejamos instalar  
*state: latest*= estado desejado da execução de uma *task* nesse caso, a última versão  
*become: yes*= indica que o comando será rodado como root  

# Importante <h1>
```
O Ansible trabalha com o princípio da idempotência,
ou seja, ele pode ser executado várias vezes,
mas não vai alterar nada se tudo estiver igual após a primeira execução.
```  
No módulo **apt** podemos ter os seguintes state:
- absent -> desinstalar um pacote
- build-dep -> instalar o pacote com as dependências dele
- latest -> o pacote mais atual
- present ← (default) -> criar 


# Simplificando o Playbook com with_items <h10>  
```
 tasks:
    - name: "Instala pacotes de dependencias do SO"
      apt:
        name: "{{ item }}"
        state: latest
      become: yes
      with_items:
        - php7.4
        - apache2
        - libapache2-mod-php7.4
        - php7.4-gd
        - libssh-4
        - mysql-server-8.0
        - php7.4-mysql
```
**Explicação**  
*- name: "Instala pacotes de dependencias do SO"*= nome da tarefa que será executada  
*apt:*= instala pacotes apt  
*name: "{{ item }}"*= nome do pacote que irá instar usa o comando "{{ item }}" para guardar a lista de itens que serão utilizadas.  
*state: latest* = estado final (última versão)  
*become: yes* = executa como root  
*with_items:* - como que chama a lista de itens  
*- php7.4*  
*- apache2*  
*- libapache2-mod-php7.4*  
*- php7.4-gd*  
*- libssh-4*  
*- mysql-server-8.0*  
*- php7.4-mysql*  = lista de itens a serem instalados  

# Importante <h1>
Também existe esta maneira abaixo de ser instalar vários pacotes:  
```
- name: 'Instala pacotes do sistema operacional'
      apt:
        name:
        - php5
        - apache2
        - libapache2-mod-php5
        state: latest
      become: yes
```
# Passando usuário e chave privada através do arquivo hosts <10>
```
[wordpress]
192.168.100.100 ansible_user=vagrant ansible_ssh_private_key_file="/mnt/Dados/rodrigo/VMs/ambiente_dev/ansible/.vagrant/machines/wordpress/virtualbox/private_key"
```
**Explicação**
*[wordpress]*= nome do projeto  
*192.168.100.100*= ip da máquina  
*ansible_user=vagrant*= nome de usuário do ssh
*ansible_ssh_private_key_file="/mnt/Dados/rodrigo/VMs/ambiente_dev/ansible/.vagrant/machines/wordpress/virtualbox/private_key"*= caminho completo da chave privada.  
  
Assim o comando do ansible ficaria reduzido a:
```
ansible-playbook provisioning.yml -i hosts
``` 
# Configurando o banco de dados - Módulo mysql_db <h10>
# MUITO IMPORTANTE  
Durante o processo de execução dos módulos do mysql tive alguns problemas.  
Para resolvê-los tive que adicionar algumas tasks antes dos módulos mysql:  
```
- name: 'apt-get Atualiza o repositório e o cache'
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600
      become: yes
```
**Explicação**
*apt:*= módulo de instalação de pacote  
*update_cache=yes*= atualiza o cache de pacotes  
*force_apt=yes*= força a atualização do repositório de pacotes  
*cache_valid_time=3600*= perído de duração do cache  
Obs: coloco no início para as próximas tasks, se precisarem de pacotes já utilizará os atualizados.  

```
- name: "Instala o Python3"
      apt:
        name: python3-pip
        state: present
      become: yes

    - name: "Instala o PyMySQL"
      pip:
        name: PyMySQL
      become: yes
```
**Explicação**

O primeiro instala o Python3  
O segundo instala o dentro do Python3 o pacote PyMySQL  

**Criando o banco de dados**
```
 - name: 'Cria o banco de dados mysql - wordpress_db'
      mysql_db:
        login_unix_socket: /var/run/mysqld/mysqld.sock
        name: wordpress_db
        state: present 
      become: yes
```
**Explicação**  
*mysql_db:*= módulo de criação do banco  
*login_unix_socket: /var/run/mysqld/mysqld.sock*=tive que adicionar para o mysqld rodar.  
*name: wordpress_db*= nome do banco  
*state: present*= estado que cria o bando. Ver abaixo a lista de estados  
*become: yes*= o Mysql8 não aceita o usuário root, por isso o módulo precisa ser executado como sudo.  

# Importante  
No módulo **mysql_db** podemos ter os seguintes state:
- absent -> destruir
- dump -> fazer cópia
- import -> importar através de um script
- present -> (default) -> criar  

**Criando usuário para o banco de dados**
```
      - name: 'Cria usuário do MySQL'
      mysql_user:
        login_unix_socket: /var/run/mysqld/mysqld.sock
        name: wordpress_user
        password: teste
        priv: 'wordpress_db.*:ALL'
        state: present
      become: yes
```
**Explicação**  
*mysql_user:*= módulo de criação de usuário
*login_unix_socket: /var/run/mysqld/mysqld.sock*=tive que adicionar para o mysqld rodar.  
*name: wordpress_user*= nome do usuário a ser criado  
*password: teste*= senha do usuário  
*priv: 'wordpress_db.*:ALL'*= permissões do usuário [banco].[tabela]:[tipo de permissão][localhost]]  
*state: present*= estado que cria o bando. Ver abaixo a lista de estados  
*become: yes*= o Mysql8 não aceita o usuário root, por isso o módulo precisa ser executado como sudo. 
 
# Importante
No módulo **mysql_user** podemos ter os seguintes state:
- present -> cria o usuário
- absent -> apaga o usuário

# Baixando e descompactando arquivo Wordpress  
 ```
 - name: 'Download do Wordpress'
      get_url:
        url: 'https://wordpress.org/latest.tar.gz'
        dest: '/tmp/wordpress.tar.gz'

    - name: 'Descompacta o wordpress'
      unarchive:
        src: '/tmp/wordpress.tar.gz'
        dest: '/var/www/'
        remote_src: True
      become: yes
 ```
 **Explicação**  
 *get_url:*= módulo baixa arquivo do site (pode ser https,http ou ftp)  
 *url:'https://wordpress.org/latest.tar.gz'*= endereço de onde baixamos o arquivo  
 *dest: '/tmp/wordpress.tar.gz'*= destino e nome que desejamos dar ao arquivo baixado  
   
*unarchive:* = módulo que decompacta arquivos  
*src: '/tmp/wordpress.tar.gz'*= qual arquivo queremos descompactar  
*dest: '/var/www/'*= destino onde queremos descompactar o arquivo  
*remote_src: True*= se o arquivo está na VM utilizamos **True** se na máquina controladora usamos **False**  
*become: yes*= executa o comando como sudo  

# Copiando e modificando o nome do arquivo de configuração Wordpress  
 ```
 - name: 'Copia o arquivo de configuração de exemplo para a pasta desejada'
      copy:
        src: '/var/www/wordpress/wp-config-sample.php'
        dest: '/var/www/wordpress/wp-config.php'
        remote_src: yes
      become: yes
 ```
 **Explicação**  
*copy:*= módulo que copia arquivos  
*src: '/var/www/wordpress/wp-config-sample.php'*= arquivo(s) que se deseja copiar  
*dest: '/var/www/wordpress/wp-config.php'*= destino que se deseja copiar o arquivo (basta colocar o novo nome se quiser mudar)  
*remote_src: yes*= se o arquivo está na VM utilizamos **yes** se na máquina controladora usamos **no**  
*become: yes*= executa o comando como sudo 

# Substituindo valores dentro do arquivo de configuração Wordpress  
 ```
 - name: 'Configura o wp-config com as entradas do banco de dados'     
      replace:
        path: '/var/www/wordpress/wp-config.php'
        regexp: "{{ item.regex }}"
        replace: "{{ item.value }}"
      with_items:
          - { regex: 'database_name_here', value: 'wordpress_db'}
          - { regex: 'username_here', value: 'wordpress_user'}
          - { regex: 'password_here', value: 'teste'}
      become: yes
 ```
 **Explicação**  
*replace:*= módulo que substitui termos no arquivo  
*path: '/var/www/wordpress/wp-config.php'*= arquivo que deseja substituir valores  
*regexp: "{{ item.regex }}"*= termo que será substituido. Nesse caso usamos uma variável que representa o valor.  
*replace: "{{ item.value }}"*= termo que substituirá o termo original. Ness caso usamos uma variável que representa o valor.  
*with_items:* módulo que utiliza uma lista de itens*  
*- { regex: 'database_name_here', value: 'wordpress_db'}*= item 1 da lista que contem **variável01**, **conteúdo01**, **variável02**, **conteúdo02**  
*- { regex: 'username_here', value: 'wordpress_user'}*= *= item 2 da lista que contem **variável01**, **conteúdo01**, **variável02**, **conteúdo02**  
*- { regex: 'password_here', value: 'teste'}*= *= item 3 da lista que contem **variável01**, **conteúdo01**, **variável02**, **conteúdo02**  
*become: yes* = executa o comando como sudo


# Substituindo valores dentro do arquivo de configuração 000-default.conf (configuração do Apache)
```
- name: 'Configura o 000-default.conf com o caminho correto'     
      replace:
        path: '/etc/apache2/sites-available/000-default.conf'
        regexp: '/var/www/html'
        replace: '/var/www/wordpress'
      become: yes
      notify:
        - reinicia o apache
```   
**Explicação**  
*replace:*= módulo que substitui termos no arquivo  
*path: '/etc/apache2/sites-available/000-default.conf'*= arquivo que deseja substituir valores  
*regexp: '/var/www/html'*= termo que será substituido.  
*replace: '/var/www/wordpress'*= termo que substituirá o termo original  
*become: yes*= executa o comando como sudo  
# Muito Importante
*notify:*= módulo que chama um handler que foi definido no início do algorítimo (olhar abaixo a explicação)  
*- reinicia o apache*= nome do handler que será executado  

```
handlers:
    - name: reinicia o apache
      service: 
        name: apache2
        state: restarted
      become: yes
```
**Explicação Handlers**  
*handlers:*= nome do módulo que gera os handlers (espécie função). Precisa estar no mesmo nível de identração do **Tasks**  
*- name: reinicia o apache*= nome do handler. Sempre que for necessário executar o handler, esse será o nome que deverá ser citado  
*service:*= módulo do handler que executa o daemond do linux  
*name: apache2*= serviço que executa o os serviços do apache  
*state: restarted*= estado que considera sucesso (reiniciado)  
*become: yes*= executa o comando como sudo  

# Dividindo os servidores em grupos diferentes dentro de hosts
**Arquivo host**  
```
[wordpress]
192.168.100.100 ansible_user=vagrant ansible_ssh_private_key_file="/mnt/Dados/rodrigo/VMs/ambiente_dev/ansible/.vagrant/machines/wordpress/virtualbox/private_key"
[database]
192.168.100.99 ansible_user=vagrant ansible_ssh_private_key_file="/mnt/Dados/rodrigo/VMs/ambiente_dev/ansible/.vagrant/machines/mysql/virtualbox/private_key"
```
**Explicação**  
*[wordpress]*= defini o grupo de hosts, o que vier abaixo estará dentro deste grupo.  

**Arquivo .yml**
```
- hosts: database
  handlers:
    - name: reinicia o mysql
      service: 
        name: mysql
        state: restarted
      become: yes

  tasks:
    - name: "Instala pacotes de dependencias do SO"
      apt:
        name: "{{ item }}"
        state: latest
      become: yes
      with_items:
        - libssh-4
        - mysql-server-8.0
        - python3-pip
```
**Explicação**  
Como qualquer grupo de hosts, se começa com **hosts: [nome do host]** e abaixo se coloca o task e as tarefas.  

# Trabalhando com variáveis
Arquivo .yml dentro da pasta group_vars. Crie um geral com o nome de all.yml como e exemplo abaixo:
```
wp_username: wordpress_user
wp_db_name: wordpress
wp_password: teste
mysql_host: '192.168.100.99'
```
Ou crie um com o nome do grupo de hosts, que se encontra dentro do arquivo hosts.  

**Declarando dentro do arquivo de provision.yml**
```
- name: 'Configura o wp-config com as entradas do banco de dados'     
      replace:
        path: '/var/www/wordpress/wp-config.php'
        regexp: "{{ item.regex }}"
        replace: "{{ item.value }}"
      with_items:
          - { regex: 'database_name_here', value: "{{ wp_db_name }}"}
          - { regex: 'username_here', value: "{{ wp_username }}"}
          - { regex: 'password_here', value: "{{ wp_password }}"}
          - { regex: 'localhost', value: "{{ mysql_host }}"}
      become: yes
```
**Explicação**  
*{ regex: 'database_name_here', value: "{{ wp_db_name }}"}*= coloque o nome entre " e {{.  

**Importante**  
Existem algumas regras que devemos considerar na hora de declarar uma varíavel (como em qualquer outra ferramenta ou linguagem de programação). Por exemplo, uma variável não deve ter um ponto, espaço ou hífen no nome. Todas as declarações abaixo são inválidas:

```
foo-var: 'nao_pode_ter_hifen'
foo var: 'nem_ter_espaco'
foo.var: 'tambem_nao_pode_ter_ponto'
12foovar: 'nao_pode_ter_numero_no_inicio'
```

# Utilizando Templates
**Importante**  
Templates são arquivos que precisam ter a extensão .j2 e ficar dentro da pasta templates dentro do projeto.  
**Arquivo 000.default.conf**
```
ServerAdmin webmaster@localhost
	DocumentRoot {{ wp_installation_dir }}
```
**Explicação**  
*DocumentRoot {{ wp_installation_dir }}*= substituímos o caminho do DocumentRoot por uma variável. (Essa variável precisa ser declarada em um arquivo .yml dentro de group_vars)  

**Arquivo provision.yml**
```
- name: 'Configura o 000-default.conf com o caminho correto'
      template:
        src: 'templates/000-default.conf.j2'
        dest: '/etc/apache2/sites-available/000-default.conf'
      become: yes
      notify:
         - reinicia o apache
```
**Explicação**  
*template:*= módulo que utiliza os templates  
*src: 'templates/000-default.conf.j2'*= arquivo de template (lembrando que precisa ter a extensão .j2 e estar dentro da pasta template)  
*dest: '/etc/apache2/sites-available/000-default.conf'*= destino do arquivo template com o novo noome.  
*become: yes*= executa o módulo como sudo  
*notify:*= handler  
*- reinicia o apache*= nome do handler declarado anteriormente.  

Fim