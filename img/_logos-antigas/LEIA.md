# Logos antigas do Clube do Ukulele — não usar

A logo para usar é **`../logo-clube-do-ukulele.webp`** (1190x778, fundo transparente).
Existe uma arte só. Tudo aqui dentro é a mesma marca com defeito de arquivo.

## O que tem de errado em cada uma

| arquivo | canvas | conteúdo real | problema |
|---|---|---|---|
| `logo-clube-horizontal.webp` | 900x471 | 465x302, ocupa 52% da largura | O nome mente. Não é lockup horizontal: é a mesma logo com 218px de vazio transparente de cada lado. Num box de 200px de largura o escudo renderiza a ~76px e fica ilegível. Foi o que estragou o hero da pré-matrícula em 27/08/2026. |
| `logo-clube-oficial.webp` | 800x800 | 797x522 | Upscale borrado: bordas com halo e contorno mole. Medido, tem menos energia de borda por pixel que a versão de 600. O "oficial" no nome também mente. |
| `Logo Clube do Ukulele.webp` | 600x600 | 595x389 | Arte correta, resolução baixa e 146px de vazio em cima e embaixo. Foi substituída pelo master de 1200px. |

## De onde veio a canônica

`/Users/mateusaugusto/Downloads/Logo Original - fundo transparente - Clube do Ukulele agosto 2026.png`
(1200x1200, alpha, pixel a pixel idêntica a `Clube do Ukulele Academy - O Produto/Marca/Logos/IMG - Logo Clube 1 x 1 (Original).png`).

Recortada no bounding box do alfa, sem redimensionar, sem recolorir, sem tocar em um pixel da arte.
O mesmo recorte de 1190x778 já era usado nos PDFs da apostila.

## Regras de uso

- Largura mínima 96px, como manda o Manual Visual 2026.
- Fundo escuro é onde ela fica melhor: o painel branco e as asas laranja saltam do índigo.
- Nunca recolorir, distorcer nem recortar o escudo.
- Precisa de mais respiro em volta? Use `padding` no CSS, nunca um arquivo com canvas maior. Foi assim que nasceu a `horizontal`.
