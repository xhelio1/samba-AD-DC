# Ingressando um cliente debian 13 no AD

Mude  XHELIONET.COM pelo seu dominio

***1-*** Configurando o Arquivo Hosts

Edite:

```bash
nano /etc/hosts
```

Altere para :

```bash
127.0.0.1   localhost
127.0.1.1   debian13.xhelionet.com  debian13
```

***2-*** Configurar o resolv.conf

```bash
nano /etc/resolv.conf
```

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

***3-*** Instalar Pacotes Necessários

```bash
apt update
apt install samba winbind libpam-winbind libnss-winbind krb5-user
```

***4-***  Configurar o krb5-user

```bash
nano /etc/krb5.conf
```

```bash
[libdefaults]
    default_realm = XHELIONET.COM
    dns_lookup_realm = false
    dns_lookup_kdc = true

[realms]
          XHELIONET.COM = {
                kdc = dc01.xhelionet.com
                admin_server = dc01.xhelionet.com
        }

[domain_realm]
         .xhelionet.com = XHELIONET.COM
         xhelionet.com = XHELIONET.COM
```

***5-*** Configurar o /etc/samba/smb.conf

```bash
nano /etc/samba/smb.conf
```

E configure da seguinte forma:

```bash
[global]
   workgroup = XHELIONET
   security = ads
   realm = XHELIONET.COM

   winbind use default domain = yes
   winbind enum users = yes
   winbind enum groups = yes
   winbind refresh tickets = yes
   winbind offline logon = yes

   idmap config * : backend = tdb
   idmap config * : range = 10000-19999

   idmap config XHELIO : backend = rid
   idmap config XHELIO : range = 20000-999999

   template shell = /bin/bash
   template homedir = /home/%U
```

Substitua XHELIONET e XHELIONET.COM conforme sua configuração real.

***6-*** Editar /etc/nsswitch.conf

```bash
nano /etc/nsswitch.conf
```

Altere as seguintes linhas para:

```bash
passwd:         compat winbind
group:          compat winbind
shadow:         compat
```

***6-*** ingressar no Dominio

```bash
net ads join -U Administrator
```
Você será solicitado a digitar a senha do usuário com permissão para adicionar máquinas ao domínio.

***7-*** Verificar se o Dns tem registro do Ad

```bash
nslookup dc01.xhelionet.com 192.168.1.121
```

Você vai ver algo como :

```bash
Server:         192.168.1.121
Address:        192.168.1.121#53

Name:   dc01.xhelionet.com
Address: 192.168.1.121
Name:   dc01.xhelionet.com
Address: 2804:21f4:8180:1044:a00:27ff:fe25:95a1
```

Caso não tenha rode

```bash
net ads dns register -U Administrator
```

***8-*** Verificar Ingresso

```bash
net ads testjoin
```

Deve retornar:

```bash
Join is OK
```

Ative e reinicie os serviços :

```bash
systemctl enable --now smbd nmbd winbind
systemctl restart smbd nmbd winbind
```

Verifique usuários e grupos do domínio:

```bash
wbinfo -u
wbinfo -g
```

Verifique ID de usuário:

```bash
getent passwd usuario
```



***8-*** Habilitar Login de Usuários do Domínio

Edite PAM para criar home automaticamente:

```bash
nano /etc/pam.d/common-session
```

Adicione ao final:

```bash
session required pam_mkhomedir.so skel=/etc/skel umask=0077
```

***9-*** Reinicie Serviços

```bash
systemctl restart smbd nmbd winbind
```

Para garantir que winbind suba com o sistema:

```bash
systemctl enable winbind
```
***-*** Sincronize o Relógio ***COMO ESTAMOS USANDO VM NÃO PRECISA CONFIGURAR***

O Kerberos exige sincronização de tempo:

```bash
sudo timedatectl set-ntp true
```

Ou configure o NTP apontando para o DC.

 Teste o Login

 ***VAMOS CONFIGURAR LOGIN VIA INTERFACE GRAFICA***
 
 Configurar LightDM para aceitar usuários do domínio

***10-*** Editar ou criar o arquivo de configuração do LightDM

```bash
nano /etc/lightdm/lightdm.conf
```

Edite ou adicione as linhas abaixo :

```bash
[Seat:*]
greeter-show-manual-login=true
greeter-hide-users=true
allow-guest=false
```

***Explicações:***

greeter-show-manual-login=true
Mostra a opção para digitar o nome de usuário na tela de login.

greeter-hide-users=true
Não exibe a lista de usuários na tela de login. (Bom para ambiente corporativo)

allow-guest=false
Não permite entrar no sistema como convidado.


***11-*** Certifique-se de que PAM está permitindo usuários do domínio

Se você usou SSSD ou Winbind, o PAM já deve estar integrado corretamente. Mas valide que o módulo home esteja presente:

```bash
nano /etc/pam.d/common-session
```

Veja se tem essa linha :

```bash
session required pam_mkhomedir.so skel=/etc/skel umask=0077
```

***12-*** Reiniciar o LightDM

```bash
systemctl restart lightdm
```





