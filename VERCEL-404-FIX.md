# 🚨 ERRO 404 NA VERCEL - SOLUÇÃO

## 🔍 Diagnóstico

Se você está vendo:
```
404: NOT_FOUND
Code: NOT_FOUND
ID: gru1::xxxxx
```

Isso significa que o deploy falhou ou há problema de configuração.

---

## ✅ SOLUÇÃO PASSO A PASSO

### 1️⃣ VERIFICAR LOGS DO BUILD

**Acesse os logs:**
1. Vá em: https://vercel.com/seu-usuario/seu-projeto
2. Clique em: `Deployments`
3. Clique no último deployment
4. Procure por erros em vermelho

**Erros comuns:**
- ❌ "Cannot find module '@prisma/client'"
- ❌ "Module not found: Can't resolve..."
- ❌ "Type error in..."
- ❌ "Environment variable not found"

---

### 2️⃣ ADICIONAR VARIÁVEIS DE AMBIENTE

**CRÍTICO:** Sem as variáveis, o build SEMPRE falha!

Vá em: `Settings` → `Environment Variables`

**Variáveis OBRIGATÓRIAS:**
```env
DATABASE_URL=postgresql://user:pass@host:5432/db
NEXTAUTH_URL=https://seu-app.vercel.app
NEXTAUTH_SECRET=sua-senha-de-32-caracteres-minimo
```

**Gerar NEXTAUTH_SECRET:**
```bash
# No seu computador:
openssl rand -base64 32

# Copie o resultado e cole na Vercel
```

**Outras variáveis importantes:**
```env
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
UPLOADTHING_SECRET=sk_live_...
UPLOADTHING_APP_ID=app-id
RESEND_API_KEY=re_...
EMAIL_FROM=noreply@seudominio.com
```

**IMPORTANTE:**
- Marque: `Production`, `Preview`, `Development` em TODAS
- Clique `Save` em cada uma
- Depois de salvar TODAS, faça Redeploy

---

### 3️⃣ CORRIGIR package.json (JÁ FEITO!)

Verifique se tem o `postinstall`:

```json
{
  "scripts": {
    "postinstall": "prisma generate"
  }
}
```

✅ Isso já foi adicionado automaticamente!

---

### 4️⃣ FAZER REDEPLOY

**Opção A: Via Interface Vercel**
1. Vá em: `Deployments`
2. Clique nos `...` do último deploy
3. Clique: `Redeploy`
4. Aguarde o build

**Opção B: Git Push**
```bash
# Faça qualquer mudança ou:
git commit --allow-empty -m "trigger redeploy"
git push
```

---

### 5️⃣ VERIFICAR BANCO DE DADOS

**O banco PRECISA estar acessível da internet!**

Teste a conexão:
```bash
# No seu computador:
psql "postgresql://user:pass@host:5432/db"

# Se conectar, significa que a Vercel também consegue
# Se não conectar, veja VPS-SETUP.md
```

**No Coolify:**
- ✅ Marque: "Make it publicly available"
- ✅ Verifique a porta (geralmente 5432)
- ✅ Teste com o IP público, não localhost

**Firewall:**
```bash
# SSH na VPS
ssh root@seu-ip

# Liberar porta do PostgreSQL
sudo ufw allow 5432/tcp
```

---

### 6️⃣ EXECUTAR MIGRATIONS

**Depois que o deploy funcionar:**

```bash
# No seu computador:
cd Esporte

# Criar .env.local com a mesma DATABASE_URL da Vercel
echo 'DATABASE_URL="postgresql://user:pass@host:5432/db"' > .env.local

# Executar migrations
npm install
npx prisma generate
npx prisma db push

# Verificar tabelas
npx prisma studio
```

---

## 🔧 TROUBLESHOOTING ESPECÍFICO

### Erro: "Cannot find module '@prisma/client'"

**Causa:** Prisma não gerou o client no build

**Solução:**
1. Verifique se tem `postinstall` no package.json
2. Commit e push
3. Redeploy

