#  Debian 13 : Compilado


Vamos configurar o Debian 13 como controlador de dominio 

### Pre-requisitos

1. Ter o debian 13 instalado na sua maquina ou no seu virtual box se você for utilizar VM

    **link para baixar o Debian**  https://www.debian.org/download

    * Se voce for utilizar o virtual box para testar o funcionamento. recomendo essas configurações

        * Armazenamento 6-10 GB
        * 1024 de memoria RAM
        * **IMPORTANTE :** Nas configurações da VM vá em rede e coloque o adptador em modo Bridge para receber o ip na sua faixa de rede e conseguir enxergar outros dispositivos

***Informações do Servidor***  

Nome da máquina : dc02  
IP da máquina   : 192.168.1.131  
Domínio         : empresa.com

* ajuste o IP e o gateway de acordo com sua rede, essas são minhas configurações, as suas certamente serão diferentes.

***1-*** Alterando o Hostname

Edite o arquivo:

```bash
hostnamectl set-hostname dc02
```

Para verificar:

```bash
cat /etc/hostname
```


Saída esperada:

```text
dc02
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
        address 192.168.1.131
        netmask 255.255.255.0
        gateway 192.168.1.254
```

O que foi alterado?

auto enp0s3
  Ativa a interface automaticamente no boot.

Mudança de dhcp para static
  Agora usamos IP fixo.

address 192.168.1.131
  Define o IP do servidor.

netmask 255.255.255.0
  Define a máscara de rede.

gateway 192.168.1.254
  Define o gateway para acesso à internet.

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
 inet 192.168.1.131/24 brd 192.168.1.255 scope global enp0s3
```

Note que ja está com o IP 192.168.1.131 que definimos.

Verificar se está como UP

```bash
ip -br link
```

***3-*** Configurando o Arquivo Hosts

Edite:

```bash
nano /etc/hosts
```

Altere para :

```bash
127.0.0.1       localhost
192.168.1.131   dc02.empresa.com  dc01
```

***4-*** Configurando o resolv.conf

Como utilizaremos o DNS interno do Samba, devemos configurar corretamente:

```bash
nano /etc/resolv.conf
```
ficando assim :

```bash
domain empresa.com
search empresa.com
nameserver 192.168.1.131
nameserver 8.8.8.8
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

baixando as dependencias do samba

```bash
export DEBIAN_FRONTEND=noninteractive;apt-get update; apt-get install vim net-tools rsync acl apt-utils attr autoconf bind9-utils binutils bison build-essential rsync ccache chrpath curl debhelper bind9-dnsutils docbook-xml docbook-xsl flex gcc gdb git glusterfs-common gzip heimdal-multidev hostname htop krb5-config krb5-user lcov libacl1-dev libarchive-dev libattr1-dev libavahi-common-dev libblkid-dev libbsd-dev libcap-dev libcephfs-dev libcups2-dev libdbus-1-dev libglib2.0-dev libgnutls28-dev libgpgme-dev libicu-dev libjansson-dev libjs-jquery libjson-perl libkrb5-dev libldap2-dev liblmdb-dev libncurses-dev libpam0g-dev libparse-yapp-perl libpcap-dev libpopt-dev libreadline-dev libsystemd-dev libtasn1-bin libtasn1-6-dev libunwind-dev lmdb-utils locales lsb-release make mawk mingw-w64 patch perl perl-modules-5.40 pkg-config procps psmisc python3 python3-cryptography python3-dbg python3-dev python3-dnspython python3-gpg python3-iso8601 python3-markdown python3-matplotlib python3-pexpect python3-pyasn1 rsync sed tar tree uuid-dev wget xfslibs-dev xsltproc zlib1g-dev -y
```

Agora vamos baixar e compilado o codigo fonte do samba4

```bash
wget https://download.samba.org/pub/samba/stable/samba-4.24.1.tar.gz
```

```bash
tar -xvzf samba-4.24.1.tar.gz
```

```bash
cd samba-4.24.1
```

