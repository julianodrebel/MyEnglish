# Meu Inglês

App web para seguir um roteiro de estudo de inglês (B1 rumo ao C1) semana a semana: ver o que estudar hoje, com qual recurso e por quantos minutos, marcar os dias concluídos e acompanhar o progresso. Uso pessoal, mobile-first, pensado para abrir no celular na hora do estudo e continuar de onde parou em outro aparelho.

Segue o mesmo padrão dos apps de treino (repositório `gym`) e de finanças (repositório `account`): mesma stack, mesmas convenções, mesmo design e o mesmo projeto Supabase.

- **Tudo está em `index.html`**: HTML, CSS e JavaScript (ES module). Não tem build, bundler, `package.json` nem código de servidor.
- **Backend:** Supabase (Auth com e-mail/senha e Google, banco Postgres com RLS), acessado direto do navegador pelo `supabase-js` v2. **O projeto Supabase é compartilhado** com o gym e o account; as tabelas deste app ficam no schema **`english`**.
- **Sem gráfico:** este app não usa Chart.js. A única biblioteca é o `supabase-js`, do CDN jsDelivr. As fontes (Barlow e Barlow Condensed) vêm do Google Fonts.
- **Hospedagem:** Cloudflare Workers servindo arquivos estáticos (`wrangler.jsonc`), no domínio `english.julianodrebel.com.br`. O deploy é automático: o push para o GitHub dispara a esteira da Cloudflare. Nunca é feito da máquina local.
- O **design** está documentado em `DESIGN.md`. Leia antes de qualquer mudança visual.

## Comandos

```bash
npx serve .                 # rodar local (ou: python -m http.server 8000)
git switch develop          # base para qualquer trabalho novo
```

- Não existem testes automatizados, lint nem formatter. Para validar, sirva o app localmente e teste no navegador **em largura de celular (~375 px) e em desktop (≥ 960 px)**, com o console sem erros.
- Abrir o `index.html` direto pelo arquivo (`file://`) não funciona, porque o app usa ES modules.
- O login com Google só funciona se a URL estiver em **Supabase → Authentication → URL Configuration** (Redirect URLs). Para testar localmente, entrar com e-mail e senha é mais simples.

## Estrutura do `index.html`

O arquivo é grande por causa do conteúdo das semanas. **Não leia o arquivo inteiro**: localize a seção com busca (`grep -n`) e leia só o trecho necessário.

1. **`<head>`**: metas (`theme-color` igual a `--bg`) e fontes.
2. **`<style>`**: tokens em `:root`, depois blocos marcados com `/* ---------- nome ---------- */` (campos e botões, login, estrutura do app, hoje, roteiro, progresso) e, no fim, `@media (min-width: 960px)` com todo o layout de desktop.
3. **Markup**: `#tela-login` e `#tela-app` (com `nav.tabbar` e `.conteudo`, onde cada aba é uma `section.aba#aba-<nome>`: `hoje`, `roteiro`, `vocabulario`, `progresso`).
4. **`<script type="module">`**: blocos marcados com `// ---------- nome ----------`, nesta ordem: configuração do Supabase, utilidades, conteúdo do roteiro, autenticação, abas, progresso (banco), onde parei (banco), hoje, roteiro, vocabulário e progresso.

Código novo vai no bloco correspondente. Se for um assunto novo, crie um bloco com o mesmo estilo de marcador, na posição que faça sentido.

## Conteúdo do roteiro

O conteúdo é fixo e fica no código, no bloco `// ---------- conteúdo do roteiro ----------`. O banco guarda só o progresso.

