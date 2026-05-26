# AXCHYS™

> **Soluções construídas com precisão para quem não aceita o suficiente.**

A AXCHYS desenvolve software interno de alto padrão para empresas que precisam de ferramentas reais — não demos bonitas. Operamos em dois universos distintos: infraestrutura de TI e gestão empresarial, com integração nativa ao WhatsApp em ambos os produtos.

---

## Produtos

### 01 — Manutenção
**Sistema interno de chamados, inventário e monitoramento de infraestrutura de TI**

Para equipes de TI que precisam de visibilidade total sobre chamados, equipamentos e estações de trabalho. Construído em Python (Flask), com banco de dados SQL Server e monitoramento em tempo real de telas.

### 02 — 3Q
**Plataforma de gestão corporativa — tarefas, metas, reuniões e rotinas**

Para empresas que precisam de clareza hierárquica sobre o que cada pessoa deve fazer e quando. Construído com React + TypeScript + Tailwind CSS, com controle de acesso por organograma.

---

## Manutenção — Detalhamento

### Chamados & Dashboard
- Dashboard de chamados com integração direta ao SQL Server
- Consulta por intervalo de datas com filtros avançados
- Histórico completo de atendimentos por setor e técnico
- Resposta e encerramento de chamados pelo sistema

### Inventário de Hardware

**Computadores**
- Cadastro por setor com hostname, MAC address e IP local
- Periféricos vinculados: até 3 monitores (polegadas, conexão, touch), mouse, teclado, impressora
- Preview SVG dinâmico gerado automaticamente para cada setup
- Campos de número de série, observações técnicas e status operacional

**Impressoras**
- Tipos suportados: Laser, Jato de tinta, Matricial, Térmica, Multifuncional
- Conexão: Rede, USB, Wi-Fi, Bluetooth, Compartilhada
- Campos: IP local, MAC, número de série, funções extras, colorida (sim/não), observações

**Nobreaks**
- Campos: potência VA/W, número de tomadas, tensão de entrada/saída
- Status de bateria: OK / Atenção / Substituir / Falha
- Data de instalação e número de série

### Monitoramento em Tempo Real
- Streaming MJPEG das telas das estações — **720p a 60 FPS**
- Suporte a 1080p em redes gigabit (configurável via variável de ambiente)
- Buffer TCP de 4 MB com codec JPEG otimizado para rede corporativa
- Agente leve instalado nas estações (`nas_maquinas.py` em background)
- Painel multi-tela para monitorar toda a rede de uma vez

### Personalização por Usuário
- Tema claro/escuro com alternância via `Ctrl+Q`
- Painel de opções individual — cada usuário escolhe quais módulos aparecem na navegação
- Cor de destaque personalizável (atualiza efeitos, sombras, badges e destaques)
- Som de clique ativável via arquivo `static/sounds/click.mp3`
- Todas as preferências salvas em `localStorage` — independentes por pessoa/navegador

### Integração WhatsApp — Manutenção
- **Abertura de chamados** pelo WhatsApp — o usuário descreve o problema e o ticket é criado automaticamente
- **Notificações de status**: chamado recebido → em atendimento → encerrado
- **Alertas críticos** de nobreak com bateria em estado Atenção ou Falha
- **Consulta de inventário** pelo chat: "qual o IP da impressora do RH?"
- **Acesso remoto sob demanda** — técnico solicita visualização da tela via WhatsApp
- Notificação para técnico quando novo chamado é aberto em seu setor
- **Relatório diário** de chamados abertos/fechados enviado automaticamente ao gestor
- **Prioridade máxima** disparada por mensagem de urgência
- Compatível com WhatsApp Business API — número dedicado, sem exposição pessoal

---

## Como rodar — Manutenção

### Servidor

```bash
pip install -r requirements.txt
python server.py
```

Acesse: `http://localhost:7575`

> Atalho **Ctrl+Q** em qualquer página alterna entre tema claro e escuro.

### Agente (instalado nas estações)

```bash
pip install psutil requests opencv-python dxcam numpy
python nas_maquinas.py
```

### Variáveis de ambiente do agente

| Variável | Padrão | Descrição |
|---|---|---|
| `MANUTENCAO_SERVER` | auto | URL completa do servidor |
| `MANUTENCAO_FRAME_WIDTH` | 1280 | Largura do frame (320–1920). 1280 = 720p |
| `MANUTENCAO_JPEG_QUALITY` | 62 | Qualidade JPEG (20–95) |
| `MANUTENCAO_TARGET_FPS` | 60 | FPS alvo (1–120) |

**Exemplo para 1080p (rede gigabit):**
```bat
set MANUTENCAO_FRAME_WIDTH=1920
set MANUTENCAO_JPEG_QUALITY=55
python nas_maquinas.py
```

**Exemplo para Wi-Fi / redes mais lentas:**
```bat
set MANUTENCAO_FRAME_WIDTH=960
set MANUTENCAO_JPEG_QUALITY=55
python nas_maquinas.py
```

### Liberação de firewall (Windows)

```bat
netsh advfirewall firewall add rule name="Manutencao Flask 7575" dir=in action=allow protocol=TCP localport=7575
netsh advfirewall firewall add rule name="Manutencao Tela TCP 7576" dir=in action=allow protocol=TCP localport=7576
```

### Estrutura do projeto — Manutenção

