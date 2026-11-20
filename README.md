# Fúria no Asfalto

Corrida de combate em motos, arcade e em pseudo-3D. Roda num único arquivo (`index.html`), sem dependências.

## Como jogar
Abra o `index.html` no navegador, ou sirva a pasta:

```bash
python3 -m http.server 8000   # depois acesse http://localhost:8000
```

| Tecla | Ação |
|---|---|
| ↑ / W | acelerar |
| ↓ / S | frear |
| ← → / A D | pilotar |
| Shift / Espaço | nitro |
| Z / J | golpe à esquerda |
| X / L | golpe à direita |
| P / Esc | pausar (Q abandona a corrida) |
| G | garagem |
| M / F | som / tela cheia |

Também funciona com gamepad e tem controles na tela para celular.

## Regras
- 8 pilotos por corrida. Termine no TOP 3 para liberar a próxima pista: **Deserto Escaldante → Metrópole Neon → Cânion Vermelho**.
- Derrube rivais (+$150 por nocaute). Quem carrega arma (taco, corrente, cano) deixa ela pra você.
- **VIGOR** zerado = você cai. **MOTO** zerada = moto destruída e fim de corrida.
- Bater em carro, em placa ou em prédio derruba você. Os carros que vêm na contramão não perdoam.
- Com o dinheiro dos prêmios, compre na garagem melhorias de motor, pneus, blindagem e nitro (o progresso fica salvo no navegador).
