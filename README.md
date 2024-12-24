# AWS Site Wordpress 

<br>
 
![https://i.ytimg.com/vi/_nU4OrQ68Us/maxresdefault.jpg](https://i.ytimg.com/vi/_nU4OrQ68Us/maxresdefault.jpg)

<br>

Aqui será seu guia para implantar WordPress em EC2 utilizando Docker/Containerd, integrando RDS (MySQL), EFS para armazenamento estático, e configurando Auto Scaling e Load Balancer para balanceamento de carga.

![alt text](https://miro.medium.com/v2/resize:fit:1050/0*oc9th8iluu3mmc8T.png)


## **Requisitos**

- **AWS EC2** (Instância configurada com Ubuntu) 
- **AWS RDS** (Banco de dados MySQL) 
- **AWS EFS** (Elastic File System)  
- **AWS Load Balancer**
- **Docker** ou **Containerd** (Dependendo da escolha)
- **Script `user_data.sh`** (para automação da inicialização da instância EC2) 
- **WordPress** (Aplicação que será implantada no container) 




## **Estrutura do Projeto**

``` ![estrutura](https://github.com/user-attachments/assets/b526dedb-c2b7-44e7-9adb-7e6b11f5e19f)

.
├── docker-compose.yml       # Arquivo de configuração do Docker Compose
├── Dockerfile               # Arquivo de configuração do Docker (se necessário)
├── user_data.sh            # Script de inicialização da instância EC2 (user data)
├── README.md               # Documentação do projeto
└── wordpress/               # Código do WordPress ou repositório do WordPress
```
1º PASSO - CRIAÇÃO DA VPC 
- Criar VPC e suas Sub-Redes (2 Publicas e 2 Privadas)
  Instalar as sub-redes em Gateways

- Criar tabelas de rotas e associar as sub-redes

![vpc subnetes ](https://github.com/user-attachments/assets/aa3df51c-1ddd-4616-88a7-12c89e72a21c)

## Etapa 2: Configurando Regras de Acesso (Grupos de Segurança)
Grupo Público, entradas permitidas:

> HTTP (porta 80) de qualquer origem (0.0.0.0/0).
> 
> HTTPS (porta 443) de qualquer origem (0.0.0.0/0).
>
> SSH (porta 22) de qualquer origem (0.0.0.0/0).


Saídas permitidas:
>Todo o tráfego, sem restrição de portas ou protocolos.




Grupo Privado, entradas permitidas:
  
> MySQL (porta 3306) de qualquer origem.
>
> HTTP (porta 80) e HTTPS (porta 443) apenas do grupo público.
>
>  SSH (porta 22) de qualquer origem.
>
> NFS (porta 2049) de qualquer origem.

Saídas permitidas:
> Todo o tráfego liberado.

# 3º PASSO: Iniciar a Criação da RDS

## 1. Criar o Banco de Dados
1. No painel do RDS, clique em **"Create database"**.
2. Escolha as configurações de banco de dados:
   - **Engine options**: Escolha **MySQL** (ou outro banco disponível no Free Tier, como PostgreSQL).
   - **Templates**: Selecione **Free tier**.

---

## 2. Configurar a Instância do Banco
### **Settings**
- **DB instance identifier**: Nome da sua instância (ex.: `meubanco`).
- **Master username**: Escolha um nome de usuário (ex.: `admin`).
- **Master password**: Defina uma senha forte e confirme.

### **DB Instance Size**
- Escolha a classe de instância: **db.t2.micro** ou **db.t3.micro** (ambas gratuitas no Free Tier).

### **Storage**
- **Tipo de armazenamento**: General Purpose (SSD).
- **Tamanho de armazenamento**: **20 GB** (máximo permitido no Free Tier).

---

## 3. Configurações de Conectividade
### **Virtual Private Cloud (VPC)**
- Escolha a VPC padrão ou configure uma VPC específica.

### **Subnet Group**
- Use um grupo de sub-rede existente ou crie um novo.

### **Public Access**
- Marque **Yes** se deseja que o banco seja acessado publicamente (recomendado apenas para testes).

### **VPC Security Group**
- Configure ou selecione um **Security Group** para permitir acesso na porta **3306**.

---

## 4. Revisar e Criar
1. Revise todas as configurações realizadas.
2. Clique em **"Create database"**.
3. Aguarde a criação do banco de dados (o processo pode levar alguns minutos).

---

Agora, sua instância RDS está pronta e configurada para uso no Free Tier. 🚀


![RDS WORDPREES](https://github.com/user-attachments/assets/bf26fb8b-6008-45ca-aa90-cdaab136560e)

# 3. Configuração do Serviço EFS AWS para Arquivos Estáticos do WordPress

## 1. Criar o EFS
1. Acesse o painel **Amazon EFS** no console AWS.
2. Clique em **"Create file system"**.
3. Configure o sistema de arquivos:
   - **VPC**: Escolha a VPC configurada anteriormente.
   - **Subnets**: Selecione as sub-redes privadas.
   - **Security Group**: Associe o Security Group configurado para o EFS.
4. Finalize a criação clicando em **"Create"**.

---

## 2. Configurar o Ponto de Montagem
1. Após a criação, copie o **Endpoint DNS** do EFS.
2. Configure um ponto de acesso:
   - Crie o diretório `/wordpress`.
   - Configure permissões adequadas para leitura e gravação.

---

## 3. Montar o EFS na Instância EC2
1. Conecte-se à instância EC2 .
2. Monte o EFS:
   ```bash
   sudo mkdir -p /var/www/html
   sudo mount -t nfs4 <DNS_DO_EFS>:/ /var/www/html
![wordprees-efs](https://github.com/user-attachments/assets/67e1f530-6f64-4070-8182-cc6c8fae5750)


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
# 4. Configuração do Serviço de Load Balancer AWS para WordPress

## 1. Criar o Load Balancer
1. Acesse o painel **EC2** no console AWS.
2. No menu lateral, clique em **"Load Balancers"** e depois em **"Create Load Balancer"**.
3. Escolha o tipo **Classic Load Balancer**.

---

## 2. Configurar o Load Balancer
1. **Name**: Defina um nome (ex.: `wordpress-lb`).
2. **Scheme**: Escolha **"Internet-facing"** (voltado para a Internet).
3. **Network**:
   - **VPC**: Selecione a VPC configurada anteriormente.
   - **Subnets**: Escolha as sub-redes públicas.
4. **Listeners**:
   - Configure uma regra de escuta para HTTP na porta **80**.

---

## 3. Configurar os Grupos de Segurança
1. Associe o **Security Group** do Load Balancer.
   - **Entrada**: Permita HTTP (porta 80) de **0.0.0.0/0**.
   - **Saída**: Permita HTTP para o **Security Group da EC2**.

---

## 4. Configurar Verificações de Integridade
1. Defina o caminho de verificação:
   - Utilize um dos seguintes endpoints do WordPress:
     - `/wp-admin/install.php`
     - `/wp-login.php`
     - `/wp-admin/index.php`
2. Configure os tempos:
   - Intervalo: **30 segundos**.
   - Timeout: **5 segundos**.
   - Tentativas não aprovadas: **3**.
   - Tentativas aprovadas: **2**.

---

## 5. Registrar Instâncias
1. Adicione as instâncias **EC2** criadas anteriormente ao Load Balancer.
2. Confirme que as instâncias estão em estado **Healthy** após as verificações de integridade.

---

## 6. Obter o DNS do Load Balancer
1. Após a criação, copie o **DNS Name** do Load Balancer.
2. Use este DNS para acessar o WordPress:
   - Acesse: `http://<DNS_DO_LOAD_BALANCER>`

---


![loadbalencer](https://github.com/user-attachments/assets/1276c919-24dd-423f-83ed-dd48cdad30cd)

Agora o **WordPress** está configurado com um Load Balancer, garantindo alta disponibilidade e distribuindo o tráfego de forma eficiente entre as instâncias EC2. 🚀

---
# 5º PASSO - Criar Instância EC2 ☁️

## Introdução
Para criar a infraestrutura de aplicação WordPress, vamos iniciar com a criação de uma **Instância EC2** na AWS. Essa instância executará o WordPress e será configurada com o Docker e Docker Compose para facilitar o gerenciamento dos containers.

## Passo a Passo para Criar e Configurar a Instância EC2

### 1. **Abrir o painel da EC2**
- Acesse o painel da **EC2** no console da AWS.
- Clique em **Executar instância**.

### 2. **Configuração da Instância**

#### **Tags**
- Atribua as tags de identificação necessárias (exemplo: `Name = WordPress-Instance`).

#### **Tipo de Instância**
- Selecione **Free Tier** (t2.micro).

#### **AMI (Amazon Machine Image)**
- Escolha a **Amazon Linux 2023**.

#### **VPC**
- Use a **VPC** criada anteriormente.

#### **User Data**
- Insira o seguinte script no campo **User Data** para configurar a instância:

```bash
#!/bin/bash 

# Atualizando o sistema
sudo yum update -y 

# Instalando o Docker
sudo yum install docker -y 

# Iniciando o serviço Docker
sudo systemctl start docker

# Habilitando o Docker para iniciar automaticamente
sudo systemctl enable docker

# Adicionando o usuário ao grupo Docker
sudo usermod -aG docker ec2-user
newgrp docker

# Baixando e instalando o Docker Compose
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Criando diretório para o WordPress
sudo mkdir /home/ec2-user/wordpress

# Criando o arquivo docker-compose.yml
cat <<EOF > /home/ec2-user/wordpress/docker-compose.yml
services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: database-1.chc4e66surqu.us-east-1.rds.amazonaws.com:3306
      WORDPRESS_DB_USER: admin
      WORDPRESS_DB_PASSWORD: 12345678lucas
      WORDPRESS_DB_NAME: bancoaws
    volumes:
      - /mnt/efs:/var/www/html
EOF

# Montando o sistema de arquivos EFS
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-034132b3f8de6bd56.efs.us-east-1.amazonaws.com:/ /mnt/efs

# Inicializando o container WordPress
docker-compose -f /home/ec2-user/wordpress/docker-compose.yml up -d