- `SEMANAS`: uma entrada por semana. Chaves: `t` (tema), `s` (subtítulo), `goal` (meta), `gram` (`t` título, `x` explicação, `e` exemplos), `pron` (dicção), `micro` (micro-hábito), `v` (6 grupos de vocabulário, `[nome, 'termo|tradução;…']`, um por dia de segunda a sábado), `res` (`[nome, tipo, como usar]`), `cont` (conteúdos para o barril), `d` (bloco de 20 min de segunda a sábado, `[o que fazer, recurso]`) e `check` (teste de domingo).
- `BLOCOS` (nome do bloco de cada dia da semana), `DIAS_ROTEIRO` (Seg…Dom) e `TEMPOS_MORTOS` (input passivo de cada dia).
- `passosDoDia(semana, dia)` monta os 30 minutos do dia a partir disso. Domingo é sempre revisão e curadoria.
- **Para adicionar semanas**, acrescente entradas em `SEMANAS` no mesmo formato. O total de dias, as porcentagens, os chips e as listas se ajustam sozinhos. O banco aceita semanas de 1 a 104.
- Em `v`, não use `;` nem `|` dentro de um termo ou tradução: são os separadores.
- Os links de recurso abrem uma busca (YouTube para o tipo `yt`, Google para os outros), não uma página específica.

## Convenções de código

- **Nomes em português**: variáveis, funções, IDs, classes CSS e comentários (`carregarProgresso`, `#lista-semanas`, `.semana-card`). As colunas do banco são em inglês (`week`, `day`), como no script do schema.
- Estilo: 2 espaços, aspas simples, ponto e vírgula, `const` por padrão, arrow functions e template strings. O bloco de conteúdo é a exceção: usa aspas duplas e crases porque o texto em inglês tem apóstrofos.
- `$` e `$$` são atalhos de `querySelector` e `querySelectorAll`.
- O estado fica em variáveis `let` no escopo do módulo (`feitos`, `datas`, `diaVisto`, `semanaAberta`, `filtroVocab`). Não há framework nem store.
- **`carregarX()`** busca do Supabase e atualiza o estado. **`desenharX()`** renderiza a partir do estado. `desenharTudo()` redesenha Hoje, Roteiro e Progresso depois de qualquer mudança no progresso.
- A renderização é feita com template string e `innerHTML`. **Todo texto passa por `esc()`**, inclusive o do conteúdo fixo.
- Eventos de listas dinâmicas usam delegação: um listener no container, `e.target.closest('.classe')` e `data-*` no botão.
- Chamadas ao Supabase seguem este padrão (o client já aponta para o schema `english`):
  ```js
  const { data, error } = await supabase.from('tabela').select('...');
  if (error) return aviso(error.message, true);
  ```
- Para feedback ao usuário use `aviso(msg)` (sucesso) ou `aviso(msg, true)` (erro). Não use `alert()`.
- Semana começa em **1** e dia vai de **0 (segunda) a 6 (domingo)**, no código e no banco. Use `chave()`, `posicao()` e `dePosicao()`.
- Datas: `hoje()`, `isoLocal()`, `fmtDia()`. Não use `toISOString()` para obter um dia, porque ele usa UTC e muda o dia à noite no Brasil.
- **Não adicione dependências, frameworks, etapas de build nem divida o arquivo** sem combinar antes.
- O texto da interface é em **português do Brasil**; termos e exemplos em inglês levam `lang="en"`.

## Banco de dados (Supabase)

Schema **`english`**, no mesmo projeto do gym e do account. O client é criado com `{ db: { schema: 'english' } }`.

| Tabela | Conteúdo |
| --- | --- |
| `day_progress` | Uma linha por dia concluído (`week`, `day`, `completed_at`). Marcar = insert, desmarcar = delete. Chave: `user_id` + `week` + `day` |
| `user_state` | Uma linha por usuário: `last_week` e `last_day` = dia aberto na aba Hoje, para continuar em outro aparelho. `started_on` existe e ainda não é usado |

