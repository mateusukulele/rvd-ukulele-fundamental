# Play rate da RVD4 fechada: diagnóstico e lista de melhorias

Página: https://cursos.comotocarukulele.com/clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl
Lido ao vivo em 16/09/2026, headless Chromium, 390x844 e 1440x900, com `?reset=1`.
Escopo: só o que decide se a pessoa dá o play. Retenção, pitch e checkout ficam de fora.
Feito sem a skill `metodo-rvd`, a pedido; a leitura pelo método fica para uma segunda rodada.

## O que está no ar (medido)

| item | estado | leitura |
|---|---|---|
| Dobra 390x844 | h1 7 a 142 px, sub 159 a 258, vídeo 271 a 908 | vídeo entra na dobra, faltam 64 px do pé |
| Dobra 1440x900 | vídeo 84 a 804, texto ao lado, nudge com seta | correta |
| `smartAutoPlay.active` | **true**, `autoUnmute: false` | pendência nº1 da RVD2 resolvida: autoplay mudo com convite |
| Convite do autoplay | caixa amarela `rgba(255,248,0,.75)`, ícone de play preto, "Aperte para ouvir!", pulso | cobre o rosto; cor fora da marca |
| Posição do convite | de 41% a 72% da altura do vídeo | em 390x844 cabe; em 360x640 o texto fica abaixo da dobra (pela proporção) |
| Poster | `/img/poster-vsl-2.webp`, frame com legenda queimada "COMO APRENDER O INSTRUMENTO" | legenda do vídeo e caixa amarela brigam no mesmo lugar |
| `bigPlay` / `temporaryBigPlay` | true / true | ok |
| `fakeBar` | true | ok |
| `progressBar` / `videoTime` | false / false | ok, duração não aparece |
| `resume.active` | true | modal para quem volta |
| `pixels.active` | **false** | Meta não recebe evento do player, desde a RVD2 |
| Teste A/B de vídeo | 3 variantes (L02, L10, FECHADO 1x L2), evento `video_variant` no dataLayer | leitura por variante já existe |
| Pitch | `pitchTime: 1070` (17:50) | fora do escopo |
| Carregamento | LCP 526 ms, CLS 0,002, player pronto em 2,3 s (3G, CPU 4x) | não é gargalo |
| Acima do h1 | nada (sem logo, sem eyebrow, sem selo) | correto |

Sem número de play rate desta página nesta rodada: a base da RVD2 era 54,2% (n = 1.688 únicos)
com meta de 65% a 70%. Pegar o número atual no VTURB antes de mexer, senão não há linha de base.

## Melhorias, em ordem de impacto sobre esforço

### 1. Os primeiros 5 segundos do vídeo são o poster agora

Com Smart Autoplay ligado, quem chega não vê a imagem parada: vê o vídeo rodando mudo. O
poster só aparece até o player carregar (2 a 3 s em 3G). Hoje esses segundos são rosto
falando com legenda queimada, ou seja, sem som não acontece nada que convença a apertar.

Fazer: abrir o vídeo com o ukulele sendo tocado em close, mão da batida em movimento, 3 a 5
segundos, antes de qualquer fala. A motivação nº1 declarada do público é o som do
instrumento (25,5%, n = 19.686); mudo, o que vende o clique é ver o som acontecendo.
Vale para as 3 variantes do teste A/B, senão a comparação entre elas fica suja.

Esforço: corte de vídeo, sem mexer na página. Medir: play rate por variante no VTURB, antes
e depois, mesmo tráfego.

### 2. Reposicionar e recolorir o convite "Aperte para ouvir!"

A caixa amarela cobre o rosto e o texto do convite fica no terço de baixo do vídeo, que é
exatamente a parte que sai da dobra em celular menor (360x640 é o Android mais comum da
base). Quem não rola não lê o convite.

Fazer no painel do VTURB, item Smart Autoplay:
- Subir a caixa para a faixa de 20% a 50% da altura do vídeo (peito, não rosto).
- Trocar o amarelo por índigo `#443468` com texto branco, ou coral `#E87A85` com texto
  `#4D1A20`: pares aprovados da marca, e continua chamando atenção sem parecer anúncio de
  terceiro.