```bash
git add package.json
git commit -m "fix: add prisma postinstall"
git push
```

### Erro: "Invalid `prisma.user.findUnique()`"

**Causa:** Tabelas não existem no banco

**Solução:**
Execute migrations localmente apontando para o banco de produção:

```bash
# .env.local
DATABASE_URL="postgresql://user:pass@host:5432/db"

# Executar
npx prisma db push
```

### Erro: "Can't reach database server"

**Causa:** Banco não está acessível

**Solução:**
1. Verifique se o banco está rodando no Coolify
2. Verifique firewall: `sudo ufw allow 5432/tcp`
3. Teste conexão: `telnet seu-ip 5432`
4. Verifique se a porta no DATABASE_URL está correta

### Erro: "Invalid connection string"

**Causa:** Formato errado da URL

**Solução:**
Formato correto:
```
postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=public
```

**Escape caracteres especiais:**
- `#` → `%23`
- `@` → `%40`
- `!` → `%21`

Exemplo:
```
Senha: Pass#123!
URL: postgresql://user:Pass%23123%21@host:5432/db
```

### Build passa mas página está em branco

**Causa:** Erro de runtime (JavaScript)

**Solução:**
1. Abra DevTools (F12)
2. Veja o Console
3. Veja erros de variáveis de ambiente

---

## 📋 CHECKLIST COMPLETO

Antes de abrir suporte, verifique:

- [ ] Variáveis de ambiente TODAS adicionadas na Vercel
- [ ] NEXTAUTH_SECRET tem pelo menos 32 caracteres
- [ ] DATABASE_URL está correta e acessível
- [ ] Banco PostgreSQL está rodando no Coolify
- [ ] Banco é acessível publicamente
- [ ] Firewall liberado (porta 5432)
- [ ] package.json tem `postinstall`
- [ ] Fez redeploy após adicionar variáveis
- [ ] Migrations executadas (`prisma db push`)
- [ ] Build logs não mostram erros
- [ ] Testou conexão ao banco localmente

---

## 🆘 AINDA NÃO FUNCIONA?

### Ver logs detalhados:

```bash
# Instalar Vercel CLI
npm i -g vercel

# Login
vercel login

# Ver logs em tempo real
vercel logs --follow
```

### Testar build localmente:

```bash
# Usar mesmas variáveis da Vercel
# Criar .env.local com TODAS as variáveis

# Build local
npm run build

# Se falhar localmente, vai falhar na Vercel
# Corrija os erros localmente primeiro
```

### Deploy manual:

```bash
# Se tudo funciona local mas não na Vercel
vercel --prod

# Isso força um novo deploy
```

---

## ✅ SOLUÇÃO RÁPIDA (90% DOS CASOS)

```bash
# 1. Adicione TODAS as variáveis na Vercel
# Principalmente:
DATABASE_URL=postgresql://...
NEXTAUTH_URL=https://seu-app.vercel.app
NEXTAUTH_SECRET=[32+ caracteres]

# 2. Redeploy
# Vá na Vercel → Deployments → ... → Redeploy

# 3. Execute migrations
npx prisma db push

# 4. Teste!
```

---

## 📞 ÚLTIMA OPÇÃO

Se nada funcionar, envie os logs:

1. Vá em: Deployments → Último deploy
2. Copie TODO o log de build
3. Procure por linhas em vermelho (erros)
4. Compartilhe os erros

**Logs importantes:**
- "Error:" ...
- "Failed to compile"
- "Module not found"
- "Type error"

---

## 💡 DICA PRO

Crie um arquivo `.env.production` localmente igual ao da Vercel:

```bash
# .env.production
DATABASE_URL="postgresql://..."
NEXTAUTH_URL="https://seu-app.vercel.app"
NEXTAUTH_SECRET="..."
# ... todas as outras
```

Teste com:
```bash
NODE_ENV=production npm run build
```

Se funcionar local, funcionará na Vercel!

---

✅ **Na maioria dos casos, é só adicionar as variáveis de ambiente e fazer redeploy!**
