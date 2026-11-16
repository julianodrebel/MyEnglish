# Meu Inglês

Personal web app to follow a week-by-week English study roadmap (B1 towards C1): see what to study today, with which resource and for how long, tick off finished days and track progress across devices.

Single `index.html` (HTML, CSS and an ES module), no build step. Backend is Supabase (Auth + Postgres with RLS), shared with the `gym` and `account` apps; this app's tables live in the `english` schema. Hosted on Cloudflare Workers static assets at `english.julianodrebel.com.br`.

## Features

- **Today**: opens on the day you left off; shows the 30-minute plan step by step (what, which resource, how many minutes), passive-listening suggestion, the week's micro-habit and the 12 words of the day. "Concluir dia" marks it done and moves to the next pending day. Arrows browse any day.
- **Roadmap**: 12 themed weeks with progress per week. A week shows its goal, the 7 days (tick or open each), grammar topic, pronunciation focus, recommended resources and content ideas.
- **Vocabulary**: 72 items per week (864 in total) with Portuguese gloss, filter by week and accent-insensitive search in both languages.
- **Progress**: overall percentage, days done, current streak and a bar per week.
- **Sync**: progress and the last opened day are stored in Supabase, so phone, tablet and desktop stay in step. Progress is refetched when the tab becomes visible again.
- Sign in with e-mail/password or Google. There is no sign-up screen; accounts are created in the Supabase dashboard.

## Running locally

```bash
npx serve .        # or: python -m http.server 8000
```

Opening the file via `file://` does not work (ES modules).

## Supabase setup

1. Run the script below in the SQL Editor (it is idempotent).
2. **Project Settings → API → Exposed schemas**: add `english` next to the existing ones.
3. **Authentication → URL Configuration → Redirect URLs**: add `https://english.julianodrebel.com.br` (and the local URL if you want Google sign-in locally). Do not change the Site URL.

```sql
-- =====================================================================
-- Roteiro de Inglês · backend Supabase em schema próprio: english
-- Rode este arquivo inteiro no SQL Editor do seu projeto.
-- Pode ser executado mais de uma vez sem erro (idempotente).
--
-- Depois de rodar: Project Settings > API (Data API) > Exposed schemas
-- e adicione "english", como você fez com os schemas dos outros apps.
--
-- Modelo:
--   english.day_progress  1 linha = 1 dia concluído (marcar = insert,
--                         desmarcar = delete). Chave: usuário+semana+dia.
--   english.user_state    1 linha por usuário: onde parou (última semana
--                         e dia abertos), para retomar em outro aparelho.
-- =====================================================================

-- ---------- 0. Schema ----------
create schema if not exists english;
comment on schema english is 'Roteiro de Inglês (app pessoal de estudo).';

-- ---------- 1. Progresso por dia ----------
create table if not exists english.day_progress (
  user_id      uuid        not null default auth.uid()
                           references auth.users (id) on delete cascade,
  week         smallint    not null check (week between 1 and 104),
  day          smallint    not null check (day between 0 and 6),  -- 0 = segunda ... 6 = domingo
  completed_at timestamptz not null default now(),
  primary key (user_id, week, day)
);

comment on table  english.day_progress is 'Dias concluídos. Linha existe = dia feito.';
comment on column english.day_progress.week is 'Número da semana do roteiro, começando em 1.';
comment on column english.day_progress.day  is '0 = segunda, 1 = terça, ... 6 = domingo.';

-- ---------- 2. Onde parei ----------
create table if not exists english.user_state (
  user_id    uuid        primary key default auth.uid()
                         references auth.users (id) on delete cascade,
  last_week  smallint    not null default 1 check (last_week between 1 and 104),
  last_day   smallint    check (last_day between 0 and 6),
  started_on date,                                   -- data em que começou a semana 1 (opcional)
  updated_at timestamptz not null default now()
);

comment on table english.user_state is 'Última semana/dia abertos, para retomar em outro aparelho.';

create or replace function english.touch_updated_at()
returns trigger
language plpgsql
set search_path = ''
as $$
begin
  new.updated_at := now();
  return new;
end;
$$;

drop trigger if exists user_state_touch on english.user_state;
create trigger user_state_touch
  before update on english.user_state
  for each row execute function english.touch_updated_at();

-- ---------- 3. Permissões do schema ----------
-- Usuário logado usa o schema; visitante anônimo não tem acesso nenhum.
revoke all on schema english from anon;
revoke all on all tables in schema english from anon;

grant usage on schema english to authenticated, service_role;
grant select, insert, delete         on english.day_progress to authenticated;
grant select, insert, update, delete on english.user_state   to authenticated;
grant all on all tables in schema english to service_role;

-- ---------- 4. RLS: cada usuário só enxerga as próprias linhas ----------
alter table english.day_progress enable row level security;
alter table english.user_state   enable row level security;

drop policy if exists "progress_select_own" on english.day_progress;
drop policy if exists "progress_insert_own" on english.day_progress;
drop policy if exists "progress_delete_own" on english.day_progress;

create policy "progress_select_own" on english.day_progress
  for select to authenticated using ((select auth.uid()) = user_id);
create policy "progress_insert_own" on english.day_progress
  for insert to authenticated with check ((select auth.uid()) = user_id);
create policy "progress_delete_own" on english.day_progress
  for delete to authenticated using ((select auth.uid()) = user_id);

drop policy if exists "state_select_own" on english.user_state;
drop policy if exists "state_insert_own" on english.user_state;
drop policy if exists "state_update_own" on english.user_state;
drop policy if exists "state_delete_own" on english.user_state;

create policy "state_select_own" on english.user_state
  for select to authenticated using ((select auth.uid()) = user_id);
create policy "state_insert_own" on english.user_state
  for insert to authenticated with check ((select auth.uid()) = user_id);
create policy "state_update_own" on english.user_state
  for update to authenticated
  using ((select auth.uid()) = user_id) with check ((select auth.uid()) = user_id);
create policy "state_delete_own" on english.user_state
  for delete to authenticated using ((select auth.uid()) = user_id);

-- ---------- 5. Realtime (opcional): outro aparelho aberto atualiza na hora ----------
do $$
begin
  if exists (select 1 from pg_publication where pubname = 'supabase_realtime')
     and not exists (
       select 1 from pg_publication_tables
       where pubname = 'supabase_realtime'
         and schemaname = 'english' and tablename = 'day_progress')
  then
    alter publication supabase_realtime add table english.day_progress;
  end if;
end $$;

-- =====================================================================
-- 6. Conferência (deve listar 2 tabelas com rls = true e 7 políticas)
-- =====================================================================
select c.relname as tabela, c.relrowsecurity as rls
from pg_class c join pg_namespace n on n.oid = c.relnamespace
where n.nspname = 'english' and c.relkind = 'r';

select tablename, policyname, cmd
from pg_policies
where schemaname = 'english'
order by tablename, cmd;
```

## Upgrading an existing database

No migrations yet.

## Deploy

Pushing to GitHub triggers the Cloudflare pipeline (`master` is production, other branches get previews). `wrangler.jsonc` maps the custom domain; `.assetsignore` keeps everything but `index.html` out of the site.
