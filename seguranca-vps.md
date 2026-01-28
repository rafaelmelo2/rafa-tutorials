# Segurança Básica da VPS

Este tutorial cobre configurações básicas de segurança para uma VPS Linux (Ubuntu/Debian), incluindo firewall, SSH e boas práticas.

## 1. Configurar Firewall (UFW)

O UFW (Uncomplicated Firewall) é uma interface simples para o iptables.

### Instalar e habilitar UFW
```bash
sudo apt update
sudo apt install ufw -y
sudo ufw enable
```

### Regras básicas recomendadas

```bash
# Permitir SSH (IMPORTANTE: faça isso ANTES de habilitar o firewall!)
sudo ufw allow 22/tcp

# Permitir HTTP e HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Ver regras ativas
sudo ufw status verbose

# Se precisar permitir uma porta específica depois
sudo ufw allow 8080/tcp

# Bloquear uma porta específica
sudo ufw deny 3306/tcp  # exemplo: bloquear MySQL
```

### Regras mais específicas (opcional)

```bash
# Permitir SSH apenas de um IP específico (mais seguro)
sudo ufw delete allow 22/tcp
sudo ufw allow from SEU_IP_AQUI to any port 22

# Permitir apenas de uma faixa de IPs
sudo ufw allow from 192.168.1.0/24 to any port 22
```

### Desabilitar UFW (se necessário)
```bash
sudo ufw disable
```

## 2. Configurar SSH com chave (sem senha)

### Gerar chave SSH no seu computador local

**No WSL/Linux:**
```bash
ssh-keygen -t ed25519 -C "seu_email@exemplo.com"
# Pressione Enter para aceitar o caminho padrão
# Digite uma senha forte (ou deixe vazio se preferir)
```

**No Windows (PowerShell):**
```powershell
ssh-keygen -t ed25519 -C "seu_email@exemplo.com"
```

### Copiar chave pública para a VPS

```bash
# Método 1: ssh-copy-id (mais fácil)
ssh-copy-id usuario@ip_da_vps

# Método 2: Manual
cat ~/.ssh/id_ed25519.pub
# Copie o conteúdo e cole no servidor (veja próximo passo)
```

### No servidor VPS: adicionar chave autorizada

```bash
# Criar diretório .ssh se não existir
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Adicionar sua chave pública
nano ~/.ssh/authorized_keys
# Cole sua chave pública aqui (uma linha)

# Ajustar permissões
chmod 600 ~/.ssh/authorized_keys
```

### Desabilitar login por senha no SSH

**IMPORTANTE:** Teste o login com chave ANTES de fazer isso!

```bash
sudo nano /etc/ssh/sshd_config
```

Procure e altere/modifique estas linhas:
```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no  # ou yes se precisar, mas não recomendado
```

Reinicie o SSH:
```bash
sudo systemctl restart sshd
# ou
sudo systemctl restart ssh
```

### Testar conexão SSH
```bash
ssh usuario@ip_da_vps
# Deve conectar sem pedir senha (se configurou corretamente)
```

## 3. Atualizar o sistema regularmente

### Atualizações de segurança
```bash
# Atualizar lista de pacotes
sudo apt update

# Ver pacotes que podem ser atualizados
sudo apt list --upgradable

# Instalar atualizações de segurança automaticamente
sudo apt upgrade -y

# Limpar pacotes antigos
sudo apt autoremove -y
sudo apt autoclean
```

### Configurar atualizações automáticas (opcional)

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades
# Escolha "Yes" quando perguntado
```

## 4. Criar usuário não-root (boas práticas)

### Criar novo usuário
```bash
sudo adduser novo_usuario
# Defina uma senha forte

# Adicionar ao grupo sudo (para poder usar sudo)
sudo usermod -aG sudo novo_usuario
```

### Testar login com novo usuário
```bash
# Sair e entrar de novo com o novo usuário
exit
ssh novo_usuario@ip_da_vps
```

### Desabilitar login do root (opcional, mas recomendado)

```bash
sudo nano /etc/ssh/sshd_config
```

Altere:
```
PermitRootLogin no
```

Reinicie:
```bash
sudo systemctl restart sshd
```

## 5. Configurar fail2ban (proteção contra ataques de força bruta)

### Instalar fail2ban
```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Configurar fail2ban para SSH
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Procure a seção `[sshd]` e ajuste:
```
[sshd]
enabled = true
port = 22
maxretry = 3
bantime = 3600
findtime = 600
```

Reinicie:
```bash
sudo systemctl restart fail2ban
```

### Ver status do fail2ban
```bash
sudo fail2ban-client status sshd
```

## 6. Configurar logwatch (monitoramento de logs)

### Instalar logwatch
```bash
sudo apt install logwatch -y
```

### Configurar (opcional)
```bash
sudo nano /etc/logwatch/conf/logwatch.conf
```

### Testar
```bash
sudo logwatch --range yesterday
```

## 7. Verificar portas abertas

### Ver quais portas estão escutando
```bash
sudo netstat -tulpn
# ou
sudo ss -tulpn
```

### Ver conexões ativas
```bash
sudo netstat -an | grep ESTABLISHED
```

## 8. Checklist de segurança básica

- [ ] Firewall (UFW) configurado e ativo
- [ ] SSH configurado com chave (sem senha)
- [ ] Login root desabilitado (ou protegido)
- [ ] Sistema atualizado (`sudo apt update && sudo apt upgrade`)
- [ ] Fail2ban instalado e ativo
- [ ] Apenas portas necessárias abertas (22, 80, 443)
- [ ] Usuário não-root criado (se aplicável)
- [ ] Senhas fortes configuradas (se ainda usar senha em algum lugar)

## 9. Comandos úteis para monitoramento

### Ver tentativas de login falhadas
```bash
sudo grep "Failed password" /var/log/auth.log
```

### Ver últimos logins
```bash
last
lastlog
```

### Ver processos rodando
```bash
ps aux
top
htop  # se instalado: sudo apt install htop
```

### Ver uso de disco
```bash
df -h
du -sh /caminho/do/diretorio
```

## 10. Observações importantes

- **SEMPRE teste SSH com chave antes de desabilitar senha**.
- **Mantenha backups** das suas chaves SSH em local seguro.
- **Atualize o sistema regularmente** (mensalmente ou quando houver alertas de segurança).
- **Monitore logs** periodicamente para detectar atividades suspeitas.
- **Use senhas fortes** mesmo quando usar chaves SSH (para a chave em si).
