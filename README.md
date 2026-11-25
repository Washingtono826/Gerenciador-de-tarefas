# WorkOS Collab — Gestão de Projetos, Tarefas e Equipes em Nuvem

Aplicação web colaborativa inspirada no **Monday.com (Work OS)**, construída com **React + Vite + TypeScript + Tailwind CSS**, **Supabase** (Auth, PostgreSQL, Realtime, RLS e Edge Functions), **SheetJS (`xlsx`)**, **Gmail SMTP** e **Google Drive API v3**.

---

## Pré-requisitos

- Node.js 20+ e npm
- Conta no [Supabase](https://supabase.com)
- Conta Google (para Senha de App Gmail e Google Cloud Console)

---

## 1. Como executar o `schema.sql` no Supabase

1. Crie um novo projeto no painel do Supabase.
2. No menu lateral, acesse **SQL Editor** > **New query**.
3. Copie todo o conteúdo de [`supabase/schema.sql`](./supabase/schema.sql) e clique em **Run**.
   - Isso criará todas as tabelas (`profiles`, `projects`, `groups`, `tasks`, `subtasks`, `comments`, `task_attachments`, `integration_settings`, `integration_logs`), políticas **RLS**, publicação **Realtime**, a função `claim_task_recurrence` e a trigger de primeiro acesso (`admin` automático para o 1º usuário).
4. Em **Project Settings > API**, copie a **Project URL** e a chave **anon public** para o seu `.env` (a partir do `.env.example`):

```bash
cp .env.example .env
```

---

## 2. Como configurar a Senha de App no Gmail (Alertas 1 dia antes)

> **Regra importante:** Tarefas com recorrência **Diária (`daily`)** não disparam e-mails. Apenas tarefas avulsas e recorrências maiores (`weekly`, `biweekly`, `monthly`, `bimonthly`, `semiannual`) notificam o responsável 1 dia antes do prazo (fuso `America/Sao_Paulo`).

1. Acesse sua Conta Google > **Segurança** e ative a **Verificação em duas etapas**.
2. Em **Senhas de app** (ou acesse `myaccount.google.com/apppasswords`), crie uma nova senha de app para "WorkOS Collab" e copie os 16 caracteres gerados.
3. Configure os **Secrets** das Edge Functions no Supabase (via Dashboard em **Edge Functions > Secrets** ou Supabase CLI):

```bash
supabase secrets set \
  GMAIL_USER="seu-email@gmail.com" \
  GMAIL_APP_PASSWORD="xxxx xxxx xxxx xxxx" \
  APP_PUBLIC_URL="https://seu-dominio-ou-localhost.com"
```

4. Faça o deploy das Edge Functions no seu projeto Supabase:

```bash
supabase functions deploy notify-due-tasks
supabase functions deploy gdrive-sync
supabase functions deploy admin-users
```

5. *(Opcional — Agendamento diário)*: Em **Database > Extensions**, ative `pg_cron` e `pg_net`, depois descomente e execute os comandos `cron.schedule(...)` no final de [`supabase/schema.sql`](./supabase/schema.sql).

---

## 3. Como configurar a Google Drive API no Google Cloud Console

1. Acesse o [Google Cloud Console](https://console.cloud.google.com/) e crie/selecione um projeto.
2. Em **APIs e Serviços > Biblioteca**, ative:
   - **Google Drive API**
   - **Google Picker API**
3. Configure a **Tela de permissão OAuth** (adicione seu e-mail em usuários de teste se o app estiver em modo Testing) com os escopos:
   - `https://www.googleapis.com/auth/drive`
4. Em **APIs e Serviços > Credenciais**:
   - Crie um **ID do cliente OAuth** (Tipo: *Aplicativo da Web*).
   - Em *Origens JavaScript autorizadas*, adicione `http://localhost:5173` (e a URL de produção).
   - Copie o **Client ID** para `VITE_GOOGLE_CLIENT_ID` no `.env`.
   - Crie também uma **Chave de API (API Key)** e coloque em `VITE_GOOGLE_API_KEY` no `.env` (necessária para o Google Picker).
5. *(Opcional — Backup automático via `pg_cron`)*: Gere um `refresh_token` offline no OAuth Playground e defina nos Secrets do Supabase:

```bash
supabase secrets set \
  GOOGLE_CLIENT_ID="seu-client-id.apps.googleusercontent.com" \
  GOOGLE_CLIENT_SECRET="seu-client-secret" \
  GOOGLE_DRIVE_REFRESH_TOKEN="seu-refresh-token"
```

---

## Execução Local

```bash
npm install
npm run dev
```

Para validar tipos TypeScript e gerar o bundle de produção:

```bash
npm run build
```