```bash
./configure --prefix=/opt/samba --with-winbind --with-shared-modules=idmap_rid,idmap_ad
```

```bash
make -j$(nproc)
```

```bash
make install
```

```bash
make clean
```

#### Agora vamos verificar se o winbind e o NSS foram compilados

```bash
/opt/samba/sbin/smbd -b | grep WINBIND
```

Deve aparecer algo como 

```bash
WITH_WINBIND
```

```bash
/opt/samba/sbin/smbd -b | grep LIBDIR
```

#### Agora vamos verificar se o módulo NSS do Winbind foi compilado

```bash
find /opt/samba -name "libnss_winbind.so*"
```

Deve aparecer algo como:

```bash
/opt/samba/lib64/libnss_winbind.so.2
```

## Linkar as bibliotecas compiladas do Winbind e NSS ao path do Sistema Operacional

Recomendo executar os comandos manualmente por segurança (não copiar e colar)

```bash
ln -s /opt/samba/lib64/libnss_winbind.so.2 /lib/x86_64-linux-gnu/
```

```bash
ln -s /lib/x86_64-linux-gnu/libnss_winbind.so.2 /lib/x86_64-linux-gnu/libnss_winbind.so
```

```bash
ldconfig
```
Após executar o comando `ldconfig`, verifique se as bibliotecas foram reconhecidas corretamente pelo sistema:

```bash
ldconfig -p | grep winbind
```

vai ver algo como :

```bash
libnss_winbind.so (libc6,x86-64) => /lib/x86_64-linux-gnu/libnss_winbind.so
```

#### Adicionando /opt/Samba ao PATH padrão do Linux

Você pode utilizar a linha completa do PATH:

#### PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/samba/bin:/opt/samba/sbin"

ou apenas adicionar os diretórios do Samba ao PATH atual:

```bash
export PATH=$PATH:/opt/samba/bin:/opt/samba/sbin
```

Neste material será utilizada a segunda opção.

```bash
nano ~/.bashrc
```

Adicione no final do arquivo:

```bash
export PATH=$PATH:/opt/samba/bin:/opt/samba/sbin
```

#### Atualizando o PATH da sessão

```bash
source ~/.bashrc
```

#### Criando o serviço do Samba4 no systemd para inicialização automática com o sistema:

```bash
nano /etc/systemd/system/samba-ad-dc.service
```

```bash
[Unit]
Description=Samba 4 Active Directory Domain Controller
After=network.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/opt/samba/sbin/samba --foreground --no-process-group
Restart=always
LimitNOFILE=16384

[Install]
WantedBy=multi-user.target
```

#### Editando o nssswitch:

```bash
nano /etc/nsswitch.conf
```

```bash
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd winbind
group:          files systemd winbind
shadow:         files systemd
gshadow:        files systemd

hosts:          files dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

#### Provisionando o novo domínio suportado pelo dc02:

```bash
samba-tool domain provision --use-rfc2307 --interactive --option="interfaces=lo enp0s3" --option="bind interfaces only=yes"
```

#### Habilitando o daemon pra subir no boot do sistema:

```bash
systemctl daemon-reload
```

```bash
systemctl enable samba-ad-dc
```

```bash
systemctl start samba-ad-dc
```

```bash
systemctl status samba-ad-dc
```

#### Configurando o Kerberos

cp /opt/samba/private/krb5.conf /etc/krb5.conf

#### Validando resolvedor de nomes pelo dc02:

```bash
nslookup dc02.empresa.com
```

### Vai validar na tela:

```bash
Server:         192.168.1.131
Address:        192.168.1.131#53

