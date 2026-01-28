# Comandos Docker na VPS

Este tutorial lista comandos úteis do Docker para gerenciar aplicações em produção na VPS, incluindo comandos compostos que fazem várias operações de uma vez.

## 1. Comandos básicos de containers

### Ver containers rodando
```bash
docker ps
```

### Ver todos os containers (incluindo parados)
```bash
docker ps -a
```

### Ver logs de um container
```bash
# Logs em tempo real (seguir)
docker logs -f nome_do_container

# Últimas 100 linhas
docker logs --tail 100 nome_do_container

# Logs com timestamp
docker logs -f --timestamps nome_do_container
```

### Entrar dentro de um container
```bash
docker exec -it nome_do_container /bin/bash
# ou /bin/sh se bash não estiver disponível
```

### Parar/Iniciar/Reiniciar um container
```bash
docker stop nome_do_container
docker start nome_do_container
docker restart nome_do_container
```

### Remover um container
```bash
# Parar e remover
docker rm -f nome_do_container
```

## 2. Comandos com Docker Compose

### Subir todos os serviços
```bash
docker compose -f docker-compose.prod.yml --env-file .env.prod up -d
```

### Parar todos os serviços
```bash
docker compose -f docker-compose.prod.yml down
```

### Ver logs de todos os serviços
```bash
docker compose -f docker-compose.prod.yml logs -f
```

### Ver logs de um serviço específico
```bash
docker compose -f docker-compose.prod.yml logs -f frontend
docker compose -f docker-compose.prod.yml logs -f backend
```

### Reiniciar um serviço específico
```bash
docker compose -f docker-compose.prod.yml restart frontend
```

### Reconstruir e subir um serviço específico (sem afetar outros)
```bash
docker compose -f docker-compose.prod.yml up -d --build --no-deps frontend
```

## 3. Comandos compostos (múltiplas ações)

### Atualizar imagens, reconstruir e reiniciar tudo
```bash
docker compose -f docker-compose.prod.yml --env-file .env.prod pull && \
docker compose -f docker-compose.prod.yml --env-file .env.prod up -d --build
```

### Atualizar apenas um serviço específico (sem downtime)
```bash
docker compose -f docker-compose.prod.yml --env-file .env.prod pull frontend && \
docker compose -f docker-compose.prod.yml --env-file .env.prod up -d --no-deps --build frontend
```

### Parar tudo, limpar volumes e subir de novo (cuidado!)
```bash
docker compose -f docker-compose.prod.yml down -v && \
docker compose -f docker-compose.prod.yml --env-file .env.prod up -d --build
```

### Verificar saúde e reiniciar se necessário
```bash
# Verificar se está rodando
docker compose -f docker-compose.prod.yml ps

# Se algum serviço estiver com problema, reiniciar
docker compose -f docker-compose.prod.yml restart
```

### Atualizar e limpar imagens antigas em uma linha
```bash
docker compose -f docker-compose.prod.yml pull && \
docker compose -f docker-compose.prod.yml up -d && \
docker image prune -f
```

## 4. Gerenciamento de imagens

### Listar imagens
```bash
docker images
```

### Baixar uma imagem específica
```bash
docker pull nome_da_imagem:tag
```

### Remover imagens não utilizadas
```bash
docker image prune -a
```

### Remover imagens antigas (mais de 7 dias)
```bash
docker image prune -a --filter "until=168h"
```

## 5. Limpeza geral do Docker

### Remover containers parados, redes não usadas e imagens órfãs
```bash
docker system prune -a
```

### Limpar tudo (containers, imagens, volumes, redes) - CUIDADO!
```bash
docker system prune -a --volumes
```

### Ver quanto espaço está sendo usado
```bash
docker system df
```

## 6. Comandos úteis para debug

### Ver uso de recursos (CPU, memória, rede)
```bash
docker stats
```

### Ver uso de recursos de um container específico
```bash
docker stats nome_do_container
```

### Verificar configuração de um container
```bash
docker inspect nome_do_container
```

### Ver variáveis de ambiente de um container
```bash
docker exec nome_do_container env
```

### Testar conectividade entre containers
```bash
docker exec -it container_origem ping container_destino
```

## 7. Exemplos práticos de workflow

### Deploy completo de uma nova versão
```bash
# 1. Entrar na pasta do projeto
cd /app/seu-projeto

# 2. Fazer backup rápido (opcional, ver tutorial de backups)
cp .env.prod .env.prod.backup

# 3. Atualizar código (se necessário)
git pull origin main

# 4. Atualizar imagens e subir
docker compose -f docker-compose.prod.yml --env-file .env.prod pull && \
docker compose -f docker-compose.prod.yml --env-file .env.prod up -d --build

# 5. Verificar se subiu corretamente
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs --tail 50
```

### Rollback rápido (voltar versão anterior)
```bash
# 1. Ver imagens disponíveis
docker images | grep nome_da_imagem

# 2. Voltar para tag anterior
docker compose -f docker-compose.prod.yml --env-file .env.prod up -d nome_da_imagem:tag_anterior

# Ou restaurar .env e subir de novo
cp .env.prod.backup .env.prod
docker compose -f docker-compose.prod.yml --env-file .env.prod up -d
```

### Verificar saúde após deploy
```bash
# Ver status
docker compose -f docker-compose.prod.yml ps

# Ver logs recentes
docker compose -f docker-compose.prod.yml logs --tail 100

# Testar endpoint (ajuste a URL)
curl -I https://app1.seudominio.com
```

## 8. Observações importantes

- **Sempre use `-f` e `--env-file`** quando trabalhar com arquivos específicos de produção.
- **`--no-deps`** é útil para atualizar um serviço sem afetar dependências.
- **`docker system prune`** pode remover dados importantes, use com cuidado.
- **Sempre verifique logs** após um deploy (`docker compose logs -f`).
- **Mantenha backups** antes de operações destrutivas (ver tutorial de backups).
