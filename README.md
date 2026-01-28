# 📚 Rafa Tutorials

Repositório com tutoriais práticos sobre infraestrutura, DevOps e ferramentas que uso no dia a dia. Todos os tutoriais são baseados em experiências reais e incluem comandos prontos para uso.

## 📋 Índice

- [Domínios e NGINX](#-domínios-e-nginx)
- [Docker na VPS](#-docker-na-vps)
- [Segurança da VPS](#-segurança-da-vps)
- [Backups](#-backups)
- [Otimização WSL](#-otimização-wsl)

---

## 🌐 Domínios e NGINX

**[📄 Ver tutorial completo](./dominios.md)**

Como configurar NGINX na VPS como gateway central para rotear múltiplos domínios para diferentes aplicações Docker.

**Tópicos cobertos:**
- Instalação e configuração do NGINX na VPS
- Configuração de DNS no provedor
- Roteamento de domínios para containers Docker
- Configuração de SSL/HTTPS usando Cloudflare
- Adicionar novos apps/domínios

**Quando usar:** Quando você precisa hospedar múltiplas aplicações na mesma VPS e rotear por domínio.

---

## 🐳 Docker na VPS

**[📄 Ver tutorial completo](./docker-comandos-vps.md)**

Comandos essenciais do Docker para gerenciar aplicações em produção, incluindo comandos compostos que executam múltiplas operações.

**Tópicos cobertos:**
- Comandos básicos de containers
- Docker Compose (subir, parar, logs, reiniciar)
- Comandos compostos (múltiplas ações em uma linha)
- Gerenciamento de imagens e limpeza
- Debug e monitoramento
- Workflows práticos (deploy, rollback, verificação)

**Quando usar:** Para gerenciar containers Docker em produção, fazer deploys e troubleshooting.

---

## 🔒 Segurança da VPS

**[📄 Ver tutorial completo](./seguranca-vps.md)**

Configurações básicas de segurança para uma VPS Linux, incluindo firewall, SSH e boas práticas.

**Tópicos cobertos:**
- Configuração de firewall (UFW)
- SSH com chave (sem senha)
- Atualizações do sistema
- Criar usuário não-root
- Fail2ban (proteção contra força bruta)
- Monitoramento de logs
- Checklist de segurança

**Quando usar:** Ao configurar uma nova VPS ou melhorar a segurança de uma existente.

---

## 💾 Backups

**[📄 Ver tutorial completo](./backups.md)**

Estratégias básicas de backup para VPS e projetos específicos (Docker, bancos de dados, arquivos de configuração).

**Tópicos cobertos:**
- Backups na VPS (arquivos gerais, NGINX)
- Backups de projetos Docker (configs, volumes)
- Backups de bancos de dados (PostgreSQL, MySQL)
- Scripts completos de backup
- Automatização com cron
- Limpeza de backups antigos
- Backup para storage externo

**Quando usar:** Para proteger seus dados e configurar rotinas de backup automatizadas.

---

## ⚡ Otimização WSL

**[📄 Ver tutorial completo](./otimizar-vhdx-wsl.md)**

Como reduzir o tamanho do arquivo VHDX do WSL2 no Windows, liberando espaço em disco.

**Tópicos cobertos:**
- Por que o VHDX cresce
- Limpeza dentro do WSL (Docker, logs, cache)
- Compactação do VHDX no Windows
- Scripts PowerShell automatizados
- Scripts bash de limpeza
- Dicas para evitar crescimento excessivo
- Troubleshooting comum

**Quando usar:** Quando o WSL está ocupando muito espaço em disco e você precisa liberar.

---

## 🛠️ Stack Tecnológica

Estes tutoriais cobrem:

- **Infraestrutura:** VPS Linux (Ubuntu/Debian), NGINX, Docker
- **Cloud:** Cloudflare (SSL/TLS, DNS)
- **Desenvolvimento:** WSL2, Docker Compose
- **Bancos de Dados:** PostgreSQL, MySQL/MariaDB
- **Segurança:** UFW, SSH, Fail2ban

## 📝 Estrutura do Repositório

```
rafa-tutorials/
├── README.md                 # Este arquivo
├── dominios.md               # Configuração de domínios e NGINX
├── docker-comandos-vps.md    # Comandos Docker para produção
├── seguranca-vps.md          # Segurança básica da VPS
├── backups.md                # Estratégias de backup
└── otimizar-vhdx-wsl.md      # Otimização do WSL
```

## 🚀 Como Usar

1. Navegue até o tutorial que você precisa
2. Siga os passos descritos
3. Ajuste os comandos conforme seu ambiente
4. Todos os tutoriais incluem exemplos práticos e comandos prontos

## 💡 Dicas

- **Sempre teste em ambiente de desenvolvimento primeiro** antes de aplicar em produção
- **Faça backups** antes de operações destrutivas
- **Ajuste os comandos** conforme sua configuração específica
- **Leia as observações** no final de cada tutorial

## 📄 Licença

Estes tutoriais são de uso livre. Sinta-se à vontade para usar, modificar e compartilhar.

---

**Última atualização:** Janeiro 2026
