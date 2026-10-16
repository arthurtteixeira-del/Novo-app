# João Bobo 3

Jogo de celular em um único arquivo: `index.html`. É um boneco 3D (WebGL) que o jogador
bate tocando na tela. Ele reage com física de verdade, faz caretas, se machuca, cai, levanta,
xinga, chora e provoca. Cada golpe dá moedas, e com as moedas o jogador compra armas na loja.

## Regras fixas

- **Feito para celular em pé (retrato).** Layout com cabeçalho em cima, palco no meio e a barra
  de ferramentas embaixo, no alcance do polegar. Largura máxima de 560 px; tudo por toque.
- **Sem bibliotecas externas.** Nada de three.js, frameworks ou bundlers: WebGL puro, SVG puro,
  Web Audio puro, tudo escrito à mão dentro do `index.html`. (A única coisa de fora são as fontes
  do Google Fonts, com fonte de reserva do sistema se não carregarem.)
- **Textos em português do Brasil**: interface, falas do boneco e comentários do código.
- O arquivo principal se chama `index.html` e continua sendo um arquivo só.

## O que o jogo faz

- **Boneco**: cabeça grande com cabelo espetado, corpo em forma de bolinha (ovo de vinil),
  mãos de luva de desenho animado e sapatões. Não tem braços nem pernas desenhados: as mãos e
  os pés "flutuam", mas seguem um esqueleto invisível.
- **Física**: esqueleto de 17 pontos (Verlet) com músculos, equilíbrio de pêndulo invertido,
  passos de recuperação, queda, sentar e levantar. Arrastar fora do boneco, usar dois dedos ou a
  barra de baixo gira o boneco.
- **Rosto**: olhos e bocas são ilustrações SVG desenhadas num atlas (textura) e pintadas na
  cabeça pelo shader. As pupilas seguem o dedo. Expressões: calmo, irritado, bravo, furioso,
  feliz, assustado, apanhando, zonzo, nocaute, deboche, triste, choro etc.
- **Comportamento**: humor (calmo/bravo, com 3 níveis de raiva), tristeza e choro em sequências
  longas de golpes, defesa com as mãos, guarda no lugar onde você insiste, gestos à toa (acenar,
  apontar, dedo do meio, sacudir o punho...).
- **Machucados** (ficam salvos até curar): olho roxo em estágios, veia estourada com lágrima de
  sangue, nariz quebrado e torto, boca sangrando, dentes que caem, orelha inchada e sangrando,
  hematomas, galos (inclusive por cima do cabelo), cortes, mão inchada/roxa, marcas de arma,
  sangue em gotas 3D que mancha o chão, a parede e o corpo.
- **Ferramentas**: Cutucão, Tapa e Soco de graça; na loja, Chinelo, Taco e Taser.
- **Personalização**: cor da pele (paleta ou qualquer cor) e roupa (terno ou camisola de paciente).
- **Cenário**: academia de boxe à noite (lona do ringue, cordas, parede de tijolo com cartazes),
  tudo desenhado no shader.

## Como o código está organizado

Tudo fica dentro de um `<script>` com uma função que roda sozinha. Seções, na ordem do arquivo
(procure por `/* ---------- `):

1. **util e matrizes** — atalhos (`$`, `rnd`, `clamp`, `mix`, `pick`) e matrizes 4×4 (`M4`).
2. **formas (campos de distância)** — esfera, elipsoide, cápsula, união suave; constantes do
   corpo (`NECK`, `PROP`) e `ballM()` (matriz do corpo-bolinha com o amassado).
3. **malhador** — `meshSDF` transforma um campo de distância em malha (surface nets);
   `sphereMesh` faz a esfera usada em quase tudo (`ball`).
4. **personagens** — `CHARS`: a ficha do boneco (cabeça, cores, acessórios, falas). Cabelo de
   pirâmides (`hairGelMesh`) e parte raspada (`shavedMesh`).
5. **WebGL e shaders** — `VS`/`FS`, a lista de uniforms `U` e o cenário (`ENV`). O shader escolhe
   o que pintar por `uMode`: `0` peça comum · `.1` peça da mão (traço fino na borda) · `.25` cabelo raspado · `1` corpo (roupa, roxo na
   barriga) · `2` cabeça (rosto, olhos, boca, machucados) · `3` contorno de nanquim · `4` chão ·
   `5` parede · `6` peça com contorno desenhado na borda (nariz, orelhas).
6. **malhas** — `Clay` (malha que guarda caixas para o toque, pinta sangue e muda de cor) e
   `buildChar` (monta corpo-bolinha, cabeça e acessórios).
7. **corpo ativo** — esqueleto (`J`, `BONES`, `X`/`Q`), poses, `solve`, equilíbrio, passos,
   `knockDown`, `physics`, gestos à toa (`idleActs`, `provoke`).
8. **estado e rosto** — estado do jogo `S`, machucados (`hurtHit`, `markHead`), sangue 3D
   (`BLOOD`) e `updateFace` (escolhe expressão de olho `S.eyeK` e de boca `S.mouthK`).
9. **efeitos, som e falas** — partículas SVG (`part`, `ono`, `stars`), atlas das bocas e dos olhos
   (`buildMouths`, `buildEyes`), mãos 3D de vinil (`HAND3`: palma e gomo de dedo, as mesmas malhas
   nas duas mãos; poses em `HPOSE`, escolhidas por `handPose` e montadas por `handParts`, que também
   guarda onde a mão está na tela em `a.hScr`), sons sintetizados (`SFX`) e balão de fala (`say`).
10. **ferramentas e toque** — `TOOLS`, moedas salvas, `pickAt` (raio contra as malhas), `armAt`
    (toque nas mãos), `strike` (o golpe e todas as reações), gestos de toque e a barra de giro.
11. **laço** — câmera (`updCam`), `resize`, `drawMesh` e `frame` (física → rosto → desenho).
    A lista de desenho `L` tem itens `[malha, matriz, modo, brilho, tinta, espessura do contorno,
    sem contorno, só contorno]`.

Depois do laço: cor da pele, saúde salva (`healAll`), moedas e loja, e a partida (`setChar`,
`requestAnimationFrame(frame)`).

## Dados salvos no aparelho (localStorage)

`jb3` (som ligado/desligado), `joao_roupa`, `joao_pele` e `joao_pele_v`, `joao_sangue`
(`'0'` desliga o sangue), `joao_moedas`, `joao_armas`, `joao_saude` (machucados e dentes).

## Dicas para mexer

- Para testar, abra o `index.html` num navegador (de preferência no modo celular das
  ferramentas de desenvolvedor). Coloque `#fps` no fim do endereço para ver quadros por segundo.
- Antes de apagar algo, procure o nome no arquivo inteiro (JavaScript, shaders e HTML).
- Mudanças em uniforms precisam aparecer em três lugares: declaração no shader, lista `U` e a
  chamada `gl.uniform*` no `frame`.
