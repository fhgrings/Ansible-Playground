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



## Inventory

Parte mais ampla da definição dos Hosts controlados pelo Ansible Master.

```bash
etc/ansible/hosts # --> Arquivo default de configuração do inventario
```

Sobreposição do arquivo

```bash
# ansible.cfg
[defaults]
inventory = hosts #-> Sobrepõe para ./hosts
host_key_checking = False
```

#### Comandos

```bash
ansible {grupo} --list-hosts
#          '---------> all, ubuntu, centos, linux
```



#### Hosts Simple file

```bash
#host
[controller]
ubuntu-c ansible_connection=local

[centos] # -> Grupo
centos1  # -> Host
centos2  # -> Host
centos3:2222 # Porta SSH 2222

[ubuntu]
ubuntu1  ansible_user=root # Loga como root 
ubuntu2  ansible_become=true ansible_become_pass=password  # Loga como user ansible com privilégios
ubuntu3  ansible_port=2222 # Porta SSH 2222

```

#### Hosts Advnced File

```bash
[control]
ubuntu-c ansible_connection=local

[centos]
centos1 ansible_port=2222
centos[2:3] # -> REGEX centos2, centos3

[centos:vars] # Variaveis para grupo
ansible_user=root

[ubuntu]
ubuntu[1:3]

[ubuntu:vars]
ansible_become=true
ansible_become_pass=password

[linux:children] # Grupo de Grupos
centos
ubuntu
```

#### Hosts.yml

```yaml
---
control:
  hosts:
    ubuntu-c:
      ansible_connection: local
centos:
  hosts:
    centos1:
      ansible_port: 2222
    centos2:
    centos3:
  vars:
    ansible_user: root
ubuntu:
  hosts:
    ubuntu1:
    ubuntu2:
    ubuntu3:
  vars:
    ansible_become: true
    ansible_become_pass: password
linux:
  children:
    centos:
    ubuntu:
...
```

#### Convert .yml to json

```bash
python3 -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin, Loader=yaml.FUllLoader), sys.stdout, indent=4)' < hosts.yaml L hosts.json
```

#### Hosts.json

```json
{
    "control": {
        "hosts": {
            "ubuntu-c": {
                "ansible_connection": "local"
            }
        }
    }, 
    "ubuntu": {
        "hosts": {
            "ubuntu1": null, 
            "ubuntu2": null, 
            "ubuntu3": null
        }, 
        "vars": {
            "ansible_become": true, 
            "ansible_become_pass": "password"
        }
    }, 
    "centos": {
        "hosts": {
            "centos3": null, 
            "centos2": null, 
            "centos1": {
                "ansible_port": 2222
            }
        }, 
        "vars": {
            "ansible_user": "root"
        }
    }, 
    "linux": {
        "children": {
            "centos": null, 
            "ubuntu": null
        }
    }
}
```





## Modules

