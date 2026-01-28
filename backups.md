# Backups Simples - VPS e Projetos

Este tutorial cobre estratégias básicas de backup tanto para a VPS em geral quanto para projetos específicos (Docker, arquivos de configuração, bancos de dados).

## 1. Backups na VPS (arquivos gerais)

### Backup de arquivos importantes do sistema

#### Script básico de backup
```bash
#!/bin/bash
# Criar arquivo: nano ~/backup-vps.sh

BACKUP_DIR="/home/usuario/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="vps-backup-$DATE.tar.gz"

mkdir -p $BACKUP_DIR

# Criar backup de arquivos importantes
tar -czf $BACKUP_DIR/$BACKUP_FILE \
    /etc/nginx \
    /etc/ssh/sshd_config \
    /home/usuario/.ssh \
    /var/log/nginx \
    2>/dev/null

echo "Backup criado: $BACKUP_DIR/$BACKUP_FILE"
```

#### Tornar executável e rodar
```bash
chmod +x ~/backup-vps.sh
~/backup-vps.sh
```

### Backup de configurações do NGINX
```bash
# Backup simples
sudo tar -czf ~/nginx-backup-$(date +%Y%m%d).tar.gz /etc/nginx

# Restaurar (cuidado!)
sudo tar -xzf ~/nginx-backup-20250128.tar.gz -C /
sudo nginx -t
sudo systemctl reload nginx
```

## 2. Backups de projetos Docker

### Backup de arquivos de configuração do projeto

#### Script de backup de projeto
```bash
#!/bin/bash
# Criar: nano ~/backup-projeto.sh

PROJECT_DIR="/app/seu-projeto"
BACKUP_DIR="/home/usuario/backups/projetos"
DATE=$(date +%Y%m%d_%H%M%S)
PROJECT_NAME=$(basename $PROJECT_DIR)

mkdir -p $BACKUP_DIR

# Backup de arquivos importantes (sem node_modules, .git, etc)
tar -czf $BACKUP_DIR/${PROJECT_NAME}-$DATE.tar.gz \
    -C $PROJECT_DIR \
    --exclude='node_modules' \
    --exclude='.git' \
    --exclude='dist' \
    --exclude='build' \
    --exclude='*.log' \
    docker-compose.prod.yml \
    .env.prod \
    Dockerfile \
    nginx.conf \
    # adicione outros arquivos importantes aqui

echo "Backup do projeto criado: $BACKUP_DIR/${PROJECT_NAME}-$DATE.tar.gz"
```

### Backup de volumes Docker (dados persistentes)

#### Listar volumes
```bash
docker volume ls
```

#### Backup de um volume específico
```bash
#!/bin/bash
# Backup de volume Docker

VOLUME_NAME="seu-projeto_db_data"
BACKUP_DIR="/home/usuario/backups/volumes"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Criar container temporário para fazer backup
docker run --rm \
    -v $VOLUME_NAME:/data \
    -v $BACKUP_DIR:/backup \
    alpine tar czf /backup/${VOLUME_NAME}-$DATE.tar.gz -C /data .

echo "Backup do volume criado: $BACKUP_DIR/${VOLUME_NAME}-$DATE.tar.gz"
```

#### Restaurar volume de um backup
```bash
# Parar containers que usam o volume
docker compose -f docker-compose.prod.yml down

# Restaurar
docker run --rm \
    -v $VOLUME_NAME:/data \
    -v $BACKUP_DIR:/backup \
    alpine sh -c "cd /data && tar xzf /backup/${VOLUME_NAME}-20250128.tar.gz"

# Subir containers de novo
docker compose -f docker-compose.prod.yml --env-file .env.prod up -d
```

## 3. Backups de banco de dados

### PostgreSQL (via Docker)

#### Backup
```bash
#!/bin/bash
# Backup PostgreSQL

CONTAINER_NAME="seu-projeto-db-1"  # ajuste o nome
DB_NAME="seu_banco"
DB_USER="seu_usuario"
BACKUP_DIR="/home/usuario/backups/db"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

docker exec $CONTAINER_NAME pg_dump -U $DB_USER $DB_NAME > $BACKUP_DIR/db-backup-$DATE.sql

# Ou comprimido
docker exec $CONTAINER_NAME pg_dump -U $DB_USER $DB_NAME | gzip > $BACKUP_DIR/db-backup-$DATE.sql.gz

echo "Backup do banco criado: $BACKUP_DIR/db-backup-$DATE.sql.gz"
```

#### Restaurar
```bash
# Descomprimir se necessário
gunzip db-backup-20250128.sql.gz

# Restaurar
docker exec -i $CONTAINER_NAME psql -U $DB_USER $DB_NAME < db-backup-20250128.sql

# Ou direto do arquivo comprimido
gunzip -c db-backup-20250128.sql.gz | docker exec -i $CONTAINER_NAME psql -U $DB_USER $DB_NAME
```

### MySQL/MariaDB (via Docker)

#### Backup
```bash
#!/bin/bash
# Backup MySQL

CONTAINER_NAME="seu-projeto-db-1"
DB_NAME="seu_banco"
DB_USER="root"
DB_PASSWORD="sua_senha"
BACKUP_DIR="/home/usuario/backups/db"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

docker exec $CONTAINER_NAME mysqldump -u $DB_USER -p$DB_PASSWORD $DB_NAME | gzip > $BACKUP_DIR/db-backup-$DATE.sql.gz

echo "Backup do banco criado: $BACKUP_DIR/db-backup-$DATE.sql.gz"
```

#### Restaurar
```bash
gunzip -c db-backup-20250128.sql.gz | docker exec -i $CONTAINER_NAME mysql -u root -p$DB_PASSWORD $DB_NAME
```

## 4. Script completo de backup de projeto

