# Como saber qual vídeo do teste A/B do VTurb estava vendendo

Documento de replicação. Vale para qualquer landing do repo que use `<vturb-smartplayer>`.
Referência funcionando em produção: `ukulele-fundamental/rvd-ukulele-fundamental-rvd3v1-vert-cl.html`
(linhas 882 a 895, 957, 966 a 969).

---

## 1. O problema

O VTurb serve um vídeo diferente por visitante quando o teste A/B está ligado. O HTML da
página não muda: o atributo `id` do `<vturb-smartplayer>` é o ID do **player**, fixo, e
continua o mesmo para todo mundo.

```html
<vturb-smartplayer id="vid-6a7cadacf642b344091be235"></vturb-smartplayer>
```

Esse `6a7cadacf642b344091be235` é o player. Não diz nada sobre qual vídeo foi entregue.

Sem resolver isso, o GA4 mostra "a página vendeu 40" e a Hotmart mostra "40 vendas", sem
separar quanto veio do vídeo A, do B ou do C. O teste A/B fica cego do lado do dinheiro.

## 2. A solução

O `<vturb-smartplayer>` é um Web Component. Depois que o script do VTurb hidrata o elemento,
a instância expõe a propriedade **`el.config`**, e dentro dela:

| Caminho | O que é |
|---|---|
| `config.video.id` | ID do vídeo **que o servidor realmente entregou nesse carregamento**. Muda por visitante quando o A/B está ligado. É o valor que interessa. |
| `config.name` | Nome do player no painel do VTurb. Ex.: `RVD V3 - Ukulele Fundamental - Agosto/26 - CUPOM 200 (FECHADO)` |
| `config.id` | ID do player. Igual ao que está no HTML. |

Lendo `config.video.id`, guardando numa variável global e carimbando esse valor em dois
lugares (dataLayer e `sck` da Hotmart), a venda volta identificada por vídeo.

## 3. O código

### Bloco 1: leitura da variante

Cola logo depois da declaração do `HOTMART_URL`, antes de qualquer coisa que use a variável.

```js
/* ===== Variante de vídeo (teste A/B do VTurb) =====
   O player expõe em config.video.id QUAL vídeo foi servido, não o que está
   escrito no HTML. Com o teste A/B desligado é sempre o mesmo; com o teste
   ligado muda por visitante. Guardamos isso pra carimbar o GTM e o checkout,
   senão não dá pra saber qual vídeo gerou a venda. */
var VIDEO_VARIANT=null;
(function(){
  function ler(){
    var el=document.querySelector('vturb-smartplayer'), c=el&&el.config;
    if(!c||!c.video||!c.video.id) return false;
    VIDEO_VARIANT={id:c.video.id,curto:c.video.id.slice(-6),nome:c.name||'',player:c.id||''};
    window.dataLayer=window.dataLayer||[];
    window.dataLayer.push({event:'video_variant',video_id:VIDEO_VARIANT.id,video_variant:VIDEO_VARIANT.curto,video_name:VIDEO_VARIANT.nome,video_player_id:VIDEO_VARIANT.player});
    return true;
  }
  if(ler()) return;
  var t=setInterval(function(){ if(ler()) clearInterval(t); },300);
  setTimeout(function(){ clearInterval(t); },15000);
})();
```

Quatro decisões que fazem isso funcionar. Não mexa nelas sem entender:

1. **Polling, não evento.** O `config` só existe depois que o `smartplayer.js` hidrata o
   componente, e não há evento público confiável para esse momento. O código tenta na hora,
   e se não achar tenta a cada 300 ms. Desiste em 15 s para não deixar timer eterno rodando.
2. **`slice(-6)` cria a versão curta.** O ID cheio tem 24 caracteres hexadecimais. Não cabe
   bem em nome de variante no GA4 nem no `sck` da Hotmart. Os 6 últimos já separam as
   variantes sem risco real de colisão.
3. **Evento próprio no dataLayer (`video_variant`).** Melhor que tentar anexar num evento
   existente, porque o valor chega tarde (depois do `sales_page_view`, quase sempre depois
   do `gtm.js`). Evento separado significa que qualquer tag pode ser acionada por ele.
4. **O ID cheio e o curto vão juntos.** O curto serve para relatório, o cheio serve para
   você conferir no painel do VTurb qual vídeo é aquele.

### Bloco 2: carimbo no checkout da Hotmart

Dentro do handler de `submit` do formulário, na montagem da URL:

