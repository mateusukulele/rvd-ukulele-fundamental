# HANDOFF: hero novo, só na página de TESTE

Escrito em 14/09/2026. Escopo: **um arquivo**, a página de teste. As duas de produção não
mudam nem por engano.

```
NÃO TOCAR   clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-op.html
            https://cursos.comotocarukulele.com/clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-op
NÃO TOCAR   clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl.html
            https://cursos.comotocarukulele.com/clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl
ALVO        clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html
            https://cursos.comotocarukulele.com/clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste
```

A página de teste é uma cópia da `-cl` (fechada, com gate) mais o hero novo. Tudo abaixo do
hero fica idêntico à produção. Nada de performance, gate, dobra, celular deitado ou fonte
embutida muda de mecanismo; só o texto do hero e as fontes que vestem esse texto.

## Assunções (mate uma antes de rodar se estiver errada)

1. **Caixa do h1: sentence case**, como o h1 de hoje. A frase veio em maiúsculas na mensagem,
   mas o `.hero h1` não tem `text-transform`, e a headline atual é em minúsculas. Fica
   `E se 20 minutinhos do seu dia virassem o momento mais leve da sua rotina?`.
2. **No celular o hero vira só h1 + subheadline.** A `.hero-strip` sai (é o "Nessa aula você
   vai conhecer os 3 passos…", que a sub nova já cobre) e o `.hero-nudge` some no celular em
   qualquer estado do gate, porque a sub termina em "Dá o play para começar" e a frase do
   nudge seria a terceira chamada para o play. No **desktop** o nudge fica como está, com a
   seta apontando para o vídeo ao lado: é o "só no web" que se mantém.
3. **Destaque = marca-texto coral atrás da metade de baixo das palavras**, texto continua
   índigo. Não é fundo cheio: índigo sobre coral cheio dá 4,3:1 de contraste e não passa no
   AA da `marca-cdu`. Com a faixa só na metade inferior, a leitura fica no creme.

## O texto

```
h1   E se {20 minutinhos} do seu dia virassem o {momento mais leve} da sua rotina?
sub  Com apenas 3 passos para tocar suas músicas favoritas, mesmo se você nunca tocou nada antes. Dá o play para começar.
```

As chaves viram `<mark>`.

## Passo a passo

### A. Recriar a página de teste a partir da fechada de hoje

```bash
cd "/Users/mateusaugusto/Claude Code/rvd-ukulele-fundamental"
cp clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl.html clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html
```

No arquivo `-cl-teste` recém-copiado, duas trocas de linha inteira (cada uma existe uma vez):

- `<meta name="robots" content="index, follow, max-image-preview:large">`
  vira `<meta name="robots" content="noindex, nofollow"><!-- pagina de teste -->`
- No comentário do gate, `/* GATE da VSL: abre` vira `/* GATE da VSL, versao de TESTE: abre`

### B. Trocar o hero (só no `-cl-teste`)

Âncoras exatas, todas existem uma vez no arquivo:

1. Trocar a linha do h1:
   ```html
   <h1>Toque <span>suas músicas favoritas</span> no Ukulele, com 20 minutinhos por dia</h1>
   ```
   por
   ```html
   <h1>E se <mark>20 minutinhos</mark> do seu dia virassem o <mark>momento mais leve</mark> da sua rotina?</h1>
   ```
2. Trocar a linha da subheadline (começa com `<p class="hero-sub">` e termina em `brasileiros</p>`) por
   ```html
   <p class="hero-sub">Com apenas 3 passos para tocar suas músicas favoritas, mesmo se você nunca tocou nada antes. Dá o play para começar.</p>
   ```
3. Apagar a linha inteira da `<p class="hero-strip">…</p>`.
4. O `<p class="hero-nudge">…</p>` **fica** (é o do desktop).

### C. CSS (só no `-cl-teste`)

1. Logo depois da regra `.hero h1 span{color:var(--coral)}` (linha única), acrescentar:
   ```css
   /* Marca-texto coral atras da metade de baixo da palavra. O texto continua indigo:
      indigo sobre coral cheio da 4,3:1 e nao passa no AA. clone e para a faixa
      acompanhar a palavra quando ela quebra de linha. */
   .hero h1 mark{background:linear-gradient(transparent 56%,var(--coral) 56%,var(--coral) 94%,transparent 94%);color:inherit;padding:0 .06em;-webkit-box-decoration-break:clone;box-decoration-break:clone}
   ```
2. Esconder o nudge no celular em qualquer estado. Existe um bloco `@media (max-width:899px){`
   do gate onde está `body.locked .hero-nudge{display:inline-flex}` ou equivalente: apagar
   essa linha, e garantir que dentro desse mesmo `@media (max-width:899px)` exista
   `.hero-nudge{display:none}`. Se já existir, não duplicar. Conferir com
   `grep -n "hero-nudge" arquivo`: no celular só pode sobrar `display:none`; as regras do
   `@media (min-width:900px)` e do bloco de celular deitado ficam como estão.
3. As regras `.hero-strip` no CSS podem ficar (não fazem mal sem o elemento). Se quiser
   limpar, apagar as três (`.hero-strip{…}`, `.hero-strip span{…}`, e as duas dentro dos
   `@media`); não é obrigatório.

### D. Regerar as fontes embutidas (OBRIGATÓRIO, senão o CLS do h1 volta)

As três fontes da dobra vêm embutidas no HTML com `unicode-range` listando **só os
caracteres do texto que vestem**. Mudou o texto, caractere novo cai na fonte de sistema e o
h1 troca de caixa quando a fonte completa chega.

```bash
/opt/homebrew/opt/python@3.14/bin/python3.14 .claude/perf/regera-subsets.py clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html
```

Ele lê h1, `.hero-sub` e `.hero-nudge` do próprio arquivo, regera os três subsets e imprime
os caracteres de cada um. Tem que aparecer `?` e `E` no da Montserrat 900 e `D`, `C`, `3`
e `.` no da Poppins 400. Exige fontTools (já instalado no Python do homebrew).

Descoberto ao preparar este handoff: a fechada em produção está com os subsets **de antes**
da última troca de texto (`ffc0969`); faltam `,` e `U` no h1 e outros no sub e no nudge.
Não mexer agora, mas é uma pendência para quando a produção receber o hero novo.

### E. Conferir antes do push (preview `rvd-repo`, porta 8129)

```bash
f=clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html
grep -c "noindex" $f                       # 1
grep -c "versao de TESTE" $f               # 1
grep -c "<mark>" $f                        # 2
grep -c "hero-strip\"" $f                  # 0
grep -c "data:font/woff2" $f               # 3
grep -c "el.onTime(" $f                    # 1
grep -o "rvd-clube-do-ukulele-rvd4v1-vert-cl[^\"']*" $f | sort -u   # só -cl-teste nas URLs canônicas e og
```

No navegador, `http://localhost:8129/clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html?reset=1`:

- `.claude/perf/medir-responsivo.py` apontado para a URL local: retrato 390x844 e 360x640
  com vídeo `0.5625` e "cabe"; deitado 844x390 em 2 colunas.
- Em 390x844: `document.querySelectorAll('.hero-top > *').length` = 3 (h1, sub, nudge) e
  `getComputedStyle(document.querySelector('.hero-nudge')).display` = `none`, com a página
  travada e com `?unlock=1`. Em 1440x900 o nudge tem que aparecer com a seta.
- Fontes: `document.fonts.check('900 24px Montserrat')` true; e para cada caractere do h1,
  `[...'E se 20 minutinhos do seu dia virassem o momento mais leve da sua rotina?'].every(ch => document.fonts.check('900 24px Montserrat', ch))` true.
- `<mark>`: `getComputedStyle(document.querySelector('.hero h1 mark')).color` = `rgb(42, 24, 84)`
  (texto índigo) e `backgroundImage` contém `linear-gradient`.
- CLS com `.claude/perf/medir-comparativo.py` trocando a URL de TESTE pela local: abaixo de
  0.1, como a produção.
- Gate: `.claude/perf/validar-gate.py` apontado para a local, `?reset=1&pitch=12`: `ABRIU`.
- Zero erro de console.

### F. Commit e push (só o arquivo de teste)

```bash
git add clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html
git commit -m "Pagina de teste: hero novo, h1 com marca-texto e so h1 + sub no celular"
git push origin HEAD
```

`git status` antes do commit tem que mostrar **só** esse arquivo modificado. Nunca `git add -A`
(o repo tem `.claude/` e arquivos untracked que não sobem).

Depois do push, esperar o `200` na URL de teste e conferir uma vez ao vivo os mesmos pontos
do item E.

## Rollback

`git revert` do commit. A produção não foi tocada, então não há o que reverter lá.

## Prompt para a próxima sessão

```
Leia rvd-ukulele-fundamental/clube-do-ukulele/HANDOFF-hero-novo-teste.md e execute as
secoes A a F nessa ordem, so na pagina -cl-teste. Antes de comecar, me mostre as tres
URLs da secao inicial e confirme qual e o alvo. Nao toque em -op nem em -cl. Nao pule a
secao D (regerar as fontes embutidas) nem a E. Um commit, git add explicito do arquivo
de teste, nunca git add -A. Ao terminar, me passe a URL de teste e o resultado do item E.
```
