# Ansible: From Zero to Hero

## Pre-requisites

Docker

Docker-compose

Ansible



## Fonts

https://github.com/spurin/diveintoansible-lab

https://www.udemy.com/course/diveintoansible/learn/lecture/23652096#overview

https://docs.ansible.com/



## Configuracao de Chave-SSH

Acesso SSH por chave

```bash
ssh-keygen
ssh-copy-id user@host
```

#### Compartilhamento da chave para diversos hosts

```bash
sudo apt install sshpass -y # Pacote para fazer o passe
echo password > password.txt

# Compartilhar a chave com os Hosts
# ubuntu1 ubuntu2 ubuntu3 => Users: ansible, root
# centos1 centos2 centos3 => Users: ansible, root
for user in ansible root; do
	for os in ubuntu centos; do
		for instance in 1 2 3; do 
			sshpass -f password.txt ssh-copy-id -o StrictHostKeyChecking=no ${user}@${os}${instance};
        done
    done
done
```

![ssh-keys](./images/ssh-keys.png)

### Verificação das comunicação com os hosts

```bash
ansible -i,ubuntu1,ubuntu2,ubuntu3,centos1,centos2,centos3 all -m ping
# -i -> inverntory (grupo a receber o comando)
# -m -> module (dafault=command)
```



## Arquivo de configuração 

```bash
ansible --version
#config file = none

# Ordem de prioridade
1 - ANSIBLE_CONOFIG # ENV var pointing to file
2 - ./ansible.cfg # Current Directory
3 - ~/ansible/.ansible.cfg # (Hidden File)
4 - /etc/ansible/ansible.cfg
```

