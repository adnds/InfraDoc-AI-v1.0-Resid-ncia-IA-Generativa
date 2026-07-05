# InfraDoc AI

**Sistema inteligente de diagnóstico e documentação de incidentes de infraestrutura de TI**

> Avaliações Intermediária e Final — IA Generativa | Stack: FastAPI + React + SQLite

---

## O que o sistema faz

InfraDoc AI é uma plataforma para técnicos e engenheiros de datacenter registrarem, diagnosticarem e acompanharem incidentes de infraestrutura. O técnico descreve o problema (rack afetado, equipamento, sintomas, histórico), e o sistema gera automaticamente um diagnóstico estruturado com causa raiz provável e próximos passos recomendados.

**Telas implementadas:**
- **Login / Criar Conta** — autenticação com e-mail obrigatório, senha, recuperação simulada
- **Dashboard** — visão geral com estatísticas, gráfico de incidentes por severidade e rack, incidentes recentes
- **Lista de Incidentes** — filtros por status, severidade, busca por texto
- **Novo Incidente** — formulário dinâmico com sugestões de sintomas por tipo de equipamento e seletor visual de severidade
- **Detalhe do Incidente** — diagnóstico gerado, causa raiz, próximos passos; encerramento exige nota obrigatória
- **Inventário** — assets agrupados por rack, status automático por severidade, adicionar novo asset
- **Relatórios** — resumo executivo, solicitação de exportação com aprovação do admin
- **Base de Conhecimento** — artigos de soluções documentadas pela equipe, busca, avaliação de utilidade
- **Usuários** — gerenciamento de contas (admin), perfis Técnico e Administrador
- **Solicitações de Exportação** — fluxo de aprovação admin para exportar relatórios

**Onde a IA atuaria:** Na versão atual (Avaliação Intermediária), o diagnóstico é gerado por regras estáticas baseadas em palavras-chave nos sintomas. Na Avaliação Final, este mock será substituído pelo Claude via Anthropic API, usando o system prompt em `prompts/system_prompt.txt`.

---

## Estrutura do Projeto

```
infradoc-ai/
├── README.md
├── backend/
│   ├── main.py               # FastAPI — rotas, lógica mock, SQLite
│   └── requirements.txt
├── frontend/
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json
│   └── src/
│       ├── App.jsx               # Router principal + controle de autenticação
│       ├── api.js                # Client HTTP para o backend
│       ├── index.css             # Design system completo
│       ├── main.jsx
│       ├── components/
│       │   ├── Sidebar.jsx       # Navegação com info do usuário logado
│       │   └── Toast.jsx         # Notificações globais
│       └── pages/
│           ├── Login.jsx         # Login, cadastro e recuperação de senha
│           ├── Dashboard.jsx
│           ├── Incidents.jsx
│           ├── NewIncident.jsx
│           ├── IncidentDetail.jsx # Com modal de nota obrigatória ao encerrar
│           ├── Assets.jsx
│           ├── Reports.jsx        # Com fluxo de solicitação de exportação
│           ├── KnowledgeBase.jsx  # Base de conhecimento da equipe
│           ├── Users.jsx          # Gerenciamento de usuários (admin)
│           └── ExportRequests.jsx # Aprovação de exportações (admin)
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

> **Atenção:** se for a primeira vez ou após adicionar novas funcionalidades, delete o arquivo `backend/infradoc.db` para recriar o banco com todas as tabelas atualizadas.

### Frontend
```bash
cd frontend
npm install
npm run dev
```

Acesse: `http://localhost:5173`

**Credenciais padrão:**
```
Usuário: admin
Senha:   admin123
```

### Endpoint público (ngrok)
```bash
ngrok http 5173
```

---

## Funcionalidades em Detalhe

### Autenticação
- Login com usuário e senha
- Cadastro com nome, usuário, e-mail (validado) e senha mínima de 6 caracteres
- Recuperação de senha simulada (mock — não envia e-mail real)
- Sessão salva em `localStorage`; logout limpa a sessão
- Perfis: **Administrador** (acesso total) e **Técnico** (acesso restrito)

### Status Automático de Assets por Severidade
Ao abrir um incidente, o asset correspondente tem seu status atualizado automaticamente:

| Severidade | Status do Asset |
|---|---|
| Crítico | 🔴 Offline |
| Alto | 🔴 Offline |
| Médio | 🟡 Degradado |
| Baixo | 🟡 Degradado |
| Incidente resolvido | 🟢 Online |

### Nota Obrigatória ao Encerrar Incidente
Ao clicar em "Marcar Resolvido", um modal exige que o técnico preencha uma nota de encerramento com mínimo de 10 caracteres descrevendo o que foi feito. Sem a nota, o incidente não pode ser fechado. A nota fica registrada no histórico do incidente.

### Fluxo de Exportação de Relatórios
- **Técnico:** clica em "Solicitar" informando motivo obrigatório (mínimo 15 caracteres)
- **Admin:** recebe a solicitação em "Exportações" no menu, revisa e aprova ou nega com nota
- **Técnico:** após aprovação, botão "Exportar" é liberado para download em `.txt`
- Admin sempre pode exportar diretamente sem solicitação

### Base de Conhecimento
- Artigos de soluções documentadas pela equipe, organizados por tipo de equipamento
- Busca por título, sintoma ou palavra-chave
- Qualquer usuário pode publicar artigos e marcar como útil
- Somente admin pode excluir artigos
- 5 artigos de exemplo pré-cadastrados (RAM, loop de rede, PDU, no-break, firewall)

