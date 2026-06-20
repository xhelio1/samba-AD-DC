#  Debian 13 : File Server Samba

***Informações do Servidor***  

Nome da máquina : arquivos  
IP da máquina   : 192.168.1.133  
Domínio         : xhelionet.com

* ajuste o IP e o gateway de acordo com sua rede, essas são minhas configurações, as suas certamente serão diferentes.

***1-*** Alterando o Hostname

Edite o arquivo:

```bash
hostnamectl set-hostname arquivos
```

Para verificar:

```bash
cat /etc/hostname
```


Saída esperada:

```text
arquivos
```

***2-*** Configurando IP Estático

Edite:

```bash
nano /etc/network/interfaces   
```

Configuração padrão

```bash
# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp
# This is an autoconfigured IPv6 interface
iface enp0s3 inet6 auto
```
Configuração após alteração (IP Estático)

```bash
# The primary network interface
#allow-hotplug enp0s3
auto enp0s3
iface enp0s3 inet static
        address 192.168.1.133
        netmask 255.255.255.0
        gateway 192.168.1.254
```

Após finalizar:

```bash
reboot
```
Vamos ver se as configurações foram aplicada: 

```bash
ip a
```
procure a linha :

```bash
 inet 192.168.1.133/24 brd 192.168.1.255 scope global enp0s3
```

Note que ja está com o IP 192.168.1.133 que definimos.


***3-*** Configurando o Arquivo Hosts

Edite:

```bash
nano /etc/hosts
```

Altere para :

```bash
127.0.0.1       localhost
192.168.1.133   arquivos.xhelionet.com  arquivos
```

***4-*** Configurando o resolv.conf

Como utilizaremos o DNS interno do Samba, devemos configurar corretamente:

```bash
nano /etc/resolv.conf
```
ficando assim :

```bash
domain xhelionet.com
search xhelionet.com
nameserver 192.168.1.121
```

Tornando o arquivo imutável  
Por padrão, o resolv.conf pode ser alterado automaticamente pelo sistema.

Para impedir alterações:

```bash
chattr +i /etc/resolv.conf
```

Para permitir edição novamente QUANDO PRECISAR:

```bash
chattr -i /etc/resolv.conf
```

***5-*** Atualizando o Sistema

```bash
apt update
apt upgrade -y
```
depois 

```bash
reboot
```

***6-*** Instalando o Samba e dependências

```bash
apt install samba winbind libnss-winbind libpam-winbind smbclient cifs-utils acl attr krb5-user dnsutils
```

Durante a instalação do krb5-user, pode aparecer uma tela pedindo:

```bash
Default Kerberos version 5 realm
```
Digite:

```bash
XHELIONET.COM
```
O realm deve estar em MAIÚSCULO.

os proximos pode colocar:

```bash
dc01
```

***7-*** Configurando o Kerberos

```bash
nano /etc/krb5.conf
```

deve ficar assim

```bash
[libdefaults]
        dns_lookup_realm = false
        dns_lookup_kdc = true
        default_realm = XHELIONET.COM
```

Vamos validar o mapeamento de nome

```bash
getent hosts arquivos
```

deve retornar

```bash
192.168.1.133   arquivos.xhelionet.com arquivos
```

***8-*** Fazendo Backup do smb.conf

```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.bkp
```

***9-*** Configurando o smb.conf

```bash
nano /etc/samba/smb.conf
```

na opção global são essas configurações

```bash
[global]

## Browsing/Identification ###

# Change this to the workgroup/NT-domain name your Samba server will part of
   workgroup = XHELIONET
   realm = XHELIONET.COM
   security = ADS
   username map = /etc/samba/user.map
   map acl inherit = yes
   store dos attributes = yes
   vfs objects = acl_xattr acl_tdb
   dedicated keytab file = /etc/krb5.keytab
   kerberos method = secrets and keytab
   idmap config * : backend = tdb
   idmap config * : range = 3000-7999
   idmap config XHELIONET: backend = rid
   idmap config XHELIONET: range = 10000-999999
   min domain uid = 0
   template shell = /bin/bash
   template homedir = /home/%U
   winbind refresh tickets = yes
   winbind use default domain = yes
   winbind enum users = yes
   winbind enum groups = yes
   winbind cache time = 7200
   winbind nss info = rfc2307
   sync always = yes
   strict sync = yes
   log file = /var/log/samba/log.%m
   log level = 3
```

***No final do arquivo coloque***

```bash
[arquivos]
    path = /srv/arquivos
    comment = Compartilhamentos da Rede
    read only = No
    browseable = yes
    writable = yes
    guest ok = no
    create mask = 0660
    directory mask = 0770
    vfs objects = acl_xattr acl_tdb full_audit
    map acl inherit = yes
    store dos attributes = yes
    full_audit:success = renameat rewinddir unlinkat
    full_audit:prefix = %U|%I|%S
    full_audit:failure = none
    full_audit:facility = local4
    full_audit:priority = alert
```

validando as configurações

```bash
testparm
```

saida importante:

```bash
Loaded services file OK.
Server role: ROLE_DOMAIN_MEMBER
```

Isso indica que o Samba aceitou sua configuração

***10-*** Criar o arquivo user.map

```bash
nano /etc/samba/user.map
```

adicione:

```bash
root = XHELIONET\Administrator
```

Altere as permissões do arquivo user.map

```bash
chown root:root /etc/samba/user.map
chmod 600 /etc/samba/user.map
```

***11-*** Vamos validar a troca de tickets do kerberos

```bash
kinit Administrator
```

deve aparecer algo como :

```bash
Password for Administrator@XHELIONET.COM:
Warning: Your password will expire in 5 days on qui 25 jun 2026 10:05:38
```

vamos ver o ticket

```bash
klist
```

vai mostrar algo como :

```bash
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: Administrator@XHELIONET.COM

Valid starting       Expires              Service principal
19/06/2026 11:20:51  19/06/2026 21:20:51  krbtgt/XHELIONET.COM@XHELIONET.COM
        renew until 20/06/2026 11:20:45
```

***12-*** Vamos criar o diretório de compartilhamento

```bash
mkdir -p /srv/arquivos
```

***13-*** Configurar as permissões

```bash
chmod -R 0770 /srv/arquivos
```

***14-*** Vamos ingressar no domínio

```bash
net ads join -U Administrator
```

Confira se ingressou

```bash
net ads testjoin
```

Deve retornar :

```bash
Join is OK
```

Ativando os serviços:

```bash
systemctl enable smbd
systemctl enable nmbd
systemctl enable winbind
```

***15-*** Agora vamos reiniciar o computador

```bash
reboot
```

***16-*** Validando usuarios e grupos do AD

```bash
wbinfo -u 
```

vai mostrar algo como :

```bash
elias
guest
administrator
krbtgt
```

validando os grupos

```bash
wbinfo -g
```

vai mostrar algo como :


```bash
dnsadmins
domain controllers
dnsupdateproxy
domain users
enterprise admins
cert publishers
read-only domain controllers
domain guests
allowed rodc password replication group
enterprise read-only domain controllers
group policy creator owners
domain computers
ras and ias servers
domain admins
protected users
schema admins
denied rodc password replication group
```

validar o Domain Users

```bash
getent group "Domain Users"
```
vai retornar algo como :

```bash
domain users:x:10513:
```

***17-*** Vamos mudar o grupo da pasta de compartilhamento


```bash
chown -R root:"Domain Users" /srv/arquivos
```



























