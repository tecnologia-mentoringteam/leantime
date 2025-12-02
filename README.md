# Documentação de Instalação do Leantime

## 1. Visão Geral
Este documento orienta como instalar e configurar o **Leantime**, tanto localmente quanto em provedores de nuvem (Hostinger/VPS, **Validado** ). A abordagem utiliza **Docker** e **Traefik** com **Let's Encrypt**, baseada no repositório:

- **Repositório do Mentoring Team - Tecnologia:** https://github.com/tecnologia-mentoringteam/leantime
- **Leantime oficial:** https://leantime.io/
- **Repositório oficial:** https://github.com/leantime/leantime
- **Docker da Leantime:** https://github.com/Leantime/docker-leantime

O conteúdo é organizado para que um usuário com conhecimentos prévio em DevOps ou infraestrutura consiga executar a instalação por conta própria.

---

## 2. Pré-requisitos
- Docker
- Docker Compose
- Git instalado
- Porta 80, 443, 3306 livres (Localmente aplicação usa a porta 8080)

O processo é semelhante para as diversas nuvens(cloud).

---

## 3. Estrutura do Projeto
A estrutura base utilizada no repositório é:

```
leantime/
├── docker-compose.yml
├── .env (variáveis de ambiente do projeto Leantime)
├── .env_db (variáveis de ambiente do banco de dados)
├── traefik-docker-compose.yml
├── traefik/
│   ├── traefik.yml
│   ├── acme.json
│   ├── log/
│   ├── dynamic/
│   │   └── leantime.yml
│   │   └── dash-traefik.yml
└── README.md
```

Essa estrutura permite modularidade e fácil replicação.

---

## 4. Clonando o Repositório
```bash
git clone https://github.com/tecnologia-mentoringteam/leantime.git
cd leantime
```

---

## 5. Configuração do Leantime
Edite o arquivo `.env` conforme necessário:

```
LEAN_APP_URL='https://seudominio.com'
LEAN_DB_HOST='leantime_db'
LEAN_DB_USER='lean'
LEAN_DB_PASSWORD='lean'
LEAN_DB_DATABASE='leantime'
LEAN_DB_PORT='3306'
LEAN_SESSION_PASSWORD='hash_forte'
```

Edite o arquivo `.env_db` conforme necessário:
```
MYSQL_ROOT_PASSWORD = 'root'
MYSQL_DATABASE = 'leantime'
MYSQL_USER = 'lean'
MYSQL_PASSWORD = 'lean'
```
Em produção desative o acesso via porta 8080 no arquivo docker-compose.yml

---

## 6. Subindo os Contêineres da aplicação Leantime.
```bash
docker compose -f docker-compose.yml up -d
```

Verifique logs:
```bash
docker compose logs -f
```


---
## 7. Configuração do Traefik, obrigatório em produção.
O Traefik funciona como proxy reverso para o Leantime e gera certificados Let's Encrypt automaticamente.

### 7.1 Arquivo traefik.yml
Aplicação traefik
### 7.2 Arquivo traefik/*, arquivos de configuração do traefik.
Inclui:
- Habilitação de dashboard(Desabilitado por padrão)
- Configuração da entrypoint http/https
- CertResolver Let's Encrypt
### 7.2.1 Altere o arquivo traefik/dynamic/leantime.yml
Altere as ocorrências de **DOMAIN.local** para seu domínio ou IP do servidor

### 7.2.1 Altere o arquivo traefik/dynamic/dash-traefik.yml para o dashboard do traefik.
Altere as ocorrências de **DOMAIN.local** para seu domínio ou IP do servidor.
**OBS:** Deixe o subdomínio **traefik**.

### 7.2.2 Adicione uma camada de segurança para o dashboard do traefik com basiAuth.
```bash
docker run --rm httpd:2-alpine htpasswd -nbB seu_usuario 'sua_senha' > dashboard_htpasswd.txt
```
### 7.2.3 Atualize middlewares da rota do traefik.
Substitua a linha abaixo
- "admin:$2y$05$EXEMPLODeHASHGeradoPeloHtpasswd"
Pelo conteudo hash do arquivo dashboard_htpasswd.txt respeitando a sintaxe e lógica.

### 7.3 Permissões para acme.json, execute o comando abaixo.
```bash
chmod 600 traefik/acme.json
```
### 7.4 Inicie o traefik
```bash
docker compose -f traefik-docker-compose.yml up -d
```

### 7.4 Verifique os logs do traefik
```bash
docker logs -f traefik
```
### 7.4 Dashboard do traefik
```
https://traefik.seudominio.com
```

---
## 8. Acessando o Sistema
## 8.1 Acessando o sistema local
Abra o navegador e acesse:
```
http://localhost:8080
```

## 8.2 Acessando o sistema em produção
Abra o navegador e acesse:
```
https://seudominio.com
```
---

# Helpers
## Backup e Restore
### Backup do banco
```bash
docker exec leantime-db mysqldump -u leantime -p leantime > backup.sql
```

### Restore
```bash
docker exec -i leantime-db mysql -u leantime -p leantime < backup.sql
```

---

## Possíveis Problemas e Soluções
### Certificado Let's Encrypt não gerado
- Porta 80 bloqueada
- DNS incorreto
- Segurança da VPS aplicando regras adicionais

### Erro 502 no Traefik
- Serviço Leantime não iniciou
- Variáveis de ambiente incorretas

### Problemas de permissão
```bash
chmod 600 traefik/acme.json
```

---

## Conclusão
Este guia fornece o passo a passo básico completo para instalar o Leantime usando Docker e Traefik, seja localmente ou em uma nuvem. Baseado no repositório configurado, qualquer profissional com conhecimento intermediário pode replicar o ambiente.
