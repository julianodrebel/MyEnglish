# Design — Meu Inglês

Referência visual do app. O design é o mesmo dos apps de treino (`gym`) e de finanças (`account`): mesmos tokens, fontes e componentes, para os três parecerem da mesma família. Todo o CSS está no `<style>` de `index.html`, e os tokens ficam em `:root`. Qualquer mudança de layout deve seguir este documento. Se surgir um padrão novo, atualize-o aqui.

## Princípios

1. **Feito para a hora do estudo.** Abrir, ver o que fazer hoje, fazer e marcar. Poucas decisões por tela.
2. **Mobile-first.** O layout base é o do celular. O desktop é uma adaptação dentro de um único breakpoint (`min-width: 960px`).
3. **Tema escuro, sempre.** Não existe tema claro (`color-scheme: dark`). Não crie variações claras.
4. **Uma ação principal por tela.** Ela usa o laranja de destaque ("Concluir dia"). O resto é neutro.
5. **Os números são os protagonistas.** Minutos, contagens (`3/7`) e porcentagens usam a fonte condensada e algarismos tabulares.

## Cores

Use sempre as variáveis. **Não escreva hex novo no CSS.** Se faltar uma cor, crie um token em `:root`. Os tokens são os mesmos do `account`:

| Grupo | Tokens |
| --- | --- |
| Superfícies | `--bg` `#0E0F11`, `--superficie` `#17191C`, `--superficie-2` `#202328`, `--nav` `#141619`, `--lateral` `#111316` |
| Bordas | `--borda-suave` `#22252A`, `--borda` `#2A2E34`, `--borda-forte` `#3A3F47`, `--lateral-borda` `#1E2125` |
| Texto | `--texto` `#F2F3F5`, `--texto-2` `#C9CDD3`, `--suave` `#9BA1AA`, `--apagado` `#6B717A` |
| Destaque | `--destaque` `#FF6A3D`, `--destaque-hover` `#FF7D55`, `--destaque-tinta` `#1B0D06`, `--destaque-fundo` `#2A1A13` |
| Semânticas | `--ok` `#4CC38A`, `--ok-fundo`, `--desce` `#FF8A63`, `--erro` `#E5484D` |

**Regra:** laranja indica o que fazer agora (o dia atual, a semana em andamento, o botão de concluir), verde indica o que já foi feito (dia marcado, semana completa). Não troque esses papéis.

## Tipografia

| Token | Fonte | Uso |
| --- | --- | --- |
| `--fonte` | Barlow 400–700 | Texto corrido, labels, botões comuns |
| `--fonte-titulo` | Barlow Condensed 500–700 | Títulos, botão primário, **todos os números** |

A base é `16px / 1.45`. **Inputs nunca ficam abaixo de 16px**, para o iOS não dar zoom.

| Elemento | Tamanho | Fonte / peso |
| --- | --- | --- |
| Título de página (`.pagina-topo h1`) e dia no seletor (`.seletor-dia h1`) | 34px (`line-height: 1`) | Condensada 700 |
| Título da semana aberta (`.topo-semana h1`) | 30px | Condensada 700 |
| Título de card (`.dia-cartao h2`, `.semana-card h2`) | 26px | Condensada 700 |
| Título de bloco (`.bloco h3`) | 22px | Condensada 700 |
| Valor de estatística (`.stat b`) | 30px | Condensada 700 |
| Minutos do passo (`.passo-min`) | 18px | Condensada 700 |
| Botão primário | 20px (22px no `.grande`) | Condensada 700 |
| Texto padrão | 16px (15px em textos longos dentro de cards) | Barlow 400 |
| Sobretítulo (`.sobretitulo`) | 13px, MAIÚSCULAS, `letter-spacing: .08em` | Barlow 700, `--suave` |
| Legenda / tradução / recurso | 13–14px | Barlow, `--suave` |

## Espaçamento, raios e tamanhos

- **Espaçamento:** escala `4 · 8 · 12 · 16 · 20 · 24`. Margem lateral de **16px** no celular e **40px** no desktop.
- **Raios:** `--raio-p` 12px (inputs, itens da barra lateral), 14px (botões), `--raio` 16px (estatísticas, botão largo), 20px (`.cartao`), metade da altura (chips, selo, avatar, círculo de marcar).
- **Alvos de toque:** 44px (ícones, setas, avatar, círculo de marcar), 48px (inputs e botões), 56px (botão primário grande, botão largo, linhas de dia). Não crie alvos menores que 44px.

## Layout

| | Celular (< 960px) | Desktop (≥ 960px) |
| --- | --- | --- |
| Navegação | `.tabbar` fixa embaixo, 72px + safe area, 4 abas (*Hoje*, *Roteiro*, *Vocabulário*, *Progresso*) | Barra lateral de 248px com marca e as 4 abas |
| Conteúdo | Coluna de até 640px, centralizada | Até 1120px, alinhado à esquerda |
| Hoje | Seletor de dia e avatar no topo; card do dia; passos; palavras; botão de concluir | Passos à esquerda e palavras à direita (`.hoje-layout`) |
| Roteiro | Lista de semanas em uma coluna; a semana aberta substitui a lista | Grade de cards (mínimo 340px); semana aberta em duas colunas (`.semana-layout`) |
| Vocabulário | Busca e chips roláveis; grupos em uma coluna | Chips quebram linha; grupos em duas colunas |
| Progresso | 4 estatísticas em 2×2 e barras por semana | 4 estatísticas numa linha |

