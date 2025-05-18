
# Guia Completo de Instalação do n8n com PostgreSQL em Docker

## 📦 Pré-requisitos
- Sistema operacional: Ubuntu/Debian ou RHEL/Oracle Linux 8+
- 2 GB de RAM mínimo (4 GB recomendado)
- 20 GB de espaço em disco
- Acesso root/sudo

## 1. Instalação do Docker Engine

### Para sistemas Ubuntu/Debian:
```bash
# Remover versões antigas
sudo apt remove docker docker-engine docker.io containerd runc

# Instalar dependências
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg software-properties-common

# Adicionar chave GPG oficial
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Adicionar repositório
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Configurar usuário
sudo usermod -aG docker $USER
newgrp docker
```

### Para RHEL/Oracle Linux 8+:
```bash
# Instalar utilitários
sudo dnf install -y yum-utils

# Adicionar repositório
sudo yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

# Instalar Docker
sudo dnf install -y docker-ce docker-ce-cli containerd.io

# Iniciar serviço
sudo systemctl enable --now docker

# Configurar usuário
sudo usermod -aG docker $USER
newgrp docker
```

## 2. Configuração do PostgreSQL
```bash
# Criar rede Docker
docker network create n8n-network

# Iniciar container PostgreSQL
docker run -d   --name postgres   --network n8n-network   -e POSTGRES_USER=n8n   -e POSTGRES_PASSWORD=n8n   -e POSTGRES_DB=n8n   -v postgres_data:/var/lib/postgresql/data   -p 5432:5432   postgres:13
```

## 3. Instalação do n8n
```bash
# Criar e iniciar container n8n
docker run -d   --name n8n   --network n8n-network   -p 5678:5678   -e DB_TYPE=postgresdb   -e DB_POSTGRESDB_DATABASE=n8n   -e DB_POSTGRESDB_HOST=postgres   -e DB_POSTGRESDB_PORT=5432   -e DB_POSTGRESDB_USER=n8n   -e DB_POSTGRESDB_PASSWORD=n8n   -e DB_POSTGRESDB_SCHEMA=public   -e N8N_ENCRYPTION_KEY='S3nh@F0rt3!N8n_2023#A1b2C3d4E5f6G7h8'   -v n8n_data:/home/node/.n8n   docker.n8n.io/n8nio/n8n
```

## 4. Pós-instalação

### Verificar status:
```bash
docker ps -a
docker logs n8n
```

### Acessar interface:
Acesse no navegador: http://seu-ip:5678

### Comandos úteis:
```bash
# Parar serviços
docker stop n8n postgres

# Iniciar serviços
docker start postgres n8n

# Backup
docker run --rm -v n8n_data:/source -v $(pwd):/backup alpine tar czf /backup/n8n_backup_$(date +%Y%m%d).tar.gz -C /source .
docker run --rm -v postgres_data:/source -v $(pwd):/backup alpine tar czf /backup/postgres_backup_$(date +%Y%m%d).tar.gz -C /source .
```

## 🔒 Recomendações de Segurança
- Altere as credenciais padrão do PostgreSQL
- Configure um proxy reverso com HTTPS
- Atualize regularmente os containers
- Mantenha backups automáticos

## 📄 Documentação Oficial
- [n8n Docker](https://hub.docker.com/r/n8nio/n8n)
- [PostgreSQL Docker](https://hub.docker.com/_/postgres)
