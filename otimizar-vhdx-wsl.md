# Otimizar VHDX do WSL

Este tutorial explica como reduzir o tamanho do arquivo VHDX do WSL2 no Windows, liberando espaço em disco quando o WSL cresceu muito.

## 1. Por que o VHDX cresce?

O arquivo VHDX do WSL2 (`ext4.vhdx`) **não diminui automaticamente** quando você deleta arquivos dentro do WSL. Ele mantém o espaço alocado mesmo após remover dados.

**Exemplos que fazem o VHDX crescer:**
- Instalação de pacotes grandes
- Imagens Docker não utilizadas
- Logs acumulados
- Cache de dependências (`node_modules`, `__pycache__`, etc.)
- Arquivos temporários

## 2. Limpar espaço dentro do WSL (antes de compactar)

### Limpar cache do apt
```bash
sudo apt clean
sudo apt autoremove -y
sudo apt autoclean
```

### Limpar imagens Docker não utilizadas
```bash
# Remover imagens não utilizadas
docker image prune -a

# Remover tudo (containers, imagens, volumes, redes)
docker system prune -a --volumes

# Ver quanto espaço está sendo usado pelo Docker
docker system df
```

### Limpar logs antigos
```bash
# Limpar logs do sistema (cuidado!)
sudo journalctl --vacuum-time=7d  # manter últimos 7 dias
sudo journalctl --vacuum-size=100M  # ou manter até 100MB

# Limpar logs de aplicações específicas
sudo find /var/log -type f -name "*.log" -mtime +30 -delete
```

### Limpar cache de pacotes (npm, pip, etc)
```bash
# npm
npm cache clean --force

# pip (Python)
pip cache purge

# yarn
yarn cache clean

# pnpm
pnpm store prune
```

### Encontrar arquivos grandes
```bash
# Top 10 maiores diretórios
du -h --max-depth=1 / | sort -hr | head -10

# Top 10 maiores arquivos
find / -type f -size +100M 2>/dev/null | xargs ls -lh | sort -k5 -hr | head -10
```

### Limpar arquivos temporários
```bash
# Limpar /tmp
sudo rm -rf /tmp/*

# Limpar cache do usuário
rm -rf ~/.cache/*
```

## 3. Compactar o VHDX no Windows

### Passo 1: Desligar o WSL completamente

**No PowerShell do Windows (como Administrador):**
```powershell
wsl --shutdown
```

**Verificar se desligou:**
```powershell
wsl --list --verbose
# Deve mostrar "Stopped" para todas as distribuições
```

### Passo 2: Localizar o arquivo VHDX

O arquivo geralmente está em:
```
C:\Users\SEU_USUARIO\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu*\LocalState\ext4.vhdx
```

**Ou para WSL2 instalado via Microsoft Store:**
```
C:\Users\SEU_USUARIO\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_*\LocalState\ext4.vhdx
```

**Para encontrar o caminho exato:**
```powershell
# No PowerShell
Get-ChildItem -Path $env:LOCALAPPDATA\Packages -Filter "ext4.vhdx" -Recurse -ErrorAction SilentlyContinue | Select-Object FullName
```

### Passo 3: Compactar o VHDX

**No PowerShell do Windows (como Administrador):**

```powershell
# Substitua o caminho pelo seu VHDX
$vhdxPath = "C:\Users\SEU_USUARIO\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_*\LocalState\ext4.vhdx"

# Compactar
Optimize-VHD -Path $vhdxPath -Mode Full
```

**Ou usando o caminho completo diretamente:**
```powershell
Optimize-VHD -Path "C:\Users\SEU_USUARIO\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_79rhkp1fndgsc\LocalState\ext4.vhdx" -Mode Full
```

**Verificar tamanho antes e depois:**
```powershell
# Antes
Get-Item "C:\Users\SEU_USUARIO\AppData\Local\Packages\...\ext4.vhdx" | Select-Object Name, @{Name="Size(GB)";Expression={[math]::Round($_.Length / 1GB, 2)}}

# Depois (mesmo comando após compactar)
```

### Passo 4: Reiniciar o WSL

```powershell
wsl
# ou
wsl -d Ubuntu
```

## 4. Script PowerShell completo (automatizado)

