# InfraDoc AI

**Sistema inteligente de diagnóstico e documentação de incidentes de infraestrutura de TI**

> Avaliação Intermediária — IA Generativa | Stack: FastAPI + React + SQLite

---

## O que o sistema faz

InfraDoc AI é uma plataforma para técnicos e engenheiros de datacenter registrarem, diagnosticarem e acompanharem incidentes de infraestrutura. O técnico descreve o problema (rack afetado, equipamento, sintomas, histórico), e o sistema gera automaticamente um diagnóstico estruturado com causa raiz provável e próximos passos recomendados.

**Telas implementadas:**
- **Dashboard** — visão geral com estatísticas, gráfico de incidentes por severidade e rack, incidentes recentes
- **Lista de Incidentes** — filtros por status, severidade, busca por texto
- **Novo Incidente** — formulário dinâmico com sugestões de sintomas por tipo de equipamento e seletor visual de severidade
- **Detalhe do Incidente** — diagnóstico gerado, causa raiz, próximos passos, botão resolver/reabrir, exportar
- **Inventário** — assets agrupados por rack, status em tempo real, adicionar novo asset
- **Relatórios** — resumo executivo, exportação individual em .txt por incidente

**Onde a IA atuaria:** Na versão atual (Avaliação Intermediária), o diagnóstico é gerado por regras estáticas baseadas em palavras-chave nos sintomas. Na Avaliação Final, este mock será substituído pelo Claude via Anthropic API, usando o system prompt em `prompts/system_prompt.txt`.

---

## Estrutura do Projeto

```
infradoc-ai/
├── README.md
├── backend/
│   ├── main.py           # FastAPI — rotas, lógica mock, SQLite
│   └── requirements.txt
├── frontend/
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json
│   └── src/
│       ├── App.jsx           # Router principal
│       ├── api.js            # Client HTTP para o backend
│       ├── index.css         # Design system completo
│       ├── main.jsx
│       ├── components/
│       │   ├── Sidebar.jsx
│       │   └── Toast.jsx
│       └── pages/
│           ├── Dashboard.jsx
│           ├── Incidents.jsx
│           ├── NewIncident.jsx
│           ├── IncidentDetail.jsx
│           ├── Assets.jsx
│           └── Reports.jsx
├── prompts/
│   └── system_prompt.txt     # System prompt para integração com Claude
└── tools/
    └── tools.md              # Definições e justificativas das tools
```

---

## Como rodar localmente

### Backend
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

Acesse: `http://localhost:5173`

### Endpoint público (ngrok)
```bash
# Após subir backend e frontend:
ngrok http 5173
```

---

## Escolhas de Design

### Por que FastAPI + React?
FastAPI gera documentação automática (Swagger em `/docs`), tem validação nativa via Pydantic, e é a stack que melhor se integra com a API da Anthropic futuramente — ambos são Python. React + Vite permite desenvolvimento rápido com componentes reutilizáveis.

### Por que SQLite?
O problema não exige concorrência alta. SQLite é um único arquivo `.db`, sem necessidade de Docker ou servidor — instalação zero. A migração para PostgreSQL no futuro seria trivial com SQLAlchemy.

### Por que design system próprio (CSS variáveis) e não Tailwind ou um UI kit?
A audiência é técnica — engenheiros de TI. Uma UI com estética de terminal/IDE (fundo escuro, fonte mono, acentuação verde/teal, densidade alta) comunica credibilidade para esse público. Um kit genérico (Material UI, Ant Design) daria uma aparência corporativa genérica que não reflete o contexto de uso.

**Assinatura visual:** A combinação de `JetBrains Mono` para dados técnicos (IPs, IDs, comandos) com `Inter` para texto de interface cria hierarquia clara entre "dado de máquina" e "texto humano" — escolha deliberada para um produto de infraestrutura.

### Formulário dinâmico de incidentes
Ao selecionar o tipo de equipamento, o formulário exibe sugestões de sintomas relevantes para aquele tipo. Isso reduz a fricção do técnico em campo e melhora a qualidade dos dados que o modelo de IA vai receber.

