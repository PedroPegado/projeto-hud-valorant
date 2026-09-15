---
name: live-valorant-overlay
description: Entender, executar e modificar este projeto Live-Valorant-Overlay, incluindo backend Flask, captura Win32, processamento de imagens e overlay para OBS.
---

# Live Valorant Overlay

Skill local deste repositório. Resolva os caminhos abaixo a partir da raiz do
projeto (dois níveis acima desta pasta). Confira o código e o estado do Git
antes de aplicar o contexto: esta documentação retrata uma implementação legada.

## Contexto e navegação

É uma prova de conceito para transmitir informações de partidas de VALORANT
para uma fonte de navegador do OBS. Não há Node, build de frontend nem banco.
O backend é Python/Flask/Flask-SocketIO; o frontend é HTML/CSS/JS puro.

- `app/app.py`: servidor HTTP e Socket.IO na porta 4445; estado em memória.
- `app/components/get_corematch.py`: dados iniciais de jogadores pelas APIs
  local e remotas do jogo, usando logs e lockfile do Riot Client.
- `app/match_utils.py`: inicializa jogadores, troca lados e incorpora detecções.
- `app/components/get_live_frames.py`: captura Win32, aproximadamente 1 fps.
- `app/components/live_details.py`: coordena os detectores de imagens.
- `app/components/`: módulos de agentes, vida, armas, escudos, ultimates,
  placar, spike e créditos; leia os módulos relevantes antes de alterar recortes.
- `app/constants.py`: estruturas de exemplo e custos de ultimates.
- `app/templates/`: imagens usadas pelo template matching.
- `app/test_images/`: screenshots de exemplo; não são uma suíte automatizada.
- `overlay/index.html`, `overlay/app.js`, `overlay/overlay.css`: layout, atualização
  e estilos. As demais pastas do overlay contêm imagens de exibição.

Leia [references/runtime.md](references/runtime.md) para iniciar, diagnosticar
ou validar a execução e para consultar contratos e limitações.

## Cuidados específicos

As coordenadas e os templates dependem da resolução. Este checkout captura
1920×1200; a menção a 1920×1080 no README aponta para outra branch.
O modo documentado é Fullscreen Windowed com Fill. Alterar só a dimensão da
captura não adapta os recortes e templates.

Os identificadores de agentes são minúsculos e `kay/o` é normalizado para
`kayo`. Mantenha compatibilidade entre constantes, templates, payloads e assets.
Não trate as listas antigas de agentes e custos como dados atuais do jogo.
Não copie tokens, senhas do lockfile ou headers de autenticação para a skill.

Atualize esta skill quando uma mudança alterar arquitetura, comandos ou
contratos. Diferencie servidor acessível, overlay demonstrativo e captura real
validada: são resultados distintos.