- Tudo que fica fixo embaixo se posiciona acima de `--altura-nav` + `--seguro-baixo`.
- **Não crie outros breakpoints** sem necessidade clara.

## Componentes

Reutilize estes componentes antes de criar um novo. Os que vêm do `gym`/`account` têm o mesmo CSS de lá.

| Classe | O que é | Observações |
| --- | --- | --- |
| `.botao` / `.botao.primario` / `.grande` | Botões base e ação principal | Só um primário por tela |
| `.botao.largo` | Botão comum de largura total, 56px | "Desmarcar dia", "Ir para onde parei", "Ver as palavras da semana" |
| `.icone` | Botão só com ícone, redondo | **Sempre com `aria-label`** |
| `.cartao`, `.sobretitulo`, `.pagina-topo`, `.dica`, `.vazio` | Base de superfície e texto | |
| `.grupo-titulo` | Sobretítulo de seção com um texto à direita | "Palavras de hoje · Standup" |
| `.conta` / `.avatar` / `.menu-conta` | Menu da conta: e-mail e "Sair" | `aria-expanded` no avatar |
| `.topo-dia` / `.seletor-dia` | Cabeçalho da aba Hoje: setas ‹ › com o dia no meio (semana em sobretítulo) e o avatar | Setas desabilitadas no primeiro e no último dia |
| `.dia-cartao` | Card do dia: tema da semana, selo de minutos, nome do bloco e meta | `.feito-em` (verde, com check) quando concluído |
| `.selo` / `.selo.feita` | Pílula com número (`30 min`, `2/7`) | Verde quando a semana está completa |
| `.passos` / `.passo` / `.passo-min` | Lista de passos: minutos à esquerda, título, texto e recurso | `.passivo` tem a caixa de minutos tracejada (tempos mortos, micro-hábito) |
| `.vocab` | Lista de palavras: termo em negrito (`lang="en"`) e tradução embaixo | |
| `.semana-card` | Card de semana tocável: sobretítulo, selo, tema, subtítulo e barra | Borda laranja e "· atual" na semana em andamento |
| `.topo-semana` | Cabeçalho da semana aberta, com botão de voltar | |
| `.dias` / `.marcar` / `.dia-abrir` | Dias da semana: círculo de marcar e linha que abre o dia na aba Hoje | `aria-pressed` no círculo; `aria-current="step"` no dia atual (laranja) |
| `.bloco` / `.exemplos` / `.lista-texto` | Card de texto (gramática, recursos, conteúdos) | Exemplos sobre `--superficie-2` |
| `.trilho` | Barra de progresso fina | Laranja em andamento, `.dentro` (verde) quando completa |
| `.barras-cat` / `.linha-cat` | Lista com nome, contagem e barra | Progresso por semana |
| `.resumo` / `.stat` | Estatísticas (rótulo + valor grande) | 2 colunas no celular, 4 no desktop |
| `.chip` / `.chips.rolavel` | Filtro de semana no vocabulário | `aria-pressed` |
| `.busca` | Input com ícone de lupa | `type="search"` + `aria-label` |
| `.concluida` | Tela de roteiro concluído, no lugar do dia | Selo verde de 72px com check de 32px |
| `#aviso` | Toast (verde para sucesso, `.erro` para vermelho) | Chamado via `aviso()`; some em 3,5s |

## Ícones

- SVG inline, `viewBox="0 0 24 24"`, `fill="none"`, `stroke="currentColor"`, `stroke-linecap/linejoin="round"`, `stroke-width` entre 2 e 3, sempre com `aria-hidden="true"`.
- A marca é um balão de fala com duas linhas, no mesmo quadrado laranja do gym e do account.
- No JS, use o objeto `ICONE` e o helper `svg()`. Não use biblioteca de ícones nem emoji como ícone.

## Movimento e acessibilidade

- O aviso aparece em `.25s`. Sem animação decorativa. `prefers-reduced-motion` desliga tudo.
- Foco visível: `:focus-visible` com contorno laranja de 2px. Não remova outlines.
- Estado sempre via ARIA: `aria-current="page"` (aba), `aria-current="step"` (dia atual), `aria-pressed` (chips e círculo de marcar), `aria-expanded` (menu).
- Cor nunca é o único sinal: dia feito também tem o check, semana atual também diz "atual", semana completa também mostra `7/7`.

## Texto da interface

- Português do Brasil, tom direto e informal. Termos em inglês só no conteúdo de estudo.
- Botões começam com verbo e são curtos: "Concluir dia", "Desmarcar dia", "Ir para onde parei".
- Títulos e botões usam só a primeira letra maiúscula.
- Avisos de sucesso são curtos ("Dia concluído!"). Erros começam com "Não foi possível …".

## Checklist para mudanças visuais

- [ ] Usei tokens e componentes existentes (ou criei token novo em `:root`)
- [ ] Testei em ~375px e em ≥ 960px
- [ ] Alvos de toque com 44px ou mais, inputs com 16px ou mais
- [ ] Elementos fixos respeitam a barra inferior e a safe area
- [ ] Estados (atual, feito, desabilitado, vazio, erro) estão cobertos
- [ ] ARIA e `aria-label` corretos, foco visível
- [ ] Atualizei este arquivo se criei um padrão novo
