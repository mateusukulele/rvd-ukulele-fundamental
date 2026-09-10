# HANDOFF: levar o pacote de performance da RVD4 para produção

Escrito em 10/09/2026. Vale para as duas páginas de venda do Clube:

- ABERTA: `clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-op.html`
- FECHADA: `clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl.html`

Fonte da verdade do pacote: `clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html`,
publicada em `/clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste` (noindex), testada
pelo Mateus em Chrome, Safari e iPhone, e medida contra a produção.

Ferramentas locais (não vão para o deploy, ficam em `.claude/perf/`, untracked):

- `port-perf-op.py`: aplica o pacote na página ABERTA por âncoras, com assert em cada passo.
- `medir-comparativo.py`: LCP, CLS, FCP, bloqueio e peso, 4 rodadas, cache frio, ordem alternada.
- `medir-responsivo.py`: proporção do vídeo e encaixe na dobra em retrato e paisagem.
- `validar-gate.py`: o gate por `onTime` abrindo com o vídeo tocando (headless mudo).

Exigem o Playwright Python do homebrew com o Chrome do sistema:
`/opt/homebrew/opt/python@3.14/bin/python3.14` e `channel='chrome'`. Sempre `--mute-audio`.

## O que o pacote muda (as quatro coisas, e por quê)

1. **Montserrat 900 embutida no HTML** (subset Latin-1 mais travessão, aspas, bullet,
   reticências e sinal de menos, 10,8kb) e as outras fontes servidas de `/fonts/`, sem Google
   Fonts e **sem preload**. Motivo medido: o h1 nascia com 3 linhas na fonte do sistema e
   virava 4 quando a Montserrat chegava, empurrando o vídeo. Era 0.09 dos 0.207 de CLS.
2. **Poster do VTurb atrás do player**, pintado assim que o teste A/B resolve o vídeo, com o
   placeholder do VTurb transparente e o player com `z-index` próprio. LCP de ~7,7s para
   ~3,5s e, mais importante, parou de oscilar entre 0,3s e 9s. Custa 81kb.
3. **Dobra sem `ResizeObserver`** no texto: remede só na fonte pronta, no resize e na rotação.
4. **Celular deitado em duas colunas**, texto de um lado e vídeo 9:16 do outro. Antes o vídeo
   ficava com 66x117 esmagado. Defeito herdado, existia nas duas páginas.

O gate por `onTime` **já está** na fechada em produção. Não faz parte deste porte.

Medido lado a lado (390x844, 4G lento, CPU 4x, mesma variante do A/B nas duas):

```
                  ATUAL      TESTE
CLS               0.207      0.097
LCP (mediana)    7684ms     3492ms
FCP               390ms      406ms
player pronto    2602ms     2279ms
```

## Passo a passo

### A. Página FECHADA (`-cl`)

A página de teste É a fechada mais o pacote. O porte é uma cópia com duas linhas de volta.

```bash
cd "/Users/mateusaugusto/Claude Code/rvd-ukulele-fundamental"
cp clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl.html
```

Depois, no arquivo `-cl` recém-copiado, exatamente estas duas trocas:

1. A linha `<meta name="robots" content="noindex, nofollow"><!-- pagina de teste -->`
   volta a ser `<meta name="robots" content="index, follow, max-image-preview:large">`.
2. No comentário do gate, `GATE da VSL, versao de TESTE: abre` vira `GATE da VSL: abre`.

Conferir antes de commitar (tudo tem que dar o valor indicado):

```bash
f=clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl.html
grep -c "noindex" $f                                  # 0
grep -c "versao de TESTE" $f                          # 0
grep -c 'content="index, follow' $f                   # 1
grep -c "data:font/woff2" $f                          # 1
grep -c "@font-face" $f                               # 8  (M 400/700/800, M 900 embutida + arquivo, P 400/500/600)
grep -c "poster-lcp" $f                               # 2
grep -c "orientation:landscape" $f                    # 2
grep -c "new ResizeObserver" $f                       # 0
grep -c "fonts.googleapis.com" $f                     # 0
grep -c "el.onTime(" $f                               # 1
grep -o "rvd-clube-do-ukulele-rvd4v1-vert-cl[^\"']*" $f | sort -u   # só a URL da -cl, sem "-teste"
```

### B. Página ABERTA (`-op`)

A aberta tem outra dobra (botão fora da dobra, sem gate), então não é cópia: é o script.

