# Projeto UOL Compass Wordpress EC2 (private) + Classic Load Balancer + Auto Scaling

![alt text](https://upload.wikimedia.org/wikipedia/commons/f/f3/LogoCompasso-positivo.png) 
 
![1_RgfvqJOB_oLtwLf8HW9spw](https://github.com/user-attachments/assets/d8346319-d9e8-4506-8bb8-58dbb6d263e8)

<br>
<br>
<br>

<p>Este guia apresenta a implantação do WordPress em EC2 com Docker/Containerd, integrando RDS (MySQL), EFS para armazenamento, e configurando Auto Scaling e Classic Load Balancer. O projeto foi solicitado pela <strong>Compass UOL</strong>, garantindo escalabilidade e alta disponibilidade para a aplicação.</p>

<br>
<br>
<br>

## **Técnologias utilizadas no laboratório**

- <img src="https://github.com/user-attachments/assets/4d1a8414-cdf7-437a-8385-b6b43cbe92ae" alt="AWS EC2" width="20"/> **AWS EC2** (Instância configurada com Ubuntu)
- <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/mysql/mysql-original-wordmark.svg" alt="MySQL" width="20"/> **AWS RDS** (Banco de dados MySQL)
- <img src="https://seeklogo.com/images/A/amazon-elastic-file-system-logo-E7053CDC9F-seeklogo.com.png" alt="AWS EFS" width="20"/> **AWS EFS** (Elastic File System)
- <img src="https://github.com/user-attachments/assets/4877e7d9-1167-47be-8527-f9c10427da9c" alt="AWS Load Balancer" width="20"/> **AWS Load Balancer**
- <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/docker/docker-original-wordmark.svg" alt="Docker" width="20"/> **Docker** ou <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/kubernetes/kubernetes-plain-wordmark.svg" alt="Containerd" width="20"/> **Containerd** (Dependendo da escolha)
- <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/bash/bash-original.svg" alt="Bash" width="20"/> **Script `user_data.sh`** (para automação da inicialização da instância EC2)
- <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/wordpress/wordpress-original.svg" alt="WordPress" width="20"/> **WordPress** (Aplicação que será implantada no container)

<br>
<br>

## **Desenho da arquitetetura para laboratório**

![68747470733a2f2f6d69726f2e6d656469756d2e636f6d2f76322f726573697a653a6669743a313035302f302a6f6339746838696c7575336d6d6338542e706e67](https://github.com/user-attachments/assets/f807b228-4285-4ee4-9375-6647f78aed46)

<br>
<br>

## **Aplicando Laboratório (Vamos iniciar?)**

<br>
<br>
<br>

### **1 - Acessar sua conta da AWS (Se não tiver conta, criar conta) :**

![Capturar](https://github.com/user-attachments/assets/1855f8bd-1386-4b2a-b68b-2b61dc051402)



<p>
    <a href="https://us-east-2.signin.aws.amazon.com/oauth?client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&code_challenge=j2zjHrNfWOzFRccugZMeGP7GVoV-z8vQ3VHeb6g0LM0&code_challenge_method=SHA-256&response_type=code&redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26nc2%3Dh_ct%26src%3Dheader-signin%26state%3DhashArgsFromTB_us-east-2_9c1e7907d450bff9" target="_blank" style="text-decoration: none; color: inherit;">
        <img src="https://cdn.icon-icons.com/icons2/2407/PNG/512/aws_icon_146074.png" alt="AWS EC2" width="40"/>
        Acessar sua conta clicando aqui
    </a>
</p>
<p>
    <a href="https://us-east-2.signin.aws.amazon.com/oauth?client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&code_challenge=j2zjHrNfWOzFRccugZMeGP7GVoV-z8vQ3VHeb6g0LM0&code_challenge_method=SHA-256&response_type=code&redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26nc2%3Dh_ct%26src%3Dheader-signin%26state%3DhashArgsFromTB_us-east-2_9c1e7907d450bff9" target="_blank" style="text-decoration: none; color: inherit;">
        <img src="https://cdn.icon-icons.com/icons2/2407/PNG/512/aws_icon_146074.png" alt="AWS EC2" width="40"/>
        Criar nova conta clicando aqui
    </a>
</p>


<br>
<br>
<br>


### **2 - No console da AWS depois de acessado vamos iniciar criando nossa VPC :**

<br>

<div style="border: 1px solid #ffa500; background-color: #fff3cd; padding: 10px; border-radius: 5px; margin-top: 10px;">
    ⚠️ <strong>Importante ao configurar a VPC:</strong> Altere apenas os parâmetros especificados abaixo. Para os demais, mantenha as configurações padrão.
</div>

<br>
<br>

![Capturar](https://github.com/user-attachments/assets/56888482-c7dd-427f-bb3e-1826e14a3238)

<br>

#### **➡️ Clicar em (VPC and More)**

<br>

![Capturar](https://github.com/user-attachments/assets/ef795485-aae6-4f0f-86eb-e8940b66cecb)

<br>

#### **➡️ Colocar o nome desejado que quiser**

<br>

![Capturar](https://github.com/user-attachments/assets/4411578b-d142-4909-81a4-0543c485623b)

<br>

#### **➡️ Deixar este CIDR Block**

<br>

![Capturar](https://github.com/user-attachments/assets/c3e1fc50-364d-4168-ae82-0b185192f0ff)

<br>

#### **➡️ Vamos aproveitar que estamos criando a VPC e vamos subir junto (1) um Gateway Nat**

<br>

![Capturar](https://github.com/user-attachments/assets/50b9e174-7533-48a6-9775-0aee0d97f086)


<br>
<br>

<div style="border: 1px solid #4CAF50; background-color: #e8f5e9; padding: 10px; border-radius: 5px; margin-top: 10px;">
    ✅ <strong>Agora com todos os passos informados:</strong> você pode criar sua VPC com segurança e conforme as orientações fornecidas.
</div>

<br>

<div style="border: 1px solid #4CAF50; background-color: #e8f5e9; padding: 10px; border-radius: 5px; margin-top: 10px;">
    ✅ <strong>Agora pode seguir o próximo passo a passo.
</div>


<br>
<br>

<hr>

<br>
<br>

### **3 - Agora que criamos e finalizamos a VPC com sucesso agora precisamos criar nossos security groups :**

<br>

#### **➡️ Vamos acessar novamente o serviço da VPC e do lado esquerdo tem a opção security groups**

<br>

![Capturar](https://github.com/user-attachments/assets/38485065-5915-4f48-abcc-833e44aeffa1)

<br>

#### **➡️ Agora vamos clicar para criar novos securitys groups, vamos criar duas, uma para usarmos tudo o que for privado e um security group para tudo publico**

<br>

![Capturar](https://github.com/user-attachments/assets/95d22609-cd7b-4d34-9b3d-b95c4d0b8e1d)

<br>
<br>

<div style="border: 1px solid #ffa500; background-color: #fff3cd; padding: 10px; border-radius: 5px; margin-top: 10px;">
    ⚠️ <strong>Importante ao configurar o Security Group:</strong> Altere apenas os parâmetros especificados abaixo. Para os demais, mantenha as configurações padrão.
</div>

<br>
<br>

<div style="border: 1px solid #ffa500; background-color: #fff3cd; padding: 10px; border-radius: 5px; margin-top: 10px;">
    ⚠️ <strong>Nesse passo VOCÊ deve escolher a VPC que você criou no passo anterior.
</div>

<br>
<br>

#### **➡️ Abaixo vamos criar (2x) duas vezes, primeiro vamos criar o security group (PRIVADO)**

<br>

![Capturar](https://github.com/user-attachments/assets/cb20c45d-9da9-4eb9-a91e-ce1a4fa99d9b)

<br>

![Capturar](https://github.com/user-attachments/assets/0f357787-a471-4568-a5c0-e65ba7aa190e)

<br>
<br>

#### **➡️ Agora vamos criar o segundo security group (PUBLICO 🌍)**

<br>

![Capturar](https://github.com/user-attachments/assets/bdf6e973-9d1f-4286-8d84-9a0ca43a3570)





<br>
<br>
<br>
<br>



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

Agora o **WordPress** está configurado com um Load Balancer, garantindo alta disponibilidade e distribuindo o tráfego de forma eficiente entre as instâncias EC2. 🚀
![loadbalencer](https://github.com/user-attachments/assets/1276c919-24dd-423f-83ed-dd48cdad30cd)



---

