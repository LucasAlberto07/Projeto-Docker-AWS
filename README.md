---

# **Deploy de Aplicação WordPress com Docker/Containerd, RDS, EFS e Load Balancer**

Este repositório contém os passos necessários para configurar uma aplicação WordPress em uma instância EC2 utilizando Docker/Containerd, integrando com o RDS (MySQL), EFS para armazenamento estático e configurando um Load Balancer para balanceamento de carga.

## **Requisitos**

- **AWS EC2** (Instância configurada com Ubuntu) etapa atual ⏲️
- **AWS RDS** (Banco de dados MySQL) ❌
- **AWS EFS** (Elastic File System)  ❌
- **AWS Load Balancer**
- **Docker** ou **Containerd** (Dependendo da escolha)
- **Git** (para versionamento de código) 
- **Script `user_data.sh`** (para automação da inicialização da instância EC2) ❌
- **WordPress** (Aplicação que será implantada no container) ❌

## **Estrutura do Projeto**

```
.
├── docker-compose.yml       # Arquivo de configuração do Docker Compose
├── Dockerfile               # Arquivo de configuração do Docker (se necessário)
├── user_data.sh            # Script de inicialização da instância EC2 (user data)
├── README.md               # Documentação do projeto
└── wordpress/               # Código do WordPress ou repositório do WordPress
```

## **Passo a Passo para Configuração**

### 1. **Instalar e Configurar o Docker/Containerd na Instância EC2**

Você pode usar o script `user_data.sh` para automatizar a instalação do Docker ou Containerd ao iniciar a instância EC2.

#### Exemplo de script `user_data.sh`:

```bash
#!/bin/bash
# Atualizar pacotes
apt-get update -y

# Instalar pacotes necessários
apt-get install -y apt-transport-https ca-certificates curl software-properties-common

# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Adicionar usuário atual ao grupo docker
usermod -aG docker ubuntu

# Iniciar o Docker
systemctl start docker
systemctl enable docker
```

### 2. **Configurar o Banco de Dados MySQL no AWS RDS**

1. Acesse o console do AWS RDS e crie uma nova instância de banco de dados MySQL.
2. Defina as credenciais de acesso (usuário, senha, etc.).
3. Crie uma VPC e uma sub-rede que o RDS estará utilizando. A instância EC2 e o RDS precisam estar na mesma VPC.
4. Conecte-se ao banco de dados do WordPress utilizando o endpoint fornecido pelo RDS.

### 3. **Configurar o EFS (Elastic File System)**

1. Crie um EFS no console da AWS.
2. Monte o EFS nas instâncias EC2. Você pode montar o EFS diretamente no caminho `/var/www/html` (onde o WordPress armazena os arquivos estáticos).

Para montar o EFS:

```bash
# Instalar cliente NFS
apt-get install -y nfs-common

# Criar diretório para montagem
mkdir -p /mnt/efs

# Montar o EFS (substitua pelo ID do seu EFS)
mount -t nfs4 -o nfsvers=4.1 <EFS-DNS-NAME>:/ /mnt/efs

# Montar automaticamente na inicialização
echo "<EFS-DNS-NAME>:/ /mnt/efs nfs4 defaults,_netdev 0 0" >> /etc/fstab
```

### 4. **Configurar o Docker para WordPress com MySQL**

Crie um arquivo `docker-compose.yml` ou `Dockerfile` para configurar o WordPress com o MySQL.

#### Exemplo `docker-compose.yml`:

```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: <RDS-ENDPOINT>
      WORDPRESS_DB_NAME: <DB-NAME>
      WORDPRESS_DB_USER: <DB-USER>
      WORDPRESS_DB_PASSWORD: <DB-PASSWORD>
    volumes:
      - /mnt/efs:/var/www/html  # Montando EFS no WordPress para persistência de arquivos
    networks:
      - wordpress-network

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: <DB-PASSWORD>
    networks:
      - wordpress-network

networks:
  wordpress-network:
    driver: bridge
```

**NOTA**: Substitua `<RDS-ENDPOINT>`, `<DB-NAME>`, `<DB-USER>`, e `<DB-PASSWORD>` pelos valores do seu banco de dados RDS.

### 5. **Configurar o Load Balancer AWS**

1. Crie um Load Balancer Classic ou ALB (Application Load Balancer) na AWS.
2. Configure o Load Balancer para rotear o tráfego HTTP (porta 80) para as instâncias EC2 onde o WordPress está sendo executado.
3. Configure a segurança para que o tráfego da internet chegue ao Load Balancer e seja redirecionado para as instâncias privadas do WordPress.

### 6. **Deploy e Teste da Aplicação**

1. Após concluir a configuração do Docker/Containerd, RDS, EFS e Load Balancer, execute o comando `docker-compose up` para iniciar o WordPress.
2. Abra o navegador e acesse o Load Balancer (não use IP público) para verificar se a aplicação WordPress está funcionando corretamente.
3. O WordPress deve ser acessado na porta 80 ou 8080 via Load Balancer. A tela de login do WordPress aparecerá.


### 7. **Documentação**

Crie um arquivo README.md detalhado, como este, para documentar todos os passos e configurações realizadas.

---

### **Conclusão**

Este guia fornece os passos para configurar e implantar uma aplicação WordPress com Docker, RDS, EFS e Load Balancer na AWS. Certifique-se de adaptar as configurações conforme a necessidade do seu ambiente de produção.

---