```js
var qp=new URLSearchParams(location.search), utms=[];
['utm_source','utm_medium','utm_campaign','utm_term','utm_content'].forEach(function(k){
  var val=qp.get(k); if(val){ url.searchParams.set(k,val); utms.push(val); }
});
/* Carimba a variante de vídeo no sck. É o que faz a venda na Hotmart
   voltar identificada por vídeo, sem isso A, B e C viram um número só. */
if(VIDEO_VARIANT) utms.push('v'+VIDEO_VARIANT.curto);
if(utms.length) url.searchParams.set('sck',utms.join('|'));
```

Resultado real, medido na página em produção com UTMs de teste:

```
sck = fb|teste|v63c448
```

O prefixo `v` existe para você distinguir a variante dos UTMs no relatório da Hotmart. Sem
ele, `63c448` no meio de uma lista de UTMs não se identifica sozinho.

### Bloco 3: carimbo nos outros eventos

Todo evento de conversão da página repete a variante no próprio push, em vez de confiar que
o GTM guardou o valor de antes:

```js
window.dataLayer.push({
  event:'lead_capture_uke',
  /* ...demais campos... */
  video_variant: VIDEO_VARIANT ? VIDEO_VARIANT.curto : 'na',
  video_id: VIDEO_VARIANT ? VIDEO_VARIANT.id : ''
});
```

E no evento de liberação do CTA (só existe na versão fechada da página):

```js
window.dataLayer.push({
  event:'vsl_unlock',
  unlock_seconds:SEGUNDOS,
  video_variant:(window.VIDEO_VARIANT ? window.VIDEO_VARIANT.curto : 'na')
});
```

O fallback `'na'` é proposital: se a leitura falhar, o relatório mostra `na` em vez de
sumir com a linha. Some no relatório é pior, porque parece que o evento não aconteceu.

## 4. Como conferir que funcionou (no navegador, antes do GTM)

Abre a página publicada, F12, aba Console, e roda:

```js
document.querySelector('vturb-smartplayer').config.video.id
```

Se voltar um hexadecimal de 24 caracteres, o mecanismo tem de onde ler. Depois:

```js
window.VIDEO_VARIANT
window.dataLayer.filter(x => x && x.event === 'video_variant')
```

O segundo comando tem que devolver um objeto assim:

```json
{
  "event": "video_variant",
  "video_id": "6a7cad437d249b839f63c448",
  "video_variant": "63c448",
  "video_name": "RVD V3 - Ukulele Fundamental - Agosto/26 - CUPOM 200 (FECHADO)",
  "video_player_id": "6a7cadacf642b344091be235"
}
```

Se `VIDEO_VARIANT` continuar `null` depois de 15 segundos, veja a seção 7.

## 5. Como fazer isso chegar no GTM

O código só empurra o dado para a camada de dados. Quem transforma isso em relatório é o
GTM. Três peças, nessa ordem. Todas em `serversidegtm.comotocarukulele.com`, container
`GTM-PDH6QGB2`.

### 5.1. Variáveis de camada de dados

Menu **Variáveis**, seção "Variáveis definidas pelo usuário", botão **Nova**, tipo
**Variável de camada de dados**. Crie uma para cada campo que for usar. Nome da variável de
camada de dados tem que ser **exatamente igual** à chave do push, sem espaço:

| Nome da variável no GTM | Nome da variável de camada de dados |
|---|---|
| `DL - video_variant` | `video_variant` |
| `DL - video_id` | `video_id` |
| `DL - video_name` | `video_name` |

Na maioria dos casos só o `video_variant` é necessário. Os outros dois servem quando você
precisa achar o vídeo no painel do VTurb.

Deixe **Versão da variável de camada de dados** em "Versão 2" (o padrão).

### 5.2. Acionador

Menu **Acionadores**, **Novo**, tipo **Evento personalizado**.

- Nome do evento: `video_variant`
- Marque "Usar correspondência de regex": **não**
- Este acionador é ativado em: **Todos os eventos personalizados**

### 5.3. Tag

Aqui existem dois caminhos. Escolha um.

**Caminho A, evento próprio no GA4** (você quer ver o carregamento por variante):

Tag do tipo **Evento do Google Analytics: GA4**, acionada pelo acionador `video_variant`,
com:

- Nome do evento: `video_variant`
- Parâmetros do evento:
  - `video_variant` = `{{DL - video_variant}}`
  - `video_id` = `{{DL - video_id}}`

**Caminho B, parâmetro nos eventos que já existem** (você quer separar as conversões que já
mede por variante):