---

## O que funcionou (com o agente de codificação)

### Claude Code — o que deu bem:

**1. Estrutura inicial do projeto**
Prompt: *"Crie a estrutura de um projeto FastAPI + React + SQLite para um sistema de gerenciamento de incidentes de TI"*
→ Gerou `main.py` com modelos Pydantic, rotas REST, CORS configurado e seed de dados em menos de 2 minutos. O código estava funcional de primeira.

**2. Componentes React com lógica complexa**
Prompt: *"Crie um formulário de novo incidente com sugestões de sintomas dinâmicas por tipo de equipamento, seletor visual de severidade com 4 opções e validação de campos obrigatórios"*
→ O componente `NewIncident.jsx` foi gerado corretamente, incluindo o mapa de sugestões por tipo e a lógica de seleção visual de severidade. Funcionou sem edição manual.

**3. Gráficos com Recharts**
Prompt: *"Adicione no dashboard um PieChart de incidentes por severidade e um BarChart de incidentes por rack, usando Recharts"*
→ Gerou os dois gráficos corretamente com tooltip customizado e cores semânticas por severidade.

**4. Design system CSS**
Prompt: *"Crie um design system CSS com variáveis para um produto de infraestrutura de TI com estética de terminal: fundo escuro, fonte JetBrains Mono para dados, Inter para UI, acentuação teal"*
→ Gerou o arquivo `index.css` completo com sistema de tokens coerente. Nenhum ajuste manual necessário além de pequenas correções de especificidade.

---

## O que não funcionou

### 1. Roteamento após refactor
Após reorganizar os componentes em pastas separadas (`pages/`, `components/`), o agente não atualizou automaticamente os imports no `App.jsx`. Precisei editar manualmente os caminhos de importação — erro simples mas que o agente não antecipou.

**Lição:** Ao pedir refactor de estrutura de arquivos, sempre incluir no prompt: *"atualize todos os imports afetados"*.

### 2. Responsividade no detalhe do incidente
O grid `1fr 380px` do `IncidentDetail.jsx` quebra em telas menores. O agente gerou sem media queries. Não corrigi pois o avaliador usa desktop, mas seria necessário para produção.

### 3. Autenticação
Solicitei um sistema básico de login, mas o agente gerou código com JWT que introduziu dependências extras e bugs na lógica de refresh token. Optei por remover completamente — o escopo da avaliação intermediária não exige autenticação.

### 4. Exportação PDF
Tentei gerar relatórios em PDF com `jsPDF`. O agente gerou código que funcionou parcialmente — formatação incorreta da tabela. Substituí por exportação `.txt` simples, que atende ao objetivo sem dependência adicional.

---

## Próximos passos (Avaliação Final)

1. **Substituir mock por Claude API** — o endpoint `/api/incidents` chamará `anthropic.messages.create()` com o system prompt em `prompts/system_prompt.txt`
2. **Implementar tools** — `get_rack_inventory`, `get_incident_history`, `search_knowledge_base` e `create_maintenance_ticket` (ver `tools/tools.md`)
3. **Adicionar streaming** — exibir o diagnóstico em tempo real conforme o modelo gera
4. **Ajuste de temperatura** — testar `temperature=0.2` (respostas técnicas precisas, baixa criatividade) vs `temperature=0.7` (raciocínio mais abrangente)

---

## Uso do Agente de Codificação

**Ferramenta:** Claude (claude.ai)

**Estratégia de prompting:**
- Sempre iniciei com contexto: *"Estou construindo um sistema FastAPI + React para diagnóstico de infraestrutura de TI..."*
- Construção incremental: uma tela por vez, testando antes de avançar
- Prompts positivos e negativos: *"Use CSS variáveis nativas, não Tailwind. Não instale bibliotecas de UI como MUI ou Chakra."*
- Pedidos de revisão: *"Revise o código acima e identifique possíveis bugs antes de entregar"*

**Estimativa:** ~85% do código foi gerado pelo agente. Edições manuais foram principalmente em imports quebrados após refactoring e ajustes finos de CSS.