```bash
#!/bin/bash
# backup-projeto-completo.sh

PROJECT_DIR="/app/seu-projeto"
BACKUP_BASE="/home/usuario/backups"
PROJECT_NAME=$(basename $PROJECT_DIR)
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="$BACKUP_BASE/$PROJECT_NAME-$DATE"

mkdir -p $BACKUP_DIR

echo "=== Iniciando backup de $PROJECT_NAME ==="

# 1. Backup de arquivos de configuração
echo "1. Backup de arquivos de configuração..."
tar -czf $BACKUP_DIR/config.tar.gz \
    -C $PROJECT_DIR \
    docker-compose.prod.yml \
    .env.prod \
    Dockerfile \
    nginx.conf

# 2. Backup de volumes Docker (se houver)
echo "2. Backup de volumes Docker..."
docker volume ls | grep $PROJECT_NAME | awk '{print $2}' | while read volume; do
    echo "  Backup do volume: $volume"
    docker run --rm \
        -v $volume:/data \
        -v $BACKUP_DIR:/backup \
        alpine tar czf /backup/volume-${volume}.tar.gz -C /data .
done

# 3. Backup de banco de dados (PostgreSQL exemplo)
echo "3. Backup de banco de dados..."
CONTAINER_DB=$(docker ps --format "{{.Names}}" | grep -i db | head -1)
if [ ! -z "$CONTAINER_DB" ]; then
    docker exec $CONTAINER_DB pg_dump -U postgres postgres | gzip > $BACKUP_DIR/database.sql.gz
fi

# 4. Criar arquivo de informações
echo "4. Criando arquivo de informações..."
cat > $BACKUP_DIR/info.txt << EOF
Projeto: $PROJECT_NAME
Data do backup: $(date)
Diretório original: $PROJECT_DIR
EOF

# 5. Comprimir tudo em um único arquivo
echo "5. Comprimindo backup completo..."
cd $BACKUP_BASE
tar -czf ${PROJECT_NAME}-${DATE}-completo.tar.gz $(basename $BACKUP_DIR)
rm -rf $BACKUP_DIR

echo "=== Backup completo criado: ${PROJECT_NAME}-${DATE}-completo.tar.gz ==="
```

## 5. Automatizar backups com cron

### Editar crontab
```bash
crontab -e
```

### Exemplos de agendamento

```bash
# Backup diário às 2h da manhã
0 2 * * * /home/usuario/backup-projeto-completo.sh >> /home/usuario/backup.log 2>&1

# Backup semanal (domingo às 3h)
0 3 * * 0 /home/usuario/backup-projeto-completo.sh >> /home/usuario/backup.log 2>&1

# Backup de banco a cada 6 horas
0 */6 * * * /home/usuario/backup-db.sh >> /home/usuario/backup-db.log 2>&1
```

## 6. Limpar backups antigos

### Script para remover backups com mais de X dias
```bash
#!/bin/bash
# limpar-backups-antigos.sh

BACKUP_DIR="/home/usuario/backups"
DAYS_TO_KEEP=30  # Manter backups dos últimos 30 dias

find $BACKUP_DIR -type f -name "*.tar.gz" -mtime +$DAYS_TO_KEEP -delete
find $BACKUP_DIR -type f -name "*.sql.gz" -mtime +$DAYS_TO_KEEP -delete

echo "Backups antigos removidos (mantidos últimos $DAYS_TO_KEEP dias)"
```

### Adicionar ao cron (semanal)
```bash
# No crontab: limpar backups antigos toda segunda-feira às 4h
0 4 * * 1 /home/usuario/limpar-backups-antigos.sh
```

## 7. Backup para storage externo (opcional)

### Usando rsync para copiar para outro servidor
```bash
#!/bin/bash
# backup-para-servidor-externo.sh

BACKUP_DIR="/home/usuario/backups"
REMOTE_HOST="usuario@servidor-backup.com"
REMOTE_PATH="/backups/vps-principal"

# Sincronizar backups
rsync -avz --delete $BACKUP_DIR/ $REMOTE_HOST:$REMOTE_PATH/

echo "Backups sincronizados com servidor externo"
```

### Usando SCP para copiar arquivo específico
```bash
scp /home/usuario/backups/projeto-20250128.tar.gz usuario@servidor-backup.com:/backups/
```

## 8. Verificar integridade dos backups

### Testar restauração de arquivos
```bash
# Extrair e verificar conteúdo
tar -tzf backup-projeto-20250128.tar.gz

# Testar restauração em diretório temporário
mkdir /tmp/test-restore
tar -xzf backup-projeto-20250128.tar.gz -C /tmp/test-restore
ls -la /tmp/test-restore
```

### Verificar backup de banco
```bash
# Ver estrutura do dump
head -50 db-backup-20250128.sql

# Verificar se está completo
tail -10 db-backup-20250128.sql
```

## 9. Checklist de backup

- [ ] Scripts de backup criados e testados
- [ ] Cron configurado para backups automáticos
- [ ] Script de limpeza de backups antigos configurado
- [ ] Backups testados (restauração em ambiente de teste)
- [ ] Local de armazenamento definido (VPS local ou servidor externo)
- [ ] Documentação de como restaurar cada tipo de backup

## 10. Observações importantes

- **Teste restaurações periodicamente** - backup sem teste não é confiável.
- **Mantenha múltiplas cópias** - local + remoto quando possível.
- **Documente o processo de restore** - você pode esquecer como fazer depois.
- **Monitore espaço em disco** - backups podem encher o disco rapidamente.
- **Use compressão** (`.tar.gz`, `.sql.gz`) para economizar espaço.
- **Exclua arquivos desnecessários** (`node_modules`, `.git`, `dist`) dos backups de código.
