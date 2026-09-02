# Handoff — inventário do conteúdo da plataforma (Clube do Ukulele)

Criado em 02/09/2026. Esta tarefa **não** foi executada: ficou parada no login.
Abrir numa sessão própria, separada da que trabalha as páginas de venda.

## O que é pra fazer

Levantar tudo que existe dentro da área de membros do Clube do Ukulele e salvar
como material reutilizável — não como resposta de chat.

Plataforma: <https://aluno.comotocarukulele.com/area/vitrine/home>
O que está lá dentro é o Clube do Ukulele.

## Onde parou

Abri a URL pelo Chrome do Mateus (ferramentas `mcp__claude-in-chrome__*`) e caí
em `/auth/login?redirect=%2Farea%2Fvitrine%2Fhome`. A sessão do Chrome não
estava autenticada. Ele relatou que **nada abriu na tela dele** — vale conferir
se a extensão está conectada ao Chrome certo antes de culpar o login.

Senha eu não digito. O Mateus precisa entrar na aba antes de a varredura começar.

### Checklist de partida
1. `ToolSearch` com `select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__get_page_text,mcp__claude-in-chrome__read_page,mcp__claude-in-chrome__javascript_tool,mcp__claude-in-chrome__find`
2. `tabs_context_mcp` com `createIfEmpty: true` e conferir que a aba aparece **na tela dele**.
3. Se cair no login, pedir para ele autenticar e confirmar.
4. Só então varrer.

## O que extrair

Por curso, na ordem em que aparecem na plataforma:

- Nome do curso e a que nível pertence (Iniciante 0, 1, 2, Intermediário 1, 2, extras).
- Módulos, com ordem e nome.
- Aulas, com ordem, nome e duração quando o player expuser.
- Materiais anexos por aula (PDF, cifra, tablatura) — só o nome e o tipo.
- Se a aula é gravação de live, marcação disso.

Para a biblioteca de músicas: nome da música, artista, nível e o que acompanha
(aula, destrinchamento, Toque Comigo, PDF). A página de venda promete **154
músicas** — conferir o número real e avisar se divergir.

Preferir `javascript_tool` extraindo do DOM em lote (uma passada por listagem)
a clicar item por item. Se a plataforma tiver API interna (olhar a aba de rede),
puxar o JSON dela é mais rápido e mais fiel que raspar HTML.

## Formato de saída

Salvar em `clube-do-ukulele/inventario/`:

- `conteudo.json` — fonte de verdade, estruturada, para script consumir depois.
- `conteudo.md` — mesma coisa em tabela, para leitura humana.

Sugestão de forma para o JSON (ajustar ao que a plataforma realmente entregar):

```json
{
  "extraido_em": "2026-09-02",
  "origem": "https://aluno.comotocarukulele.com/area/vitrine/home",
  "cursos": [
    {
      "nome": "Iniciante 1",
      "nivel": "iniciante",
      "modulos": [
        {
          "nome": "Primeiros acordes",
          "aulas": [
            {"ordem": 1, "nome": "...", "duracao": "07:12", "materiais": ["cifra.pdf"]}
          ]
        }
      ]
    }
  ],
  "biblioteca": [
    {"musica": "...", "artista": "...", "nivel": "iniciante", "tem": ["aula", "toque-comigo", "pdf"]}
  ]
}
```

## Perguntas que precisam de resposta antes de publicar qualquer número

1. A biblioteca de músicas está na mesma vitrine ou é área separada? Muda o tamanho da varredura.
2. Existe curso lá dentro que **não** entra na oferta do Clube (produto antigo, turma fechada)?
   Não pode entrar na página de venda o que o comprador novo não recebe.
3. O total de aulas por nível bate com o que as páginas prometem hoje?

## Para que isso vai servir depois

- Alimentar uma seção de conteúdo nas páginas de venda (trilha por nível com
  contagem real de aulas, em vez de descrição genérica).
- Roteiro de vídeos de walkthrough pela plataforma.
- Base para material de apoio e para responder dúvida de aluno sem abrir a área.

## Contexto das páginas (não mexer nesta sessão)

As páginas de venda vivem em `clube-do-ukulele/ev-semana-grupo-vip/`:
`prematricula-inscricao.html` (captura), `o-que-voce-vai-receber.html` (prévia)
e `aproveite-a-oferta.html` (oferta do dia 03/09, 8h às 20h).
Se a varredura revelar número diferente de 154 músicas, **avisar** em vez de
editar a página: a sessão das páginas cuida disso.
