# AGENTS.md — Fúria no Asfalto

Jogo arcade de corrida de combate em motos, em pseudo-3D (projeção por segmentos, no estilo dos racers 2D clássicos). É um jogo original: não copie nomes, personagens nem visuais de jogos existentes.

## Estrutura
- `index.html`: o jogo inteiro (HTML, CSS e JS num único `<script>`). Não tem build, dependências nem assets externos: todos os sprites, fundos e sons são gerados por código.
- `README.md`: instruções para o jogador, em pt-BR.
- `.agents/skills/vercel-react-best-practices`: skill instalado via `npx skills` (não tem relação com o jogo, que não usa React).

## Rodar
Abra o `index.html` no navegador, ou rode `python3 -m http.server 8000`.

## Idioma
Todo texto voltado ao jogador (UI, HUD, README) é em **português do Brasil**.

## Arquitetura do `index.html` (na ordem em que aparece no arquivo)
1. **Utils/constantes**: o canvas interno tem 1280x720, escalado para caber na janela. `SEG=200`, `ROAD_W=2000` (meia largura da pista), `DRAW_DIST`, `CAM_H`, `PLAYER_Z`, `BASE_MAX = SEG/STEP` (velocidade de referência, 12000 u/s), `BIKE_W`, `PW` (largura de colisão), `CENTRIFUGAL`.
   - Posição lateral `x` em unidades de pista: `|x| = 1` é a borda do asfalto e `|x| > 1` é fora da pista.
   - Posições ao longo da pista: `d` é a distância absoluta (rivais e jogador) e `z = mod(d, L)` é a posição dentro da volta.
2. **Dados**: `WEAPONS`, `PLAYER_LOOK`, `RIVAL_DEFS` (7 rivais com `skill`, `aggr` e `weapon`), `PRIZES`, `KO_BONUS`, `UPGRADES` (motor, pneus, blindagem e nitro, com 3 níveis cada).
3. **Save**: fica no `localStorage` com a chave `furia-asfalto-v1`, no formato `{money, up, unlocked, best}`.
4. **`Snd`**: áudio WebAudio sintetizado. Tem um motor (osciladores mais filtro passa-baixa), SFX (`noise`/`tone`) e um sequenciador de música. Só é iniciado depois de um gesto do usuário (`Snd.init()`).
5. **Entrada**: `kb`, `pad`, `tch` e `pressed`. `inp(acao)` diz se a ação está segurada; `pressed[acao]` marca o toque único e é limpo depois de cada passo de update. Ações disponíveis: `up`, `down`, `left`, `right`, `atkL`, `atkR`, `nitro`, `ok`, `pause`, `mute`, `garage`, `quit`, `full`. Há suporte a gamepad e a botões de toque (o `#touch` no HTML).
6. **Sprites**: pré-renderizados em canvas offscreen pelas funções `*Img()`. São registrados em `SPR` via `defSprite(key, img, larguraMundo, fracaoColisao, ancoraX)`. Os carros ficam em `CARS` (vista traseira e frontal).
7. **Temas (`THEMES`)**: `desert`, `city` e `canyon`, nessa ordem (`THEME_ORDER`, que também é a ordem de desbloqueio). Cada tema define cores da pista, neblina, número de faixas, `diff` (multiplicador da velocidade dos rivais), `prizeMul`, os parâmetros de geração `track`, a densidade de `traffic` e as funções `sky()`, `layers()` (parallax) e `place()` (cenário por segmento).
8. **Pista**: `buildTrack(key)` gera os segmentos com `addRoad(enter, hold, leave, curve, hill)`, usando um RNG com semente fixa (é determinístico por tema). A geração:
   - termina a pista com `y = 0`;
   - põe as faixas de LARGADA (segmento 30) e de CHEGADA (`length - 40`);
   - coloca as placas de curva (chevrons);
   - cria o tráfego com `makeTraffic()`. As faixas da metade esquerda são contramão (`dir = -1`).
9. **Corrida**:
   - `startRace(key)`: monta o grid de 8 posições, com o jogador no slot 5.
   - `updatePlayer`: física, nitro, ataques e colisões com cenário, tráfego e rivais.
   - `updateRival`: IA com rubber band, desvio do tráfego, "engage" (persegue o jogador) e ataques. O ritmo dos ataques é global (`R.nextRivalAtk`), para que os rivais não ataquem todos de uma vez.
   - `crashPlayer`, `hitPlayer`, `rivalDown` (nocaute, bônus e roubo da arma), `placeOf`, `endRace`.
   - Os ataques usam `ext(p)` para animar a preparação e o golpe; o acerto é verificado em `p ≈ 0.55–0.6`.
10. **Render**:
    - `renderWorld(camZ, camX, drawPlayer)` desenha, nesta ordem: céu, camadas de parallax, segmentos de frente para trás (com clip por `maxy`), e por fim sprites e entidades de trás para frente.
    - O jogador é inserido na ordem certa de profundidade.
    - As motos são desenhadas proceduralmente por `drawBike`, `drawDowned` e `drawRagdoll`, num espaço local de ~100 unidades de largura com origem na roda.
11. **HUD e telas**: `drawHUD`, menu (com a pista em modo demo ao fundo), garagem, pausa e resultados.
    - Estados possíveis em `game.state`: `menu`, `garage`, `race`, `pause`, `results`.
12. **Loop**: passo fixo de 1/60 s com acumulador, e `render()` a cada frame.

## Equilíbrio (já ajustado via simulação)
- Dificuldade por tema (`diff`): deserto 0.91, cidade 0.98, cânion 1.03.
- Densidade de tráfego (`traffic`, 1 carro a cada N segmentos): deserto 75, cidade 55, cânion 95.
- Dano na moto por queda: `(10 + 12·vel) · (1 − 0.13·blindagem)`. Com 100 de dano, a moto é destruída.
- Bater na traseira de um carro só derruba se a velocidade relativa for maior que `0.42·BASE_MAX`. Bater em carro na contramão sempre derruba.
- O cânion foi pensado para exigir upgrades.

## Testes
Não há suíte de testes. Para validar:
- **Sintaxe**: extraia o `<script>` e rode `node --check`.
- **Smoke test**: Playwright com Chromium, instalado fora do repo (em um scratchpad). Carregue `file:///workspaces/moto/index.html`, capture `pageerror` e tire screenshots. Use `page.evaluate` para chamar `startRace('city')` etc.
- **Simulação rápida**: dentro de `page.evaluate`, chame `update(STEP)` em loop com um bot que mexe em `kb`/`pressed`, e meça tempo de prova, posição, quedas e golpes recebidos.

## Convenções
- Mantenha tudo no arquivo único e sem dependências externas.
- Para mudar o equilíbrio, prefira ajustar as constantes e os campos dos temas.
- Sempre faça commit e push (em `origin/main`) ao concluir cada alteração — pedido explícito do usuário.
