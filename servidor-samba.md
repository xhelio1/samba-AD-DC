#  Debian 13 : Instalação via Pacotes Binários


Vamos configurar o Debian 13 como controlador de dominio 

### Pre-requisitos

1. Ter o debian 13 instalado na sua maquina ou no seu virtual box se você for utilizar VM

    **link para baixar o Debian**  https://www.debian.org/download

    * Se voce for utilizar o virtual box para testar o funcionamento. recomendo essas configurações

        * Armazenamento 6-10 GB
        * 1024 de memoria RAM
        * **IMPORTANTE :** Nas configurações da VM vá em rede e coloque o adptador em modo Bridge para receber o ip na sua faixa de rede e conseguir enxergar outros dispositivos 


***Informações do Servidor***  

Nome da máquina : dc01  
IP da máquina   : 192.168.1.121  
Domínio         : xhelionet.com

* ajuste o IP e o gateway de acordo com sua rede, essas são minhas configurações, as suas certamente serão diferentes.

***1-*** Alterando o Hostname

Edite o arquivo:

```bash
hostnamectl set-hostname dc01
```

Para verificar:

```bash
cat /etc/hostname
```


Saída esperada:

```text
dc01
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
        address 192.168.1.121
        netmask 255.255.255.0
        gateway 192.168.1.254
```

O que foi alterado?

auto enp0s3
  Ativa a interface automaticamente no boot.

Mudança de dhcp para static
  Agora usamos IP fixo.

address 192.168.1.121
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
 inet 192.168.1.121/24 brd 192.168.1.255 scope global enp0s3
```

Note que ja está com o IP 192.168.1.121 que definimos.

***3-*** Configurando o Arquivo Hosts

Edite:

```bash
nano /etc/hosts
```

Altere para :

```bash
127.0.0.1       localhost
192.168.1.121   dc01.xhelionet.com  dc01
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

***6-*** Instalando o Samba e dependências

```bash
apt install samba smbclient winbind krb5-user dnsutils -y
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

***7-*** Parando serviços antigos

```bash
systemctl stop smbd nmbd winbind
systemctl disable smbd nmbd winbind
systemctl mask smbd nmbd winbind
```

***8-*** Fazendo Backup do smb.conf

```bash
mv /etc/samba/smb.conf /etc/samba/smb.conf.bkp
```

***9-*** Executando o Provisionamento
```bash
samba-tool domain provision --use-rfc2307 --interactive
```
```text
Server Role (dc, member, standalone) [dc]: dc 
(tipo usado pelo Active Directory)DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]:
SAMBA_INTERNAL (confirma uso do DNS interno)DNS forwarder IP address (write 'none' to disable forwarding) [192.168.1.131]: 8.8.8.8 
(DNS usado para consultas externas)Administrator password: 
#senha do Administrador do Active Directory (deve ser forte, com no mínimo 7 caracteres)
```
***10-*** Configurando o Kerberos

```bash
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```
***11-*** Iniciando o serviço do Samba AD DC

```bash
systemctl unmask samba-ad-dc
systemctl enable samba-ad-dc
systemctl start samba-ad-dc
```

Verifique:

```bash
systemctl status samba-ad-dc
```
deve aparecer : 

```text
active (running)
```

***12-*** Testando o AD

Testar DNS

```bash
host -t A dc01.xhelionet.com
```

Testar Kerberos

```bash
kinit administrator
```
Digite a senha.  
Se tudo der certo vai aparecer assim : 

```text
root@dc01:~# kinit administrator
Password for administrator@HELIO.NET:
Warning: Your password will expire in 41 days on dom 12 abr 2026 11:50:46
root@dc01:~#
```

Teste de conexão:

```bash
smbclient -L //localhost -U Administrator
```

Digite senha do Administrator  

E logo em seguida vai mostrar algo como : 

```bash
Password for [XHELIONET\Administrator]:

        Sharename       Type      Comment
        ---------       ----      -------
        sysvol          Disk
        netlogon        Disk
        IPC$            IPC       IPC Service (Samba 4.22.6-Debian-4.22.6+dfsg-0+deb13u1)
SMB1 disabled -- no workgroup available
```

***IMPORTANTE:*** Em ambientes com Samba AD reais é necessário que o relógio do servidor e dos clientes esteja sincronizado. 
Neste guia não abordarei essa configuração, pois é simples e existem diversos tutoriais disponíveis na internet.






















