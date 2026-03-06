# Debian 13 como controlador de dominio com samba4

![Debian](https://img.shields.io/badge/Debian-D70A53?style=for-the-badge&logo=debian&logoColor=white) ![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black) ![Samba4](https://img.shields.io/badge/Samba_4-DC3545?style=for-the-badge&logo=samba&logoColor=white)

## O que será abordado :

Este projeto guia você através da implementação completa de uma infraestrutura de rede baseada em software livre, utilizando o Debian 13 (Trixie) como base sólida. O foco principal é a substituição ou integração de soluções proprietárias por um ambiente Samba AD DC .

## Neste guia, você vai aprender:


* **Configuração Base do Debian 13:** Ajuste de hostname, IP estático e repositórios para um servidor.
* **Instalação e Provisionamento do Samba4:** Como transformar o Debian em um Active Directory Domain Controller (AD DC).
* **Gestão de DNS e Kerberos:** Configuração essencial para que o domínio seja localizado e a autenticação funcione corretamente.
* **Ingresso de Clientes Linux:** ingressar clientes linux mint e debian com winbind e também configurar o login atráves da interface grafica
* **Servidor de Arquivos (File Server):** Criação de compartilhamentos com permissões baseadas em usuários e grupos do AD.



##  Sumário dos Guias

Clique nos links abaixo para acessar cada etapa do projeto:

*  **[Configuração do Servidor Samba AD DC -Binario](./servidor-samba.md)**
    * Preparação do Debian 13 e instalação de pacotes.

*  **[Configuração do Servidor Samba AD DC - Compilado](./servidor-sambaComp.md)**
    * Preparação do Debian 13 e compilação dos pacotes