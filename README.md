# Projeto-Linux compass.uol-DevSecOps

## 1.1 Criar VPC
  1. Acesse o Console AWS
  2. Navegue até **VPC > Your VPCs > Create VPC**
- **Configurações**:
  - **Nome**: `ProjectLinux-VPC`
  - **IPv4 CIDR**: `10.0.0.0/16` 
  - **Por que /16?**: Para ter espaço para várias sub-redes

### 1.2 Criar Sub-redes Públicas
o	Acesse Subnets > Create Subnet
#### Configurações Public-Subnet-1:
| Parâmetro | Valor |
|-----------|-------|
| Nome | `Public-Subnet-1` |
| VPC | Selecionar a VPC criada |
| Availability Zone | `us-east-1a` |
| IPv4 CIDR | `10.0.0.0/20` |

#### Configurações Public-Subnet-2:
| Parâmetro | Valor |
|-----------|-------|
| Nome | `Public-Subnet-2` |
| VPC | Selecionar a VPC criada |
| Availability Zone | `us-east-1b` |
| IPv4 CIDR | `10.0.16.0/20` |

### 1.3 Criar Sub-redes Privadas
#### Private-Subnet-1:
- **Name tag**: `Private-Subnet-1`
- **Availability Zone**: `us-east-1a` (pode ser a mesma da primeira pública).
- **IPv4 CIDR block**: `10.0.128.0/20`

#### Private-Subnet-2:
- **Name tag**: `Private-Subnet-2`
- **Availability Zone**: `us-east-1b`
- **IPv4 CIDR block**: `10.0.144.0/20`

  
![Imagem1](https://github.com/user-attachments/assets/687a1850-8d2f-41f4-ac24-6e86ce440d20)


### 1.4 Internet Gateway (IGW)

 Caminho para criação:
VPC > Internet Gateways > Create Internet Gateway

- **Nome**: `ProjetoLinux-IGW`
- **Ação**: Clicar em "Attach to VPC"
- **Propósito**: Dar acesso à internet às sub-redes públicas

![Imagem2](https://github.com/user-attachments/assets/e5adb943-e45e-4891-ba80-237d1dd33cbd)


### 1.5 Tabela de Rotas

 Caminho para configuração:
Route Tables > Create Route Table

- **Adicionar rota**:
  - **Destination**: `0.0.0.0/0` (todo o tráfego)
  - **Target**: Selecionar o IGW criado

---

## 2. Criação da Instância EC2

### 2.0 criação das tags: crie essas tags marcadas:

![Imagem3](https://github.com/user-attachments/assets/5905eca9-f979-42e8-b91d-887478a20cd9)


### 2.1 AMI (Imagem da Máquina)
- Escolha **Ubuntu 22.04 LTS** (estável e bem documentada)

### 2.2 Tipo de Instância
- `t2.micro` (Free Tier, suficiente para testes)

### 2.3 Configuração de Rede
- **VPC**: `ProjetoLinux-VPC`
- **Subnet**: `Public-Subnet-1`
- **Auto-assign Public IP**: `Enable` (para acessar via navegador)

### 2.4 Security Group (Firewall)
- **Nome**: `Web-Server-SG`
- **Regras**:
  - **Porta 80 (HTTP)**: Liberada para `0.0.0.0/0` (acesso público)
  - **Porta 22 (SSH)**: Liberada apenas para seu IP (ex: `123.123.123.123/32`)

### 2.5 Key Pair
- Crie ou use um par de chaves existente (ex: `projeto-linux.pem`)
- **Importância**: Para autenticação segura via SSH

---

## 3. Instalação do Nginx

### 3.1 Atualizar pacotes
```bash
sudo apt update
```
*O que faz?* Sincroniza a lista de pacotes disponíveis.

### 3.2 Instalar Nginx
```bash
sudo apt install nginx -y
```
O `-y` confirma automaticamente a instalação.

### 3.3 Iniciar o serviço
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 3.4 Verificar status
```bash
sudo systemctl status nginx
```

![Imagem4](https://github.com/user-attachments/assets/00206eb1-5649-4aa1-bcc6-9d94a2a3d845)


---

## 4. Página HTML Customizada

### 4.1 Criar arquivo
```bash
sudo nano /var/www/html/index.html
```
Personalize a página HTML como preferir. Para acessar, use no navegador:  
`http://IP_PUBLICO_DA_EC2`

---

## 5. Monitoramento com Discord Webhook

### 5.1 Configurar Webhook no Discord
1. No seu servidor Discord:
2. **Configurações do Canal > Integrações > Webhooks > Novo Webhook**

![Imagem5](https://github.com/user-attachments/assets/cca85523-5cf5-4ad6-a3a2-41c92ed3946b)


### 5.2 Script de Monitoramento
```bash
#!/bin/bash
SITE="http://localhost" 
LOG="/var/log/monitoramento.log" 
WEBHOOK="URL_DO_SEU_WEBHOOK" # Substitua pela URL real

# Verifica se o site está online
if curl -s -I "$SITE" | grep -q "200 OK"; then
    echo "$(date) - ONLINE" >> "$LOG" # Registra sucesso
else
    echo "$(date) - OFFLINE" >> "$LOG" # Registra falha
    # Envia mensagem para o Discord
    curl -H "Content-Type: application/json" -d '{
        "username": "Bot Alerta",
        "embeds": [{
            "title": "Servidor Offline",
            "description": "O servidor está sem resposta!",
            "color": 16711680 
        }]
    }' "$WEBHOOK"
fi
```
### como funciona o script em um diagrama:
![Imagem6](https://github.com/user-attachments/assets/836fc271-d69e-4a07-ace8-b09e27ff0b68)


### 5.3 Permissões
```bash
sudo chmod +x /usr/local/bin/monitor_site.sh
```

### 5.4 Agendamento com Cron
**Editar crontab**:
```bash
sudo crontab -e
```
Adicione a linha:
```bash
* * * * * /usr/local/bin/monitor_site.sh >> /var/log/monitoramento.log 2>&1
```
- `* * * * *`: Executa a cada minuto
- `>> /var/log/monitoramento.log`: Redireciona a saída para o arquivo de log
- `2>&1`: Captura erros

---

## 6. Testes e Validação

### 6.1 Simular Falha
```bash
sudo systemctl stop nginx
```

### 6.2 Verificar Logs
```bash
tail -f /var/log/monitoramento.log
```
**Saída esperada**:
```
[Data-Hora] - OFFLINE
```

![Imagem7](https://github.com/user-attachments/assets/5ce76681-68ce-4f9d-a43e-7d3b9efb4bf1)

