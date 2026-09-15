# Execução e contratos

## Iniciar no Windows / PowerShell

Use a raiz do repositório como diretório inicial. Existe um ambiente `.venv`;
confira suas dependências antes de reinstalar. Para iniciar o backend:

```powershell
Set-Location app
..\.venv\Scripts\python.exe app.py
```

Em outro terminal, na raiz, sirva apenas a pasta do overlay:

```powershell
.venv\Scripts\python.exe -m http.server 8000 --bind 127.0.0.1 --directory overlay
```

Abra `http://127.0.0.1:8000/`. O OBS também pode abrir `overlay/index.html`
diretamente como arquivo local. O HTML começa com dados fictícios e usa
Socket.IO 4.2.0 e fontes externas por CDN. O backend não serve o frontend;
um 404 na raiz da porta 4445 não indica falha do servidor.

O entrypoint permite Werkzeug para desenvolvimento local, sem reloader,
mantendo bind em loopback. Não é configuração de publicação em produção.

## Dependências

`requirements.txt` contém pins legados; não suponha que foi instalado por inteiro.
O servidor importa Flask, Flask-Cors, Flask-SocketIO, requests, urllib3 e prettytable.
A captura também exige OpenCV, NumPy, pywin32 e EasyOCR (incluindo seu runtime
e modelos). `pytesseract` aparece no módulo de créditos; o README descreve
Tesseract como alternativa, não como troca automática do OCR.

Na verificação de 14/09/2026, foram instalados OpenCV, NumPy, pywin32, EasyOCR
e os modelos iniciais do EasyOCR na `.venv`. `pywinpty==0.5.7` não tem wheel
para Python 3.11 e falha sem Microsoft C++ Build Tools; ele não é usado pelo
overlay e não bloqueia a captura.

## Fluxo HTTP e Socket.IO

1. `/start_match` (GET/POST) busca a partida e dispara `python .\components\get_live_frames.py`.
   Esse comando usa `python` do PATH: para captura, ative a `.venv` no processo
   pai ou corrija o lançamento para usar `sys.executable` quando necessário.
2. A captura emite `new_event` com `{event: deteccoes}`.
3. O servidor agrega o evento e transmite `receive_details` com
   `{match_details: estado}`; é este evento que o frontend escuta.
4. `/get_match_details` (GET/POST) retorna `{response: estado}`.
5. `/stop_match` (GET/POST) emite `kill_self` e restaura o estado inicial.
6. `/register_events` aceita JSON `{events: deteccoes}` e atualiza o estado,
   mas não tem a mesma transmissão do handler Socket.IO.
7. `/edit_team_details` está sem implementação.

O estado contém `blue` e `red`, dicionários indexados por agente. Os jogadores
incluem `agent`, `name`, `alive`, `health` e dados de classificação; arma, escudo
e pontos de ultimate aparecem conforme a agregação. O estado inicial é
`{red: {}, blue: {}, events: []}`. Há exemplos em constants.py com outros
formatos: não os confunda com o contrato efetivamente utilizado.

## Limitações conhecidas no código

- A captura procura a janela pelo título literal `VALORANT  ` (dois espaços).
- Sem jogo e partida ativa, iniciar a captura não é uma verificação válida.
  `get_region` e `get_current_version` podem permanecer em loop se os marcadores
  esperados não existirem no log; evite chamar start_match como health check.
- Armas, escudos e ultimates agregados vêm da parte superior do scoreboard;
  o atualizador aplica esses dados ao lado azul. Vida vermelha é 100 ou 0.
- A detecção de spike existe, mas não é incorporada ao estado por MatchUtils;
  o placar é um placeholder no detector e inicia em [0, 0]. Créditos não são integrados.
- Iniciar várias vezes pode criar vários capturadores. Não há controle robusto
  de processo nem descoberta automática de início/fim de partida.
- Conexão inicial do overlay não busca um snapshot: ele espera receive_details.
- Clientes recentes registram a versão já com `-shipping-` e colocam
  `sessionLoopState` dentro de `matchPresenceData`; preserve os dois formatos.

## Validação proporcional

Para execução básica, verifique HTTP 200 de `/get_match_details`, HTML e assets
na porta 8000, e handshake `/socket.io/?EIO=4&transport=polling` na porta 4445.
Para mudanças de detecção, use imagens em `app/test_images` adequadas à resolução
e inspecione resultados por módulo. Para captura real, confirme jogo aberto,
dependências, resolução e recepção de atualizações pelo frontend. Informe
explicitamente se esse último estágio não foi validado.
