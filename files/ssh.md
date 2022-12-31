# Roteiro SSH-Server

### Login nas VMs

* Usuário da VM: ``administrador``
* Senha da VM: ``adminifal``

## Instalando o servidor SSH

```bash
$ sudo apt update       # atualiza as definições e versões de pacotes/bibliotecas dos repositórios do ubuntu
$ sudo apt upgrade -y   # atualiza os pacotes com as novas definições e versões 
```

### Instale o SSH Server

```bash
$ systemctl status ssh
$ sudo apt-get install openssh-server
$ systemctl status ssh
```

### Verifique o status das portas do sistema
```bash
$ netstat -an | grep LISTEN.  #verifique as conexões TCP na porta 22 se está como LINSTENING
```

### Firewall 
* Para garantir o funcioamento correto do controle de acesso devemos configurar o firewall para permitir conexões remota via protocolo SSH, na porta 22.
 
```bash
$ sudo ufw status
$ sudo ufw allow ssh.    # ativa o ssh no firewall UFW do ubuntu.
$ sudo ufw status
```
    
* Para ativar o firewall:
```bash 
$ sudo ufw enable
```


### Acessando uma VM remotamente:

* Exemplo: $ ssh ``<user>``@``<ipServidorRemoto>``
```bash
ssh administrador@10.9.24.102
```