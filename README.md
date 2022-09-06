>Esse é um projeto que criei para minha aula sobre Ansible na Alura. <h1>

# Comando inicial <h10>
```
ansible -vvv wordpress -u vagrant --private-key .vagrant/machines/wordpress/virtualbox/private_key -i hosts -m shell -a 'echo Hello, World'
```
**Explicação:**  
*ansible -vvv* = roda o ansible no modo verbose.  
*wordpress* = grupo criado no arquivo hosts  
*-u vagrant* = usuário do vagrant  
*--private-key* .vagrant/machines/wordpress/virtualbox/private_key = arquivo de chave primária já criado pelo vagrant  
*-i hosts* =  indica o arquivo hosts que será usado  
*-m shell* = indica o módulo do ansible a ser usado (nesse caso o shell)  
*-a 'echo Hello, World'* = argumento que foi passado pro sheel (nesse caso o comando echo 'Hello, World'  

Fim