```
manutencao/
├── server.py                    # Flask app (rotas + agente TCP de tela)
├── nas_maquinas.py              # Agente das estações (720p @ 60 FPS)
├── DESKTOP COTEFI.json          # Base de computadores
├── IMPRESSORAS.json             # Base de impressoras (criado automaticamente)
├── NOBREAKS.json                # Base de nobreaks (criado automaticamente)
├── requirements.txt
├── iniciar_servidor.bat
├── iniciar_cliente.bat
├── static/
│   ├── style.css
│   ├── common.js                # Tema, toast, atalhos, gerenciador de painéis
│   ├── script.js                # Editor de computadores (com periféricos)
│   ├── setup_svg.js             # Renderizador do preview SVG
│   ├── setor_mapa.js
│   └── sounds/
│       └── click.mp3            # (adicionar manualmente para ativar som)
└── templates/
    ├── base.html
    ├── manut.html               # / — Dashboard de chamados
    ├── select.html              # /select — Lista de computadores
    ├── index.html               # /edit — Editor de computador
    ├── monitor.html             # /monitor — Painel de telas
    ├── opcoes.html              # /opcoes — Painel de opções
    ├── impressoras.html         # /impressoras
    ├── edit_impressora.html     # /edit_impressora
    ├── nobreaks.html            # /nobreaks
    ├── edit_nobreak.html        # /edit_nobreak
    ├── setores.html
    ├── setor_mapa.html
    └── resposta.html
```

### Rotas — Manutenção

| Método | Rota | Descrição |
|---|---|---|
| GET | `/` | Dashboard de chamados (SQL Server) |
| POST | `/consulta` | Consulta chamados por intervalo |
| GET | `/select` | Lista de computadores |
| GET | `/edit?id=<n>` | Editor de computador |
| GET | `/get/<id>` | JSON de um computador |
| POST | `/save/<id>` | Salva alterações |
| POST | `/novo` | Cria computador |
| POST | `/excluir/<id>` | Exclui computador |
| GET | `/lista_ids` | Lista de IDs |
| GET | `/monitor` | Painel de telas |
| GET | `/api/live/stream/<id>` | Stream MJPEG da tela |
| POST | `/api/agent/update` | Update do agente |
| POST | `/api/agent/frame` | Upload de frame |
| GET | `/api/ping` | Health check |
| GET | `/opcoes` | Painel de opções |
| GET/POST | `/impressoras` | Lista e CRUD de impressoras |
| GET/POST | `/nobreaks` | Lista e CRUD de nobreaks |

---

## 3Q — Detalhamento

### Tarefas & Delegação
- Criação e delegação de tarefas com prazos e responsáveis definidos
- Acompanhamento de status em tempo real por gestor e executor
- Histórico completo de alterações e comentários por tarefa
- Prioridades, tags e filtros por setor, pessoa ou prazo
- Dashboard com visão geral de todas as tarefas da equipe

### Metas & Resultados
- Metas corporativas com sub-metas encadeadas por hierarquia
- Barra de progresso visual com atualização em tempo real
- Vinculação de tarefas a metas — cada entrega conta no resultado
- Relatórios de performance por setor, período e responsável

### Reuniões & Atas
- Agendamento com participantes e pauta predefinida
- Geração automática de atas com itens discutidos e decisões tomadas
- Controle de acesso às atas por cargo (organograma)
- Encaminhamentos registrados com responsáveis e prazos direto da ata

### Rotinas & Checklists
- Rotinas e checklists diários/semanais por setor e cargo
- Confirmação de execução com registro de horário e responsável
- Alertas automáticos para rotinas não realizadas no prazo

### Organização & Acesso
- Controle de acesso por cargo — organograma define visibilidade e permissões
- Autenticação segura com sessão protegida por usuário
- Múltiplos setores e hierarquias numa mesma organização
- Tema claro e escuro com preferência salva por usuário

### Integração WhatsApp — 3Q
- **Criação de tarefas por mensagem**: "cria tarefa para João: revisar relatório até sexta"
- **Agendamento de reuniões** — data, hora e participantes configurados pelo chat
- **Alertas de vencimento**: notificação 24h e 1h antes do prazo
- **Aprovação de tarefas** respondendo diretamente no WhatsApp ("aprovado" / "ajustar")
- **Check-in de rotinas** sem abrir o sistema — "rotina ok" registrado com horário
- **Envio automático de atas** para todos os participantes ao encerrar a reunião
- **Consulta de progresso**: "qual o status da meta do setor comercial?"
- **Alertas de metas em atraso** enviados para o responsável e gestor direto
- **Notificação de novas tarefas** delegadas — imediata no WhatsApp
- Compatível com WhatsApp Business API — número dedicado, sem exposição pessoal

### Stack — 3Q
- **Frontend**: React 18 + TypeScript + Tailwind CSS + shadcn/ui
- **Build**: Vite
- **Roteamento**: React Router
- **Estado**: TanStack Query

### Como rodar — 3Q

```bash
# Instalar dependências
npm install

# Desenvolvimento
npm run dev

# Build de produção
npm run build
```

---

## Integração WhatsApp — Visão Geral

Ambos os produtos utilizam integração via **WhatsApp Business API** através de um número dedicado fornecido no momento da implantação. A integração é bidirecional:

- **Entrada**: usuários enviam mensagens para o número do sistema e as ações são processadas automaticamente
- **Saída**: o sistema envia notificações, alertas, relatórios e atas proativamente para usuários e gestores

Nenhuma conta pessoal de WhatsApp é utilizada ou exposta.

---

## Status dos produtos

| Produto | Status |
|---|---|
| Manutenção | Em refatoração ativa |
| 3Q | Em reconstrução (nova interface e arquitetura) |
| Integração WhatsApp | Planejada — integração completa prevista em ambos |

---

## Sobre a AXCHYS

A AXCHYS acredita que bom software não impressiona em demos — resiste ao uso diário. É o tipo de ferramenta que ninguém nota quando funciona, e todo mundo sente falta quando some.

Operamos na fronteira entre técnica e gestão porque é lá que as maiores ineficiências vivem. É lá que construímos.

---

*© 2025 AXCHYS. Todos os direitos reservados.*