---

## Escolhas de Design

### Por que FastAPI + React?
FastAPI gera documentação automática (Swagger em `/docs`), tem validação nativa via Pydantic, e é a stack que melhor se integra com a API da Anthropic futuramente. React + Vite permite desenvolvimento rápido com componentes reutilizáveis.

### Por que SQLite?
O problema não exige concorrência alta. SQLite é um único arquivo `.db`, sem necessidade de Docker ou servidor — instalação zero. A migração para PostgreSQL no futuro seria trivial com SQLAlchemy.

### Por que design system próprio e não Tailwind ou UI kit?
A audiência é técnica — engenheiros de TI. Uma UI com estética de terminal/IDE (fundo escuro, fonte mono, acentuação teal) comunica credibilidade para esse público. A combinação de `JetBrains Mono` para dados técnicos com `Inter` para texto de interface cria hierarquia clara entre "dado de máquina" e "texto humano".

### Autenticação no localStorage
Usuários são armazenados no `localStorage` do navegador. Decisão deliberada para manter o projeto sem dependência de backend de autenticação na avaliação intermediária. Na produção real, seria migrado para o banco SQLite com senhas em hash bcrypt.

---

## O que funcionou (com o agente de codificação)

**1. Estrutura inicial do projeto**
Prompt: *"Crie a estrutura de um projeto FastAPI + React + SQLite para um sistema de gerenciamento de incidentes de TI"*
→ Gerou `main.py` com modelos Pydantic, rotas REST, CORS configurado e seed de dados funcional de primeira.

**2. Componentes React com lógica complexa**
Prompt: *"Crie um formulário de novo incidente com sugestões de sintomas dinâmicas por tipo de equipamento, seletor visual de severidade e validação de campos obrigatórios"*
→ `NewIncident.jsx` gerado corretamente com toda a lógica condicional. Funcionou sem edição manual.

**3. Fluxo de aprovação de exportações**
Prompt: *"Implemente um fluxo onde técnico solicita exportação com motivo obrigatório e admin aprova ou nega com nota"*
→ Gerou corretamente as rotas no backend, o modal de solicitação e a tela de revisão do admin.

**4. Status automático de assets por severidade**
Prompt: *"Ao criar incidente crítico ou alto, marcar asset como offline. Médio e baixo como degraded. Resolver incidente volta para online."*
→ Implementado diretamente no endpoint de criação e atualização de incidentes sem erros.

**5. Base de conhecimento completa**
Prompt: *"Crie uma base de conhecimento com tabela no SQLite, rotas CRUD, busca por palavras-chave e contador de visualizações e utilidade"*
→ Gerou tabela, seed com 5 artigos, todas as rotas e o componente React com expansão inline.

---

## O que não funcionou

**1. Roteamento após refactor**
Após reorganizar componentes em pastas, o agente não atualizou os imports automaticamente no `App.jsx`. Edição manual necessária.
**Lição:** Sempre incluir no prompt: *"atualize todos os imports afetados"*.

**2. Responsividade**
O grid `1fr 380px` do `IncidentDetail.jsx` quebra em telas menores. O agente não gerou media queries. Não corrigido pois o avaliador usa desktop.

**3. Exportação PDF**
Tentativa com `jsPDF` gerou código com formatação incorreta. Substituído por exportação `.txt` simples.

**4. Envio real de e-mail**
Solicitado envio de confirmação de cadastro via e-mail. O agente gerou código com SendGrid mas introduziu dependências e complexidade desnecessária para o escopo. Substituído por simulação mock com mensagem de confirmação na tela.

**5. Tabela `export_requests` com schema duplicado**
Na primeira geração, a tabela foi criada duas vezes no `init_db`. O agente não detectou a duplicação. Corrigido manualmente removendo o bloco duplicado.

---

## Próximos passos (Avaliação Final)

1. **Substituir mock por Claude API** — o endpoint `/api/incidents` chamará `anthropic.messages.create()` com o system prompt em `prompts/system_prompt.txt`
2. **Implementar tools** — `get_rack_inventory`, `get_incident_history`, `search_knowledge_base` e `create_maintenance_ticket` (ver `tools/tools.md`)
3. **Novo campo `knowledge_base_suggestion`** — exibir sugestão do Claude para documentar na base de conhecimento ao encerrar incidente
4. **Streaming** — exibir diagnóstico em tempo real conforme o modelo gera
5. **Temperatura 0.2** — validado como ideal para respostas técnicas precisas sem texto fora do JSON

---

## Uso do Agente de Codificação

**Ferramenta:** Claude (claude.ai)

**Estratégia de prompting:**
- Sempre iniciei com contexto: *"Estou construindo um sistema FastAPI + React para diagnóstico de infraestrutura de TI..."*
- Construção incremental: uma funcionalidade por vez, testando antes de avançar
- Prompts positivos e negativos: *"Use CSS variáveis nativas, não Tailwind. Não instale bibliotecas de UI como MUI ou Chakra."*
- Pedidos de revisão: *"Revise o código acima e identifique possíveis bugs antes de entregar"*
- Iteração em erros: *"O agente gerou X mas o comportamento esperado é Y — corrija mantendo o restante intacto"*

**Estimativa:** ~85% do código foi gerado pelo agente. Edições manuais foram em imports quebrados após refactoring, schema duplicado no banco e ajustes finos de CSS.
