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

# Criando o primeiro Playbook <h10>
Playbook tem a extensão .yml, sua função é passar uma série de comandos para o ansible.  
**Ex:**  
```
---
- hosts: all
    tasks: 
     - shel: 'echo hello > /vagrant/world.txt'
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

# Instalando dependências <h10>
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

Fim