# Simplifica GOV

Plataforma de engajamento cívico que torna projetos de lei do Congresso Nacional acessíveis aos cidadãos brasileiros através de tradução por IA e linguagem simples.

## 🚀 Como Rodar o Projeto

### Pré-requisitos
- Node.js 18+ instalado
- npm, yarn ou pnpm

### Passo a Passo
1. **Instalação das dependências:**
   \`\`\`bash
   npm install
   ou
   yarn install
   \`\`\`

2. **Rodar o servidor de desenvolvimento:**
   \`\`\`bash
   npm run dev
   ou
   yarn dev
   \`\`\`

3. **Acessar a aplicação:**
   Abra seu navegador em [http://localhost:3000](http://localhost:3000)

## 💾 Integração com Banco de Dados

Atualmente, o projeto utiliza dados simulados ("mock") localizados em `lib/data.ts`. Para produção, recomenda-se a migração para um banco de dados relacional (PostgreSQL) utilizando um ORM como Prisma ou Drizzle.

### Sugestão de Schema (SQL)

\`\`\`sql
-- Tabela de Parlamentares
CREATE TABLE parliamentarians (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  party TEXT NOT NULL,
  state VARCHAR(2) NOT NULL,
  image TEXT,
  bio TEXT
);

-- Tabela de Projetos de Lei
CREATE TABLE projects (
  id TEXT PRIMARY KEY, -- Ex: PL-1234-2025
  number TEXT NOT NULL,
  title TEXT NOT NULL,
  simplified_title TEXT, -- Gerado por IA
  summary TEXT,
  category TEXT NOT NULL,
  status TEXT NOT NULL, -- Em tramitação, Aprovado, etc.
  house TEXT NOT NULL, -- Câmara, Senado, Bicameral
  urgency BOOLEAN DEFAULT FALSE,
  author_id TEXT REFERENCES parliamentarians(id),
  votes INTEGER DEFAULT 0,
  favorites INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Tabela de Resumos e Impactos (IA)
CREATE TABLE project_ai_analysis (
  project_id TEXT REFERENCES projects(id),
  ai_summary JSONB, -- Lista de pontos simplificados
  impact JSONB -- Lista de impactos diretos
);
\`\`\`

### Passos para Migração
1. **Configurar Variáveis de Ambiente:** Crie um arquivo `.env.local` com sua `DATABASE_URL`.
2. **Instalar ORM:** `npm install prisma --save-dev` (exemplo).
3. **Substituir `lib/data.ts`:** Crie *Server Actions* ou *API Routes* em `app/api/` para buscar os dados do banco ao invés de ler o array estático.

## 🌐 Integração com APIs

### 1. Dados Abertos (Câmara e Senado)
Para manter os projetos atualizados, você deve criar um "Job" (Cron) que consome as APIs oficiais diariamente:

- **Câmara dos Deputados:** [https://dadosabertos.camara.leg.br/swagger/api.html](https://dadosabertos.camara.leg.br/swagger/api.html)
  - Endpoint principal: `/proposicoes` (para listar PLs)
- **Senado Federal:** [https://legis.senado.leg.br/dadosabertos/docs/](https://legis.senado.leg.br/dadosabertos/docs/)

**Fluxo Sugerido:**
1. Job roda às 00:00.
2. Busca novas proposições do dia.
3. Salva no Banco de Dados (`projects`).
4. Dispara gatilho para o "Agente de IA".

### 2. IA Generativa (Simplinho)
O chatbot e os resumos simplificados utilizam o **Vercel AI SDK**.

- **Configuração:** Adicione sua chave de API (OpenAI, Anthropic, etc.) no `.env.local`.
  \`\`\`env
  OPENAI_API_KEY=sk-...
  \`\`\`
- **Geração de Resumos:** Quando um novo PL for salvo no banco, chame a API de IA com um prompt do sistema:
  > "Aja como um especialista em linguagem simples. Resuma o seguinte projeto de lei para um cidadão leigo em 3 tópicos..."

### 3. Autenticação
Para recursos como "Favoritos" e "Alertas", integre com **Auth.js** ou **Clerk**.
- Adicione tabela `users` e `user_favorites` no banco de dados para persistir as preferências do "Mapa de Afetos".
