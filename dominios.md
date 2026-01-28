# Estrutura de Domínios na VPS

Este tutorial explica, de forma genérica, como usar **NGINX instalado diretamente na VPS** (fora do Docker) como gateway central para rotear múltiplos domínios para diferentes aplicações (normalmente containers Docker rodando em portas internas).

- **`app1.seudominio.com`** → App 1 → `localhost:8080`
- **`app2.seudominio.com`** → App 2 → `localhost:8081`
- Outros domínios podem ser adicionados criando novos arquivos em `/etc/nginx/sites-available/`

**Fluxo de requisições (exemplo com frontend + backend):**
```
Internet → NGINX na VPS (porta 80/443) → localhost:8080 → Container Frontend (porta 80)
                                                                  ↓
                                                             Backend (porta interna, ex: 3001)
```

## 1. Instalar NGINX na VPS

Se ainda não tiver NGINX instalado na VPS (Ubuntu/Debian):

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 2. Configurar DNS no provedor

No painel do seu provedor de domínio (ex.: `seudominio.com`):

- **Para o App 1**
  - Crie um registro **A**: `app1` → IP público da VPS
- **Para o App 2 (opcional)**
  - Crie um registro **A**: `app2` → IP público da VPS

Depois da propagação do DNS, `app1.seudominio.com` e `app2.seudominio.com` devem apontar para a mesma VPS (quem decide para qual app ir é o NGINX).

## 3. Configurar NGINX para um domínio

Exemplo para o domínio `app1.seudominio.com` apontando para um serviço ouvindo em `localhost:8080` (pode ser um container Docker expondo `8080:80`):

```bash
sudo nano /etc/nginx/sites-available/app1.seudominio.com
```

Conteúdo:
```nginx
server {
    listen 80;
    server_name app1.seudominio.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
    }
}
```

Ative o site e recarregue o NGINX:

```bash
sudo ln -s /etc/nginx/sites-available/app1.seudominio.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## 4. Habilitar SSL/HTTPS usando Cloudflare

Neste fluxo, **o SSL fica 100% gerenciado pelo Cloudflare**. A VPS responde em **HTTP (porta 80)** e o Cloudflare faz HTTPS com o usuário final.

### 4.1. Pré‑requisitos

- Domínio (`seudominio.com`) adicionado ao Cloudflare.
- Nameservers do domínio já apontando para os do Cloudflare.
- Registro **A** para `app1.seudominio.com` criado no Cloudflare, apontando para o IP da VPS, com o ícone de nuvem **laranja** (proxy habilitado).

### 4.2. Configurar SSL no painel do Cloudflare

1. Acesse o painel do Cloudflare e clique no seu domínio (`seudominio.com`).
2. No menu lateral, vá em **"SSL/TLS" → "Overview"**.
3. Em **"SSL/TLS encryption mode"**, escolha:
   - **Flexible**: Cloudflare fala HTTPS com o usuário e HTTP com a VPS (mais simples, mas menos seguro).
   - **Full**: Cloudflare fala HTTPS com a VPS usando um certificado (pode ser autoassinado).
   - **Full (strict)**: recomenda‑se quando você instalar um certificado válido na VPS (por exemplo, Origin Certificate da própria Cloudflare).

Para começar rápido, você pode usar **"Flexible"** (sem precisar mexer em certificados na VPS). Depois, se quiser endurecer a segurança, pode migrar para **"Full (strict)"** gerando um **Origin Certificate** em **"SSL/TLS" → "Origin Server"** e instalando no NGINX.

### 4.3. Redirecionar tudo para HTTPS

Ainda no Cloudflare:

1. Vá em **"SSL/TLS" → "Edge Certificates"**.
2. Ative a opção **"Always Use HTTPS"**.

Isso faz com que qualquer acesso `http://app1.seudominio.com` seja automaticamente redirecionado para `https://app1.seudominio.com` na borda do Cloudflare.

### 4.4. Ajustar URL da aplicação

Se a aplicação usa variáveis como `FRONTEND_URL`, use sempre a URL HTTPS final, por exemplo:

```env
FRONTEND_URL=https://app1.seudominio.com
```

Depois, reinicie seus containers (ajuste o comando para o seu projeto):

```bash
docker compose -f docker-compose.prod.yml --env-file .env.prod restart frontend backend
```

## 5. Adicionar novos apps/domínios

Para um novo app em `app2.seudominio.com`:

1. **DNS**: criar registro **A** `app2` → IP da VPS.
2. **Novo arquivo NGINX**: criar `/etc/nginx/sites-available/app2.seudominio.com` apontando para `localhost:8081` (outra porta).
3. **Container do novo app**: expor a porta correta, por exemplo `8081:80` no `docker compose` ou no `docker run`.
4. **Ativar o site**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/app2.seudominio.com /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```
5. (Opcional) Rodar `certbot` de novo para esse novo domínio:
   ```bash
   sudo certbot --nginx -d app2.seudominio.com
   ```

## 6. Observações gerais

- **NGINX na VPS** expõe as portas 80/443 para a internet.
- **Containers Docker** normalmente expõem portas locais (ex.: `127.0.0.1:8080`, `127.0.0.1:8081`) que só a própria VPS enxerga.
- O NGINX faz o papel de "porteiro" por domínio (`server_name`) e encaminha para a porta interna correta.
- Se o seu frontend já faz proxy de `/api` para o backend (via um `nginx.conf` dentro do container do frontend), normalmente você só precisa expor **uma porta** (do frontend) para a VPS, e de lá o NGINX da VPS encaminha tudo para esse frontend.