Name:   dc02.empresa.com
Address: 192.168.1.131
Name:   dc02.empresa.com
Address: 2804:21f4:8180:2a83:a00:27ff:fe9e:de77
```

#### Reboot do servidor dc02:

```bash
reboot
```

#### Validando os serviços do samba4:

```bash
ps aux | grep samba
```

```bash
ps aux | egrep "samba|smbd|nmbd|winbind"
```

```bash
find / -name samba.pid
```

```bash
pgrep samba
```

#### Dando poderes de root ao Administrator

```bash
nano /opt/samba/etc/user.map
```

```bash
!root=empresa.com\Administrator
```

#### Relendo a configuração do Samba4:

```bash
smbcontrol all reload-config
```

#### Validando usuários da base do ldap local:

```bash
cat /etc/passwd | grep root
```

#### Validando usuários de rede do Samba4 (Intermediados pelo winbind):

```bash
samba-tool user show administrator
```

```bash
getent passwd administrator
```

```bash
wbinfo -u
```

```bash
wbinfo -g
```

```bash
wbinfo --ping-dc
```

```bash
getent group "Domain Admins"
```

#### Consultando serviços do SAMBA4:

```bash
smbclient --version
```


```bash
smbclient -L dc02 -U Administrator
```


```bash
Password for [EMPRESA\Administrator]:

        Sharename       Type      Comment
        ---------       ----      -------
        sysvol          Disk
        netlogon        Disk
        IPC$            IPC       IPC Service (Samba 4.24.1)
SMB1 disabled -- no workgroup available
```

```bash
smbclient //localhost/netlogon -UAdministrator -c "ls"
```

```bash
Password for [EMPRESA\Administrator]:
  .                                   D        0  Wed May 13 14:35:56 2026
  ..                                  D        0  Wed May 13 14:35:56 2026

                6025216 blocks of size 1024. 345696 blocks available
```

```bash
testparm
```

```bash
Load smb config files from /opt/samba/etc/smb.conf
Loaded services file OK.
Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

Server role: ROLE_ACTIVE_DIRECTORY_DC

Press enter to see a dump of your service definitions

# Global parameters
[global]
        bind interfaces only = Yes
        dns forwarder = 8.8.8.8
        interfaces = lo enp0s3
        passdb backend = samba_dsdb
        realm = EMPRESA.COM
        server role = active directory domain controller
        workgroup = EMPRESA
        rpc_server:tcpip = no
        rpc_daemon:spoolssd = embedded
        rpc_server:spoolss = embedded
        rpc_server:winreg = embedded
        rpc_server:ntsvcs = embedded
        rpc_server:eventlog = embedded
        rpc_server:srvsvc = embedded
        rpc_server:svcctl = embedded
        rpc_server:default = external
        winbindd:use external pipes = true
        idmap_ldb:use rfc2307 = yes
        idmap config * : backend = tdb
        map archive = No
        vfs objects = dfs_samba4 acl_xattr


[sysvol]
        path = /opt/samba/var/locks/sysvol
        read only = No


[netlogon]
        path = /opt/samba/var/locks/sysvol/empresa.com/scripts
        read only = No
```

```bash
samba-tool domain level show
```

```bash
Domain and forest function level for domain 'DC=empresa,DC=com'

Forest function level: (Windows) 2008 R2
Domain function level: (Windows) 2008 R2
Lowest function level of a DC: (Windows) 2008 R2
```

#### Validando a troca de tickets do Kerberos:

```bash
kinit Administrator@EMPRESA.COM
```

```bash
Warning: Your password will expire in 41 days on qua 24 jun 2026 14:36:02
```

```bash
klist
```

```bash
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: Administrator@EMPRESA.COM

Valid starting       Expires              Service principal
13/05/2026 15:36:03  14/05/2026 01:36:03  krbtgt/EMPRESA.COM@EMPRESA.COM
        renew until 14/05/2026 15:35:55
```

#### Consultando as bases do kerberos e ldap:

```bash
host -t srv _kerberos._tcp.empresa.com
```

Vai mostar algo como :

```
_kerberos._tcp.officinas.edu has SRV record 0 100 88 pdc01.officinas.edu.
```

```bash
host -t srv _ldap._tcp.empresa.com
```

Vai mostar algo como :

```bash
_ldap._tcp.empresa.com has SRV record 0 100 389 dc02.empresa.com.
```

















































