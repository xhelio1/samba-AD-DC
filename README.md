# Debian 13 como controlador de dominio com samba4

![Debian](https://img.shields.io/badge/Debian-D70A53?style=for-the-badge&logo=debian&logoColor=white) ![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black) ![Samba4](https://img.shields.io/badge/Samba_4-DC3545?style=for-the-badge&logo=samba&logoColor=white)

## Objetivo

Um guia prático focado no provisionamento e administração de um controlador de domínio Samba4 no Debian 13, cobrindo desde a compilação do servidor até a integração de clientes e compartilhamento de arquivos em rede.

## Neste guia, você vai aprender:


* **Configuração Base do Debian 13:** Ajuste de hostname, IP estático e repositórios para um servidor.
* **Instalação e Provisionamento do Samba4:** Como transformar o Debian em um Active Directory Domain Controller (AD DC).
* **Gestão de DNS e Kerberos:** Configuração essencial para que o domínio seja localizado e a autenticação funcione corretamente.
* **Ingresso de Clientes Linux:** ingressar clientes linux com winbind e também configurar o login atráves da interface grafica
* **Servidor de Arquivos (File Server):** Criação de compartilhamentos com permissões baseadas em usuários e grupos do AD.



##  Sumário dos Guias

Clique nos links abaixo para acessar cada etapa do projeto:

*1.1*  **[Configuração do Servidor Samba AD DC -Binario](./servidor-samba-Binario.md)**
    * Preparação do Debian 13 e instalação de pacotes.

*1.2*  **[Configuração do Servidor Samba AD DC - Compilado](./servidor-samba-Compilado.md)**
    * Preparação do Debian 13 e compilação dos pacotes.

*2*  **[Configuração para Ingressar Debian 13 no AD ](./ingressar-cliente-debian.md)**
    * Configuração para ingressar no AD

*3*  **[Servidor de Arquivos (File Server) ](./file-server.md)**
    * Compartilhamento de arquivos em rede


## Referências

[![GitHub](https://img.shields.io/badge/Edu_Charquero-181717?style=flat&logo=github&logoColor=white)](https://github.com/educharquero)
[![YouTube](https://img.shields.io/badge/Void_Linux_BR-FF0000?style=flat&logo=youtube&logoColor=white)](https://www.youtube.com/@voidlinuxbr)
[![LinkedIn](https://img.shields.io/badge/Jo%C3%A3o_Almeida-0077B5?style=flat&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0id2hpdGUiPjxwYXRoIGQ9Ik0yMC40NDcgMjAuNDUyaC0zLjU1NHYtNS41NjljMC0xLjMyOC0uMDI3LTMuMDM3LTEuODUyLTMuMDM3LTEuODUzIDAtMi4xMzYgMS40NDUtMi4xMzYgMi45Mzl2NS42NjdIOS4zNTFWOWgzLjQxNHYxLjU2MWguMDQ2Yy40NzctLjkgMS42MzctMS44NSAzLjM3LTEuODUgMy42MDEgMCA0LjI2NyAyLjM3IDQuMjY3IDUuNDU1djYuMjg2ek01LjMzNyA3LjQzM2EyLjA2MiAyLjA2MiAwIDAxLTIuMDYzLTIuMDY1IDIuMDY0IDIuMDY0IDAgMTEyLjA2MyAyLjA2NXptMS43ODIgMTMuMDE5SDMuNTU1VjloMy41NjR2MTEuNDUyek0yMi4yMjUgMEgxLjc3MUMuNzkyIDAgMCAuNzc0IDAgMS43Mjl2MjAuNTQyQzAgMjMuMjI3Ljc5MiAyNCAxLjc3MSAyNGgyMC40NTFDMjMuMiAyNCAyNCAyMy4yMjcgMjQgMjIuMjcxVjEuNzI5QzI0IC43NzQgMjMuMiAwIDIyLjIyNSAweiIvPjwvc3ZnPg==&logoColor=white)](https://www.linkedin.com/in/joao-almeida-6087048/)