```powershell
# otimizar-wsl.ps1
# Execute como Administrador

Write-Host "=== Otimização do WSL VHDX ===" -ForegroundColor Cyan

# 1. Desligar WSL
Write-Host "1. Desligando WSL..." -ForegroundColor Yellow
wsl --shutdown
Start-Sleep -Seconds 3

# 2. Encontrar VHDX
Write-Host "2. Procurando arquivo VHDX..." -ForegroundColor Yellow
$vhdxFiles = Get-ChildItem -Path $env:LOCALAPPDATA\Packages -Filter "ext4.vhdx" -Recurse -ErrorAction SilentlyContinue

if ($vhdxFiles.Count -eq 0) {
    Write-Host "Nenhum arquivo VHDX encontrado!" -ForegroundColor Red
    exit
}

foreach ($vhdx in $vhdxFiles) {
    Write-Host "`nArquivo encontrado: $($vhdx.FullName)" -ForegroundColor Green
    
    # Tamanho antes
    $sizeBefore = [math]::Round($vhdx.Length / 1GB, 2)
    Write-Host "Tamanho antes: $sizeBefore GB" -ForegroundColor Cyan
    
    # 3. Compactar
    Write-Host "3. Compactando (isso pode demorar alguns minutos)..." -ForegroundColor Yellow
    Optimize-VHD -Path $vhdx.FullName -Mode Full
    
    # Tamanho depois
    $vhdx.Refresh()
    $sizeAfter = [math]::Round($vhdx.Length / 1GB, 2)
    $saved = [math]::Round($sizeBefore - $sizeAfter, 2)
    
    Write-Host "Tamanho depois: $sizeAfter GB" -ForegroundColor Green
    Write-Host "Espaço liberado: $saved GB" -ForegroundColor Green
}

Write-Host "`n=== Otimização concluída! ===" -ForegroundColor Cyan
```

**Como usar:**
```powershell
# Salvar como otimizar-wsl.ps1
# Executar como Administrador
.\otimizar-wsl.ps1
```

## 5. Limpeza dentro do WSL antes de compactar (script completo)

```bash
#!/bin/bash
# limpar-wsl.sh

echo "=== Limpeza do WSL antes de compactar VHDX ==="

echo "1. Limpando cache do apt..."
sudo apt clean
sudo apt autoremove -y
sudo apt autoclean

echo "2. Limpando Docker..."
if command -v docker &> /dev/null; then
    docker system prune -a --volumes -f
    echo "   Espaço usado pelo Docker:"
    docker system df
fi

echo "3. Limpando logs do sistema..."
sudo journalctl --vacuum-time=7d

echo "4. Limpando cache de pacotes..."
# npm
if command -v npm &> /dev/null; then
    npm cache clean --force
fi

# pip
if command -v pip &> /dev/null; then
    pip cache purge
fi

echo "5. Limpando arquivos temporários..."
sudo rm -rf /tmp/*
rm -rf ~/.cache/*

echo "6. Verificando espaço em disco..."
df -h /

echo "=== Limpeza concluída! ==="
echo "Agora execute 'wsl --shutdown' no PowerShell e compacte o VHDX"
```

## 6. Verificar espaço liberado

### Dentro do WSL
```bash
df -h /
```

### No Windows (PowerShell)
```powershell
# Ver tamanho do VHDX
Get-Item "C:\Users\SEU_USUARIO\AppData\Local\Packages\...\ext4.vhdx" | 
    Select-Object Name, @{Name="Size(GB)";Expression={[math]::Round($_.Length / 1GB, 2)}}
```

## 7. Dicas para evitar crescimento excessivo

### Configurar limite de tamanho do VHDX (WSL2)

Crie/edite `.wslconfig` em `C:\Users\SEU_USUARIO\.wslconfig`:

```ini
[wsl2]
memory=4GB
processors=2
swap=2GB
localhostForwarding=true
# Limite de tamanho do VHDX (ajuste conforme necessário)
disk=100GB
```

Depois:
```powershell
wsl --shutdown
wsl
```

### Limpar Docker regularmente
```bash
# Adicionar ao crontab (limpeza semanal)
0 3 * * 0 docker system prune -a --volumes -f
```

### Monitorar espaço
```bash
# Script simples para verificar espaço
#!/bin/bash
df -h / | tail -1 | awk '{print "Espaço usado: " $3 " / " $2 " (" $5 ")"}'
```

## 8. Troubleshooting

### Erro: "The process cannot access the file because it is being used"

**Solução:** Certifique-se de que o WSL está completamente desligado:
```powershell
wsl --shutdown
# Aguarde alguns segundos
Get-Process | Where-Object {$_.ProcessName -like "*wsl*"}
# Se ainda houver processos, force o encerramento ou reinicie o computador
```

### Erro: "Access Denied"

**Solução:** Execute o PowerShell como Administrador.

### VHDX não diminui muito

**Possíveis causas:**
- Ainda há muitos arquivos dentro do WSL (faça mais limpeza)
- O VHDX tem fragmentação interna (pode precisar de mais tempo)
- Arquivos grandes ainda estão presentes

**Solução:** Execute a limpeza dentro do WSL novamente e tente compactar de novo.

## 9. Observações importantes

- **Sempre desligue o WSL completamente** antes de compactar (`wsl --shutdown`).
- **Faça backup** se tiver dados importantes (raramente necessário, mas é uma boa prática).
- **A compactação pode demorar** dependendo do tamanho do VHDX (vá fazer um café ☕).
- **Limpe dentro do WSL primeiro** para melhor resultado na compactação.
- **Execute PowerShell como Administrador** para ter permissões necessárias.