- **Toda tabela nova precisa, no mesmo script:** `GRANT` para `authenticated` (e `service_role`), RLS e policy. **Nunca dê acesso ao papel `anon`.**
- `user_id` tem default `auth.uid()`. **Não envie `user_id` nos inserts.**
- `day_progress` não tem permissão de `UPDATE` de propósito: o app só insere e apaga.
- A chave em `index.html` é a *publishable* (pública por design). **Nunca coloque a chave `service_role` ou secret no código.**
- Em *Exposed schemas* do Supabase, `english` fica **junto** com `public` e `financas`. Tirar um deles derruba o app correspondente.
- **Ao mudar o schema:** atualize o script no `README.md`, adicione o SQL de migração em *Upgrading an existing database* dentro de `begin; … commit;` e lembre o usuário de rodar o SQL no Supabase **antes** de publicar o novo `index.html`.

### Decisões

- **Schema próprio (`english`)**, como o `financas`, para todos os sistemas pessoais ficarem no mesmo projeto Supabase sem se misturar.
- **Sem "Criar conta"** no login: as contas são criadas pelo painel do Supabase. O usuário é o mesmo do gym e do account.
- **Conteúdo no código, progresso no banco.** O roteiro muda pouco e não precisa de tela de edição.
- **Dia concluído = linha em `day_progress`.** Uma linha por dia (e não um JSON único) evita que dois aparelhos sobrescrevam o progresso um do outro.
- **A tela muda antes de o banco responder** ao marcar um dia; se o banco recusar, volta ao que era e mostra o aviso.
- **Onde parei:** a aba Hoje abre no dia salvo em `user_state`; sem nada salvo, no primeiro dia não concluído. Concluir um dia leva ao próximo pendente. As setas navegam por todos os dias e o botão "Ir para onde parei" volta ao primeiro pendente.
- **Os dias do roteiro não são presos ao calendário.** "Segunda" é o primeiro dia da semana do roteiro, mesmo que seja feito numa quinta.
- **Sequência (aba Progresso)** conta os dias de calendário seguidos com pelo menos um dia concluído, terminando hoje ou ontem, pela data de `completed_at`.
- **Sincronização entre aparelhos:** o progresso é buscado de novo quando a aba do navegador volta a ficar visível. Não usa Realtime (a tabela está na publicação, se um dia for preciso).
- **Vocabulário:** a busca procura em todas as semanas e ignora acentos e maiúsculas; sem busca, vale o chip da semana.

## Git e fluxo de trabalho

- `master` é produção. `develop` é integração (e é a branch padrão no GitHub).
- Trabalho novo sai de `develop` numa branch de feature e volta por pull request para `develop`.
- Release: merge de `develop` em `master` com a versão como mensagem (`1.0.0`, `1.0.1`…).
- **Push é deploy.** Cada push para o GitHub dispara a esteira da Cloudflare. Mudanças de schema precisam estar aplicadas no Supabase antes do push.
- Mensagens de commit em **inglês**, no formato Conventional Commits (`feat: …`, `fix: …`, `refactor: …`, `docs: …`).
- **Nunca commite `client_secret*.json`** (está no `.gitignore`).
- Só faça commit, push ou merge quando o usuário pedir.

## Documentação

- `README.md` é em **inglês**. Ao criar ou mudar uma funcionalidade, atualize a lista *Features*.
- `DESIGN.md` é a referência visual. Se uma mudança criar um padrão novo (token, componente, regra), registre lá.
- Decisões importantes tomadas durante uma tarefa devem ser registradas neste arquivo.

## Pontos de atenção

- `onAuthStateChange` chama `mostrarTela` dentro de `setTimeout`, como a documentação do Supabase recomenda. Não remova.
- O botão *Roteiro* da navegação sempre volta para a lista de semanas (`semanaAberta = null`).
- O avatar e o menu da conta ficam só no topo da aba Hoje.
- O diretório de assets é a raiz do repositório. O `.assetsignore` deixa só o `index.html` ir para o site; arquivo novo que não deva ser público precisa estar coberto por ele.
