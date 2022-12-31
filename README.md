### Roteiro para a instalação dos serviços de redes.

## Integrantes

* Ana Beatriz
* Karllisson Henrique
* Lívia Beatriz
* Maísa Lira

## Sumário 
 
 * <a href="#vm-req">Requisitos</a>
 * <a href="#vm-configs"> Definição da rede </a>
 * <a href="#vm-gw-network"> Configuração do Gateway </a>
 * <a href="#vm-smb-network"> Configuração do Samba</a>
 * <a href="#vm-ns1-network"> Configuração do NameServer1 (master)</a>
 * <a href="#vm-ns2-network"> Configuração do NameServer2 (slave)</a>
 * <a href="#vm-config-network"> Configuração da rede interna</a>

<section id="vm-req">

## Requisitos

Para a realização do projeto, faz-se necessário o conhecimento prévio acerca de acesso remoto através de SSH, além da configuração de VPNs no sistema operacional. Portanto, recomenda-se que o leitor siga o manual para ![SSH](https://github.com/alaelson/2022-924-notasdeaula/blob/main/Aula.924-2022.07.21.md) e o manual para ![VPN](https://github.com/alaelson/2022-924-notasdeaula/blob/main/Aula.924-2022.11.04.md).

<br>
Aplicações necessárias:
  * OpenVPN3
  * OpenSSH-Server 
</section>


<section id="vm-configs">

## Definição da rede

Faz-se necessário declarar um IP único na rede para todas as VMs de cada computador.
Para isso, foram criadas tabelas com os dados referentes a rede:
  
Tabela 1. <br>

| DESCRICAO        | IP (ens160)     | IP (ens192)      |
|------------------|-----------------|------------------|
| Endereço da rede | 10.9.24.0/24    | 192.168.24.48/28 |
| Gateway          | 10.9.24.1       | 192.168.24.51    |
| Broadcast        | 10.9.24.255     | 192.168.24.63    |
| Máscara          | 255.255.255.0   | 255.255.255.240  |
<br>

Tabela 2. <br>

| HOSTNAME   | IP (ens160) | IP (ens192)   |          FQDN                     | ALIASE | 
|------------|-------------|---------------|-----------------------------------|--------|
| GRUPO4-VM1 | 10.9.24.102 | 192.168.24.51 | gw.grupo4.turma924.ifalara.local  | gw     |
| GRUPO4-VM2 | 10.9.24.111 | 192.168.24.52 | smb.grupo4.turma924.ifalara.local | samba  |
| GRUPO4-VM3 | 10.9.24.114 | 192.168.24.53 | ns1.grupo4.turma924.ifalara.local | ns1    |
| GRUPO4-VM4 | 10.9.24.115 | 192.168.24.54 | ns2.grupo4.turma924.ifalara.local | ns2    |
| GRUPO4-VM5 | 10.9.24.217 | 192.168.24.55 | www.grupo4.turma924.ifalara.local | www    |
| GRUPO4-VM6 | 10.9.24.218 | 192.168.24.56 | bd.grupo4.turma924.ifalara.local  | bd     |
<br>

### Credenciais da rede

| USER          | PASSWORD   |
|---------------|------------|
| administrador | adminifal  |
| aluno         | ifal@aluno |

### Acesso à VM
Para realizar o realizar o acesso ao servidor, deve-se usar o arquivo ![vpn924.labredes](https://github.com/alaelson/2022-924-notasdeaula/blob/main/Aula.924-2022.11.04.md), que pode ser baixado com o comando:
``` bash
curl -fsSL https://www.dropbox.com/s/hb8ee3kiwkhutbl/vpn924.labredes.arapiraca.ifal.edu.br.ovpn?dl=0 > ~/vpn924.labredes.arapiraca.ifal.edu.br.ovpn
```
<br>
Após realizar as configurações do arquivo, seguindo o manual, está pronto para acessar a VPN com o comando:

```bash
openvpn3 session-start --config CONFIG_NAME
```

Exemplo:
```bash
openvpn3 session-start --config vpn924.labredes
```
Exemplo:
![acesso_via_ssh](/Images/access_via_ssh.png)
---

Para realizar o acesso, o usuário deve logar com as suas credeciais, definidas pelo professor, onde cada pessoa apresenta um nome de usuário e uma senha (correspondente ao número de matrícula). <br>
Com o acesso realizado, já é possívá é possível acessar os servidores do grupo. Vale destacar que os servidores estão configurados com duas interfaces: a rede externa (ens160) e a rede interna (ens192). Para acessar o servidor do seu terminal, deve-se usar o SSH para o IP da rede externa do servidor em específico:

```bash
ssh <user>@<ip_rede_externa>
```

Com o acesso via SSH realizado, o usuário verá esta tela de boas-vindas:
![tela_boas_vindas](/Images/welcome_screen.png)

</section>


<section id="vm_gateway_network">

## Configuração do servidor Gateway como NAT

### Configuração do firewall/NAT

   * Para configurar um servidor como gateway de rede é necessário configurar o firewall do linux (iptables). Desse modo, segue o tutorial:

   1. habilitar o firewall e permitir o acesso ssh:
```bash
 $ sudo ufw enable
 $ sudo ufw allow ssh
```
   2. habilitar o encaminhamento de pacotes das interfaces WAN para LAN, ajustando-se os parâmetros no arquivo **/etc/ufw/sysctl.conf**, removendo-se a marca de comentário (#) da seguinte linha _# net/ipv4/ip_forward=1_

```bash
$ sudo nano /etc/ufw/sysctl.conf
``` 
![uncomment_ipv4_forward](/Images/uncomment_ipv4_forward.png)

    3. Configurar a interface de rede
```bash
$ sudo nano /etc/netplan/<file_name>.yaml 
```

``` bash
network:
    ethernets:
        ens160:
            dhcp4: false
            addresses: [10.9.24.102/24]
            gateway4: 10.9.24.1
            nameservers:
                addresses:
                    - 
                    - 
                search: []
        ens192:
            addresses: [192.168.24.51/28]
            dhcp4: false              
    version: 2
```

```bash
$ sudo netplan apply
$ ifconfig -a
```

   4. no ubuntu 18.04 o arquivo /etc/rc.local não existe mais. Então é necessário recriá-lo.
```bash
$ sudo nano /etc/rc.local
```

   5. A seguir, adicione o seguinte script no arquivo [/etc/rc.local](/files/rc.local)

   6. converte o arquivo em executável e o torna inicializável no boot
```bash
$ sudo chmod 755 /etc/rc.local
```

   7. verificar se o firewall está funcionando
```bash
$ sudo ufw status
```

   8.  reiniciar a máquina
```bash
$ sudo reboot
```
   

  9. Encaminhamento de portas para acesso externo à serviços da rede interna.
  
  * Para permitir que o serviço de compartilhamento de arquivos esteja disponível externamente, adicione as informações do IPTABLES sobre portas, IP e Interface no arquivo /etc/rc.local conforme o exemplo abaixo, depois reinicie a máquina:
  
   a. SAMBA: Para permitir que o serviço de compartilhamento de arquivos esteja disponível externamente:
        * Portas: 445 e 139
        * Interface Externa aqui é a WAN: ens160
        * IP do servidor = 10.9.24.111
        
```bash
#Recebe pacotes na porta 445 da interface externa do gw e encaminha para o servidor interno na porta 445
iptables -A PREROUTING -t nat -i ens160 -p tcp –-dport 445 -j DNAT –-to 10.9.24.111:445
iptables -A FORWARD -p tcp -d 10.9.24.111 –-dport 445 -j ACCEPT

#Recebe pacotes na porta 139 da interface externa do gw e encaminha para o servidor interno na porta 139
iptables -A PREROUTING -t nat -i ens160 -p tcp –-dport 139 -j DNAT –-to 10.9.24.111:139
iptables -A FORWARD -p tcp -d 10.9.24.111 –-dport 445 -j ACCEPT

```
   b. DNS: Para permitir que o serviço de resolução de nomes (DNS) esteja disponível externamente:
        * Porta: 53
        * Interface Externa aqui é a WAN: ens160
        * IP do servidor nameserver1 = 10.9.24.114
        
```bash
#Recebe pacotes na porta 53 da interface externa do gw e encaminha para o servidor DNS Master interno na porta 53
iptables -A PREROUTING -t nat -i ens160 -p udp --dport 53 -j DNAT --to 10.9.24.114:53
iptables -A FORWARD -p udp -d 10.9.24.114 --dport 53 -j ACCEPT
```
---

### Teste do Gateway com Traceroute
A partir da imagem, é possível observar que primeiro o traceroute passa pelo Gateway da interface **ens192** (rede interna) para, assim, passar pelo Gateway da interface **ens160** (rede externa).
![gateway_traceroute_test](/Images/gateway_traceroute_test.png)

</section>

<section id="vm_samba_network">

## Configuração do servidor Samba

3. instalar o servidor samba na MV samba-srv

```bash
$ sudo apt update
$ sudo apt install samba
```
   
   4. Verfificar se o samba está rodando
![where_is_samba](/Images/where_is_samba.png)
![samba_status](/Images/samba_status.png)

```bash
$ netstat -an | grep LISTEN
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN   
```

    5. Faça o backup do arquivo de configuração do samba e cria um arquivo novo somente com os comandos necessários.
    
```bash
$ sudo cp /etc/samba/smb.conf{,.backup}
$ ls -la
-rw-r--r--  1 root root 8942 Mar 22 20:55 smb.conf
-rw-r--r--  1 root root 8942 Mar 23 01:42 smb.conf.backup
$
$ sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
```

```bash
$ sudo nano /etc/samba/smb.conf
```
    6. Edite o arquivo de configuração /etc/samba/smb.conf

* adicione as interfaces da sua máquina na linha "interfaces = 127.0.0.1/8 ens160", separando os nomes das interfaces por espaços.

![interfaces_samba](/Images/interfaces_samba.png)

* Renicie o serviço smbd
    
```bash
$ sudo systemctl restart smbd
```

* modifica a pasta /samba/public para acesso a somente usuários do grupo sambashare
![access_samba](/Images/access_samba.png)

* É necessário vincular o usuário do S.O. ao Serviço Samba. Repita a senha de aluno ou crie uma senha nova somente para acessar o compartilhamento de arquivo. Neste caso repetiremos a senha do usuário aluno
    
```bash
$ sudo smbpasswd -a aluno
New SMB password:
Retype new SMB password:
Added user aluno.

$ sudo usermod -aG sambashare aluno

```
    
    * O Samba já está instalado, agora precisamos criar um diretório para compartilhá-lo em rede.
   
```bash
$ mkdir /home/<username>/sambashare/
$ sudo mkdir -p /samba/public
```
    * configure as permissões para que qualquer um possa acessar o compartilhamento público.

```bash
sudo chown -R nobody:nogroup /samba/public
sudo chmod -R 0775 /samba/public
sudo chgrp sambashare /samba/public

```
   7. Cliente do compartilhamento:
   
    * Em um máquina com Windows (também pode usar linux os MacOS) digite no Windows Explorer o endereço IP do servidor samba da seguinte forma:
    **\\ip_do_maquina**. Exemplo: \\10.9.24.111
   
![access_via_windows_samba]() 

</section>

<section id="vm_ns1_network">

## Configuração do NameServer1 (master)

* O BIND9 é a aplicação de DNS que roda no servidor.
   * Instalar o bind9 via apt-get
```bash
$ sudo apt-get install bind9 dnsutils bind9-doc 
```
   * Verifique o status do serviço:
```bash
$ sudo systemctl status bind9
```
   * Se não estiver rodando:
```bash
$ sudo systemctl enable bind9
```

   * Os arquivos do bind ficam na no diretório **/etc/bind**. 
![bind_dir](/Images/bind_dir.png)

### Zonas
   * As zonas são especificadas em arquivos **db**. Vamos criar um diretório para armazendar os arquivos de zonas, que sera o diretório ***/etc/bind/zones***  
```bash
$ sudo mkdir /etc/bind/zones
```

#### Criar arquivos db
   * Criar o arquivo **db** no diretório ***/etc/bind/zones***. 
   * Os arquivos **db** são bancos de dados de resolução de nomes, ou seja, quando se sabe o nome da máquina mas não se conhece o IP. Cada zona no DNS deve ter seu próprio arquivo **db**, por exemplo: a zona *meusite.com.br* terá o arquivo **db.meusite.com.br**, já a zona *outrosite.net* terá o arquivo **db.outrosite.net**. 
   * No nosso caso o domínio/zona local será labredes.ifalarapiraca.local. Assim o arquivo db será db.labredes.ifalarapiraca.local
   
##### zona direta
   * o arquivo db.labredes.ifalarapiraca.local conterá os nomes das máquinas do domínio labredes.ifalarapiraca.local
   * Para isso faremos uma cópia do arquivo /etc/bind/db.empty
```bash
$ sudo cp /etc/bind/db.empty /etc/bind/zones/db.labredes.ifalarapiraca.local 
```

##### zona reversa
   * Utilizado quando não se conhece o IP mas sabe-se o nome do host.
   * vamos criar a zona reversa a partir do arquivo /etc/bind/db.127
```bash
  $ sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.24.rev
```

   * Assim, o arquivo **db.10.9.14.rev** conterá a zona reversa da rede 10.9.24.0. 

   
### Editar arquivos db:

   #### zona direta: db.labredes.ifalarapiraca.local
   * edite o arquivo  **db.grupo4.turma924.ifalara.local** para adcionar as informações do seu domínio
      * As linhas iniciadas com **;** são comentários 
      
```bash   
    $ sudo nano db.grupo4.turma924.ifalara.local 
```

![bind_zone_config](/Images/bind_zone_config.png)
---

   #### zona reversa: db.10.9.14.rev
   * edite o arquivo **db.10.9.24.rev** para adcionar as informações da zona reversa
      * As linhas iniciadas com **;** são comentários.

![bind_reverse_zone_config](/Images/bind_reverse_zone_config.png)   
---

### Configuração do named.conf.local
   * Para ativar as zonas descritas nos arquivos **db** deve-se editar o arquivo de configuracão do bind para informar onde eles foram salvos. As zonas são adicionadas em **/etc/bind/named.conf.local**.
   
```bash
$ sudo nano /etc/bind/named.conf.local
```

![bind_named_conf](/Images/bind_named_conf.png)
---

### Verificação de sintaxe 
   * Para checar a sintaxe de configuração do BIND deve-se executar o comando named-checkconf. Este scritp checa os arquivos /etc/bind/named.conf.local.*

```bash
$ sudo named-checkconf
```

###  Verificar a sintaxe dos arquivos de dados
   * Para verificar se a formatação da sintaxe dos arquivos db está correta, utiliza-se o script named-checkconf da seguinte forma: ***named-check-zone <zone> <db_file>***

![bind_sintax_checkout](/Images/bind_sintax_checkout.png)
---

### Configure para somente resolver endereços IPv4

```bash
$ sudo nano /etc/default/named
```
- adicione a linha ***OPTIONS="-4 -u bind"***
![bind_ipv4_config]()
---

### Execute o BIND 
```bash
$ sudo systemctl enable bind9
$ sudo systemctl restart bind9
```
---
### Testando o servidor DNS:

#### Teste de configuração como cliente. 
   * Observe se os campos **DNS servers** e **DNS Domain** estão corretos.
```bash
$ systemd-resolve --status ens160
```
![dns_test](/Images/dns_test.png)
---

#### Teste o serviço DNS para a máquina ns1. 
   * Veja a resposta em **ANSWER SECTION**.
```bash
$ dig ns1.grupo4.turma924.ifalara.local
```
![dns_dig_ns1](/Images/dns_dig_ns1.png)
---

#### Teste o serviço DNS reverso para a máquina ns1. 
```bash    
$ dig -x 10.9.24.114
```

![dns_reverse_dig_ns1](/Images/dns_reverse_dig_ns1.png)
---

#### Teste o serviço DNS reverso para a máquina ns2. 
```bash  
$ dig -x 10.9.24.115
```
![dns_reverse_dig_ns2](/Images/dns_reverse_dig_ns2.png)
---
</section>

<section id="vm_ns2_network">

## Configuração do NameServer2 (slave)

### Configurar e instalar servidor DNS secundário (slave)
```bash
$ sudo apt-get install bind9 dnsutils bind9-doc -y
```

   * Verifique o status do serviço:
```bash
$ sudo systemctl status bind9
```
   * Se não estiver rodando:
```bash
$ sudo systemctl enable bind9
```

### configuração de zonas
```bash
$ sudo nano /etc/bind/named.conf.local
```
![bind_named_ns2](/Images/bind_named_ns2.png)
---

### Checagem de sintaxe

```bash
$ sudo named-checkconf
```

</section>

<section id="vm-config-network">

## Configuração da rede interna

De modo a dar continuidade ao projeto, é necessário configurar a interface da rede interna de cada servidor (gateway, samba, nameserver1, nameserver2). Para isso, deve-se editar o arquivo, localizado em  **/etc/netplan/.yaml**, com o comando:

```bash
sudo nano /etc/netplan/<name_file>.yaml
```

Desse modo, os IPs da interface ens192 devem ser substituídos conforme à Tabela 2. 

### Configuração do gateway na rede interna
O Gateway, com excessão do GRUPO4-VM1 (10.9.24.102 | 192.168.24.51), deve ser desativado na interface da rede externa (ens160), e ser ativado na interface da rede interna (ens192), sendo preenchido com o endereço IP (ens192) do GRUPO4-VM1, o qual desempenha a função de gateway para a rede interna. 

### Configuração dos clientes
   * Configure o dns no nas máquina ns1, ns2 e us adicionando os campos abaixo na interface de rede local deses servidores. Observe que na máquina gw essa configuração deve ser inserida na interface de rede interna (ens192)
```bash
nameservers: 
    addresses:
        - 10.9.24.114
        - 10.9.24.115
    search: [grupo4.turma924.ifalara.local]
```
---
![netplan_file_gateway](/Images/netplan_file_gateway.png)
---
![netplan_file_samba](/Images/netplan_file_samba.png)
---
![netplan_file_ns1](/Images/netplan_file_ns1.png)
---
![netplan_file_ns2](/Images/netplan_file_ns2.png)

</section>