```bash
cd "/Users/mateusaugusto/Claude Code/rvd-ukulele-fundamental"
/opt/homebrew/opt/python@3.14/bin/python3.14 .claude/perf/port-perf-op.py \
  clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-op.html \
  clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-op.html
```

O script escreve por cima da própria `-op`. Se qualquer âncora não bater uma vez, ele para
com `AssertionError` **antes** de escrever. Ele já foi rodado num rascunho em 10/09/2026 e
aplicou limpo, com os dois scripts inline passando em `node --check`.

Conferir:

```bash
f=clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-op.html
grep -c "data:font/woff2" $f            # 1
grep -c "@font-face" $f                 # 8  (sem a 800, botoes e titulos cairiam na 900)
grep -c "poster-lcp" $f                 # 2
grep -c "orientation:landscape" $f      # 2
grep -c "new ResizeObserver" $f         # 0
grep -c "fonts.googleapis.com" $f       # 0
grep -c "deitado=" $f                   # 1
grep -c 'content="index, follow' $f     # 1
grep -c "el.onTime(" $f                 # 0  (a aberta não tem gate, e não deve ganhar um)
```

### C. Validar as duas no preview local antes do push

Subir o preview `rvd-repo` (porta 8129) e rodar, apontando para `localhost:8129`:

- `medir-responsivo.py`: as quatro larguras com ratio `0.5625` e "cabe". Na fechada, o
  vídeo deitado fica 186x330; na aberta o número pode diferir, o ratio não.
- `validar-gate.py` só na fechada: `ABRIU` com o vídeo em ~12s, `vsl_unlock` no dataLayer.
- Na aberta, abrir em 390x844 e conferir por DOM: `.cta-block .btn` com `top >= innerHeight-6`
  (o botão continua fora da dobra), vídeo 9:16, `#poster-lcp` com `naturalWidth > 0`,
  `getComputedStyle(vturb-smartplayer).zIndex === '1'`.
- Nas duas: `document.fonts.check('900 24px Montserrat')` true, e a largura de `$` em
  `900 40px Montserrat` diferente da largura em `system-ui` (prova que o cifrão está no subset).
- Nas duas: `dataLayer` com `sales_page_view` e `video_variant`; `HOTMART_URL` intacto; zero
  erro de console.

### D. Commit e push

Um commit por página, com os dois arquivos explicitamente (nunca `git add -A`: o repo tem
`.claude/` e arquivos de teste untracked que não podem subir).

```bash
git add clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl.html
git commit -m "Fechada recebe o pacote de performance da pagina de teste" ...
git add clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-op.html
git commit -m "Aberta recebe o pacote de performance, portado por script" ...
git push origin HEAD
```

Depois do push, esperar o `200` e conferir em produção com `medir-comparativo.py` trocando
as duas URLs pelas de produção antes e depois (o "antes" só existe até o deploy; se quiser
guardar, rode antes do push).

### E. Limpeza (só depois de as duas estarem no ar e conferidas)

- Apagar do repo (commit próprio): `clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl-teste.html`.
- Apagar do disco, nunca commitados: `rvd4-perf-teste.html`, `teste-ontime.html`.
- **Não tocar** em `area-clube-do-ukulele/bloco-biblioteca-sem-acesso.html`: untracked, é de outro trabalho.

## Rollback

`git revert` do commit da página que quebrou. As duas páginas são independentes.

## O que ainda depende do Mateus

- Manter ou tirar o **poster** (81kb a mais por visita; ganho é LCP estável e o rosto aparecer
  durante o carregamento). O pacote vai com ele salvo ordem em contrário.
- **CLS em 0.097** contra limite de 0.1: passa por 3 milésimos. Um aparelho que quebre a
  headline em outro número de linhas pode estourar. Se aparecer no CrUX, o suspeito é o h1.
- O `vsl_unlock` da fechada sai com `unlock_motivo` (`onTime` ou `teto-relogio`). Vale uma
  variável no GTM para ver quanta gente cai no teto, que é gente que abre e não dá play.

## Prompt para a próxima sessão ou subagente

```
Leia rvd-ukulele-fundamental/clube-do-ukulele/HANDOFF-perf-rvd4.md e execute as
seções A, B, C e D nessa ordem. Nao pule a conferencia C. Um commit por pagina,
git add explicito dos dois arquivos, nunca git add -A. Mantenha o poster.
Ao terminar, rode .claude/perf/medir-comparativo.py contra as URLs de producao
e reporte CLS, LCP e FCP das duas paginas. Nao execute a secao E.
```
