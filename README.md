>Esse é um projeto que criei para minha aula sobre Ansible na Alura. <h1>

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
O Ansible trabalha com o princípio da idempotência, ou seja, ele pode ser executado várias vezes, mas não vai alterar nada se tudo estiver igual após a primeira execução.
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
- present ← (default) -> criar  

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
*priv: 'wordpress_db.*:ALL'*= permissões do usuário [banco.[tabela]:[tipo de permissão][localhost]]  
*state: present*= estado que cria o bando. Ver abaixo a lista de estados  
*become: yes*= o Mysql8 não aceita o usuário root, por isso o módulo precisa ser executado como sudo. 
 
# Importante
No módulo **mysql_user** podemos ter os seguintes state:
- present-> cria o usuário
- absent -> apaga o usuário

Fim