Não cria tag nova. Vá nas tags de conversão que já existem (`lead_capture_uke`, compra,
etc.) e acrescente o parâmetro `video_variant` = `{{DL - video_variant}}` em cada uma. Como
os pushes de conversão já carregam o campo no próprio evento (bloco 3 acima), a variável de
camada de dados resolve certo na hora do disparo, sem depender de persistência.

O caminho B é o que responde "qual vídeo vendeu". O caminho A responde "qual vídeo foi
servido, e quantas vezes". Dá para ter os dois.

**Atenção:** não invente nome de evento nem de parâmetro que o gestor de tráfego ainda não
configurou do lado do GA4. Antes de publicar, alinhe os nomes com quem cuida do container.
Este documento descreve o que a página envia; o que o container faz com isso é decisão de
quem administra o GTM.

### 5.4. Conferir no GTM

Modo de **Visualização** (Preview) do GTM, abra a página. Na linha do tempo do Tag Assistant
tem que aparecer o evento `video_variant` depois do `gtm.js`. Clique nele, aba **Variables**,
e confirme que `DL - video_variant` tem valor, não `undefined`.

Se aparecer `undefined`: o nome da variável de camada de dados está diferente da chave do
push, ou você está olhando um evento anterior ao `video_variant` na linha do tempo.

### 5.5. Onde ver o resultado na Hotmart

Relatório de vendas, coluna **SCK** (aparece como "src/sck" dependendo da tela). O valor vem
no formato `utm_source|utm_campaign|vXXXXXX`. Filtre pelo trecho `v63c448` para isolar as
vendas de uma variante. Exportando o CSV, dá para separar por `|` no Sheets e somar por
variante.

## 6. Checklist para replicar numa página nova

1. A página tem `<vturb-smartplayer>` e carrega o `smartplayer.js` do ConverteAI.
2. Colar o **bloco 1** logo depois da linha do `var HOTMART_URL=...`.
3. No `submit` do formulário, colar as duas linhas do **bloco 2** (o `utms.push` e o
   `searchParams.set('sck', ...)`), depois do loop dos UTMs.
4. Nos pushes de conversão da página, acrescentar `video_variant` e `video_id` (**bloco 3**).
5. Conferir o `page_name` do `sales_page_view`: ao copiar uma página, esse campo vem com o
   nome da página antiga e ninguém percebe, porque não gera erro. Corrija para o nome da
   página nova.
6. Publicar e conferir no console pelos comandos da seção 4.
7. Conferir no modo Visualização do GTM (seção 5.4).

## 7. Armadilhas

**`el.config` é propriedade interna do componente, sem contrato público do VTurb.** Se eles
mudarem o formato numa atualização do `smartplayer.js`, a leitura para de preencher e cai
silenciosamente no `'na'`. O código não lança erro nenhum, de propósito, para não quebrar a
página de vendas. O preço disso é que a falha é silenciosa. Confira no console de tempos em
tempos, e sempre depois de trocar o player.

**O `config` não existe no primeiro tick.** Quem tentar ler `el.config` direto, sem o
polling, vai pegar `undefined` na maior parte das visitas. O intervalo de 300 ms não é
enfeite.

**O `sales_page_view` dispara antes do `video_variant`.** Ordem medida em produção:
`sales_page_view` (id 3), depois `video_variant` (id 10). Então não adianta tentar carimbar
a variante no evento de pageview: naquele instante ela ainda não existe. É exatamente por
isso que o mecanismo usa evento separado.

**Bloqueador de anúncio.** Se o visitante bloqueia o `scripts.converteai.net`, o player não
carrega, o `config` nunca aparece e a variante vira `'na'`. Esse é o caminho legítimo do
fallback, não é bug.

**Não confunda os dois IDs.** `config.id` é o player (fixo, igual no HTML). `config.video.id`
é o vídeo (muda com o A/B). Trocar um pelo outro faz o relatório mostrar 100% de uma
variante só, e parecer que o teste A/B está desligado.

---

Última verificação em produção: 08/09/2026, na página
`cursos.comotocarukulele.com/ukulele-fundamental/rvd-ukulele-fundamental-rvd3v1-vert-cl`.
Variante servida no teste: `6a7cad437d249b839f63c448` (curto `63c448`).

---

# Parte 2: a Conversion Key do VTurb (parâmetro `src`)

Mecanismo diferente do de cima e complementar a ele. A variante responde "qual
vídeo foi servido". A Conversion Key responde "qual sessão de vídeo gerou esta
venda", e é o que o painel do VTurb usa para atribuir conversão.

