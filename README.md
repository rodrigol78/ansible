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

** Explicação**
*ansible-playbook*= comando para rodar o playbook
*provisioning.yml*= nome do arquivo .yml que contem os comandos a serem executados
*-u vagrant* = usuário do vagrant  
*-i hosts* =  indica o arquivo hosts que será usado
*--private-key .vagrant/machines/wordpress/virtualbox/private_key* = arquivo de chave primária já criado pelo vagrant  

Fim