- Copy com o objeto do desejo, uma leitura só: "Toque para ouvir o ukulele". "Aperte para
  ouvir!" é genérico e não diz o que vai tocar.

Esforço: 10 minutos no painel. Medir: cliques no convite (o VTURB registra) e play rate em
360 de largura separado de 390.

### 3. Poster sem legenda queimada

O frame de poster tem "COMO APRENDER O INSTRUMENTO" queimado, e a caixa amarela cai em cima.
Duas mensagens no mesmo lugar, nenhuma legível. O poster ainda importa: é o que aparece nos
2 a 3 s de carregamento, no iOS com economia de dados (autoplay bloqueado) e em quem volta.

Fazer: exportar o poster de um frame do item 1 (mão nas cordas, sem texto), mesmo peso e
mesmo caminho `/img/poster-vsl-2.webp` para não mexer no preload.

Esforço: um export. Medir: junto com o item 2.

### 4. Subheadline que diz o que o vídeo mostra

A headline nova é emocional e funciona como gancho. A sub promete "3 passos" e manda dar o
play, mas não diz o que a pessoa vai ver quando apertar. Play rate sobe quando o texto perto
do player descreve a cena, não o benefício.

Testar uma variante da sub, uma variável por vez, mantendo a headline:

```
No vídeo eu pego uma música e mostro os 3 passos que fazem ela sair inteira,
mesmo se você nunca tocou nada antes. Dá o play para ver.
```

Esforço: uma linha no HTML, subsets regerados. Medir: A/B de página (a -cl-teste existe
para isso), dois dias por variante no mínimo.

### 5. Dobra em 360x640

Em Android de 360 px o h1 fica com 4 linhas, a sub com 4, e sobram cerca de 380 px de vídeo
na tela. O convite do item 2 resolve a metade de baixo; a metade de cima ainda pode ganhar
espaço:

- `.hero h1` de `clamp(1.6rem,7.6vw,3rem)` para `clamp(1.5rem,7vw,3rem)` abaixo de 380 px.
- `.hero-sub` de 4 para 3 linhas cortando "para tocar suas músicas favoritas" só nessa faixa,
  ou aceitando 3 linhas com `font-size` menor.

Só depois de medir play rate por largura de tela (GA4 tem `screen_resolution`). Se 360 não
converte pior que 390, não mexer.

### 6. Pixel dentro do player

`pixels.active: false` desde a RVD2. Sem isso o Meta não recebe play, 25%, 50% nem pitch, e
a campanha não consegue otimizar para quem assiste. Não muda o play rate da página, muda a
qualidade de quem chega nela, que é a outra metade da conta.

Esforço: painel do VTURB, colar o ID do pixel. Medir: eventos chegando no Gerenciador.

### 7. Coerência anúncio e página

Nunca vista nesta análise nem na anterior. Play rate é anúncio mais dobra. Se o criativo
promete "4 acordes" e a página abre com "20 minutinhos", a pessoa hesita antes do play.

Fazer: colocar lado a lado o criativo ativo e a dobra; a headline do anúncio e a da página
precisam ser a mesma promessa em palavras diferentes. Sem isso, todo teste acima mede ruído.

### 8. Quem volta

`resume.active: true` abre modal "continuar de onde parou" para quem volta. Correto para
retenção, mas cada retorno conta como visualização única sem play novo se a pessoa fecha o
modal. Conferir no VTURB se a métrica de play rate exclui retornos; se não exclui, o número
está deprimido artificialmente e a meta de 65% precisa ser relida.

## O que não mexer

- Estrutura da página fechada: nada além de h1, sub e vídeo até o pitch. Está certo.
- Duração escondida (`videoTime: false`). Mostrar 17:50 derruba play.
- Carregamento: LCP e CLS já estão no verde, não há ganho aqui.
- Desktop: dobra correta, nudge com seta apontando para o vídeo.

## Ordem sugerida

1. Pegar o play rate atual no VTURB (linha de base).
2. Itens 2 e 3 no mesmo dia (painel e um export, sem deploy de código).
3. Item 1 nas três variantes.
4. Item 4 como A/B na -cl-teste.
5. Itens 6 e 7 em paralelo, porque não dependem da página.

Uma variável por vez na página. Painel do VTURB e pixel não contam como variável de página.