No ar desde 08/09/2026 em `rvd-ukulele-fundamental-rvd2v1-op`,
`rvd-ukulele-fundamental-rvd2v1-cl`, `clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-op`,
`clube-do-ukulele/rvd-clube-do-ukulele-rvd4v1-vert-cl` e na página de teste
`clube-do-ukulele/rvd-teste-src`.

## 1. O formato

```
v3_<session_id>_<player_id>_<ms no vídeo>[_t-<turbo×10>][_h-<headline>][_s-<smartautoplay>]
```

Exemplo do próprio VTurb:

```
v3_bff0a123-5c10-49d2-8059-ed03bcbb38e7_67f9605e3c95e6f3915a01f8_2800_t-13_s-1
```

O nome do parâmetro que carrega essa chave é escolhido no painel, em
Configurações → Conversões. Nos quatro players das nossas páginas está `src`, e
dá para conferir sem abrir o painel: `document.querySelector('vturb-smartplayer').config.conversion`
devolve `["src"]`.

## 2. Por que a injeção automática não pega nestas páginas

O VTurb promete injetar a chave sozinho "mesmo que o botão tenha sido feito em
html". Lendo o `smartplayer.js`, a injeção escuta `mouseover`, `pointerdown` e
`touchstart` na janela, acha o elemento clicável mais próximo e:

- se for `<a href>`, reescreve o `href`;
- se for campo de formulário, reescreve o `action` do formulário.

Nenhuma das nossas páginas se encaixa. O formulário de lead não tem `action`, e a
ida para o checkout é feita com `location.href` dentro do `submit`. Resultado: sem
código nosso, a chave nunca chega na Hotmart. Isso é silencioso, não gera erro.

## 3. Como a chave é lida

O player dispara no próprio `<vturb-smartplayer>` o evento
`conversion-tracking:update`, com `detail.key` sendo a chave. Primeiro disparo no
`firstUpdated` do componente, e depois um a cada `timeupdate` do vídeo, com o `ms`
atualizado.

Duas consequências que decidem onde o código fica:

1. **O listener tem que estar registrado antes do `player.js`.** Por isso o bloco
   fica logo depois da tag `<vturb-smartplayer>`, e não junto do resto do JS lá
   embaixo. Quem registra depois perde a primeira chave.
2. **Só o primeiro disparo vira evento no dataLayer.** Os seguintes chegam a cada
   `timeupdate` e encheriam o dataLayer de ruído. O valor atualizado fica em
   `window.VTURB_SRC` e é lido de novo na hora de montar a URL do checkout.

O fallback é uma âncora escondida (`#vt-src-probe`), sem `href` no HTML. Na hora
do checkout o código escreve o `HOTMART_URL` nela e dispara um `pointerdown`: o
próprio VTurb carimba o `src` no `href` e a gente lê de volta. É o caminho
nativo, usado só se o evento não tiver chegado.

## 4. O que a página passa a emitir

```js
{
  event: 'vturb_src',
  vturb_src: 'v3_...',        // chave inteira, é ela que vai no checkout
  vturb_session: '...',        // UUID da sessão de vídeo
  vturb_player: '...',         // id do player
  vturb_video_ms: '2800',      // ms de vídeo no momento
  vturb_turbo: '13',           // velocidade × 10, quando há teste de turbo
  vturb_headline: '',          // número da headline, quando há teste
  vturb_autoplay: '1'          // número do SmartAutoPlay, quando há teste
}
```

Os mesmos campos vão junto no `lead_capture_uke` / `lead_capture_clube`, então
não dependem de ordem de push. E a URL da Hotmart ganha `src=<chave>`.

## 5. Como conferir

Na página publicada, console:

```js
document.querySelector('vturb-smartplayer').config.conversion   // ["src"]
window.VTURB_SRC                                                 // v3_...
window.dataLayer.filter(x => x && x.event === 'vturb_src')
```

Se `VTURB_SRC` estiver `null`, o player ainda não montou. Ele só monta com a aba
visível: em navegador headless ou com a aba em segundo plano, `document.hidden` é
`true`, o player para no thumbnail e a chave nunca nasce. Isso não é bug da
página, e é a razão de esse teste não fechar pelo preview interno.

## 6. Armadilha do lado da Hotmart

`src` é campo de origem de tráfego na Hotmart e aparece no relatório de vendas.
Depois desta mudança ele passa a mostrar a Conversion Key, não um rótulo de
origem. Quem quiser origem continua olhando o `sck`, que segue no formato
`utm_source|utm_campaign|vXXXXXX` e não foi tocado.
