# 📋 Relatório Técnico do Projecto — HydroSync (Hydrolean Angola)

> **Versão do documento:** 1.0  
> **Data:** Junho 2026  
> **Repositório:** `C:\Users\Blender_STG\Videos\Captures\Hydrolean---Angola`  
> **Estado actual:** MVP em desenvolvimento activo

---

## 1. Visão Geral e Contexto

### 1.1 Identidade do Produto

O projecto é denominado **HydroSync** (nome de produto) e encontra-se alojado sob a pasta `Hydrolean---Angola`, reflectindo o seu contexto geográfico e a fase inicial de concepção. Trata-se de um **sistema inteligente de irrigação parametrizada**, pensado especificamente para o contexto angolano.

O slogan do produto é:

> *"A água certa, no momento certo."*

### 1.2 Problemática

Em Angola, a grande maioria dos pequenos e médios agricultores recorre a métodos de irrigação manual ou a horários fixos pré-definidos, sem qualquer integração com dados reais do ambiente ou do solo. Esta abordagem ignora variáveis críticas como:

- **Umidade real do solo** — O agricultor não sabe com precisão quando o solo precisa de água.
- **Tipo de cultura** — Diferentes culturas têm necessidades hídricas completamente distintas.
- **Condições climáticas** — A chuva prevista não é considerada antes de activar a irrigação.
- **Necessidade hídrica específica** — Não existe um sistema que calcule volumes adequados.

As consequências directas são:

| Problema | Impacto |
|---|---|
| Desperdício de água | Custos operacionais elevados e esgotamento de recursos |
| Sub-irrigação | Stress hídrico nas plantas e redução da produtividade |
| Sobre-irrigação | Lixiviação de nutrientes e encharcamento do solo |
| Ausência de dados | Impossibilidade de optimizar práticas ao longo do tempo |

### 1.3 Público-Alvo

**Primário:**
- Pequenos e médios agricultores em Angola
- Cooperativas agrícolas
- Projectos comunitários de irrigação

**Secundário:**
- ONGs agrícolas e de desenvolvimento rural
- Projectos governamentais de modernização agrícola
- Iniciativas de agricultura sustentável

---

## 2. Proposta de Valor

O HydroSync oferece um sistema que:

1. **Activa irrigação com base na necessidade real do solo** — leitura de sensores e análise de dados em tempo real.
2. **Ajusta-se ao tipo de cultura** — o modelo de Machine Learning recomenda a cultura ideal para cada conjunto de parâmetros de solo.
3. **Utiliza previsão meteorológica em tempo real** — dados de temperatura, chuva e evapotranspiração via API Open-Meteo.
4. **Gera recomendações via IA Generativa** — o motor DeepSeek (LLM) produz planos de irrigação detalhados em português.
5. **Dashboard interactivo** — visualização do mapa da fazenda, sensores e métricas de eficiência.
6. **É acessível e replicável** — sem hardware proprietário caro; o MVP usa simulação via dashboard web.

### Diferenciais Competitivos

- Parametrização por tipo de cultura (22 culturas suportadas)
- Modelo preditivo de necessidade hídrica (GaussianNB treinado com dataset agronómico)
- Dashboard com mapa interactivo por fazenda (Leaflet.js)
- Base agronómica validada (dataset `Crop_recommendation.csv`)
- Foco exclusivo no contexto angolano e tropical
- Solução de baixo custo e arquitectura cloud (Render — plano gratuito)
- Chatbot AgroIntel com contexto real da fazenda do utilizador

---

## 3. Arquitectura do Sistema

O projecto adopta uma arquitectura **fullstack desacoplada**, com backend e frontend separados e comunicação via API REST.

```
┌─────────────────────────────────────────────────────────────────┐
│                        UTILIZADOR                               │
│                    (Agricultor / Admin)                          │
└─────────────────────────┬───────────────────────────────────────┘
                          │  HTTPS
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                  FRONTEND (Render Static)                        │
│              React 19 + Vite + TypeScript + TailwindCSS          │
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ Landing  │  │  Login   │  │ Register │  │   Dashboard   │  │
│  └──────────┘  └──────────┘  └──────────┘  │               │  │
│                                            │ ┌───────────┐ │  │
│                                            │ │  FarmMap  │ │  │
│                                            │ │ (Leaflet) │ │  │
│                                            │ └───────────┘ │  │
│                                            │ ┌───────────┐ │  │
│                                            │ │ AgroIntel │ │  │
│                                            │ │  ChatBot  │ │  │
│                                            │ └───────────┘ │  │
│                                            └───────────────┘  │
└─────────────────────────┬───────────────────────────────────────┘
                          │  REST API (JSON / JWT)
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                  BACKEND (Render Web Service)                    │
│              FastAPI 0.115 + Python + Uvicorn                   │
│                                                                 │
│  ┌─────────┐  ┌──────────┐  ┌──────────┐  ┌────────────────┐  │
│  │  Auth   │  │  ML/AI   │  │  Meteo   │  │  Farm & Zones  │  │
│  │  (JWT)  │  │(NaiveBay)│  │(OpenMet.)│  │  (CRUD SQLAlch)│  │
│  └─────────┘  └──────────┘  └──────────┘  └────────────────┘  │
│                          │                                      │
│                  ┌───────┴────────┐                            │
│                  │ DeepSeek (LLM) │                            │
│                  │  via OpenAI SDK│                            │
│                  └────────────────┘                            │
└─────────────────────────┬───────────────────────────────────────┘
                          │  SQL (PostgreSQL)
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BASE DE DADOS                                 │
│              PostgreSQL (Render — Managed DB)                   │
│                                                                 │
│  ┌──────────┐   ┌──────────┐   ┌────────────────┐             │
│  │ fazendas │   │ usuarios │   │  sensor_zones  │             │
│  └──────────┘   └──────────┘   └────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                  SERVIÇOS EXTERNOS                               │
│                                                                 │
│  ┌──────────────────────┐  ┌──────────────────────────────┐   │
│  │  Open-Meteo API      │  │     DeepSeek API             │   │
│  │  (Gratuito, sem key) │  │  (deepseek-chat via OpenAI   │   │
│  │  Previsão meteo 48h  │  │   SDK — requer DEEPSEEK_API  │   │
│  └──────────────────────┘  │   _KEY no ambiente)          │   │
│                             └──────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 3.1 Estrutura de Pastas do Projecto

```
Hydrolean---Angola/
│
├── backend/                        # Backend Python/FastAPI
│   ├── main.py                     # Aplicação principal — todos os endpoints
│   ├── auth.py                     # Módulo de autenticação JWT e registo
│   ├── database.py                 # Configuração SQLAlchemy + PostgreSQL
│   ├── models.py                   # Modelos ORM (Fazenda, Usuario, SensorZone)
│   ├── train_model.py              # Script de treino do modelo ML
│   ├── Crop_recommendation.csv     # Dataset agronómico (22 culturas)
│   ├── requirements.txt            # Dependências Python
│   ├── models/
│   │   └── NAIVEBAYES.pkl          # Modelo ML serializado (GaussianNB treinado)
│   ├── uploads/
│   │   └── logos/                  # Logos das fazendas carregados
│   ├── check_db.py                 # Utilitário: verifica estado da BD
│   ├── diagnose_db.py              # Utilitário: diagnóstico avançado da BD
│   ├── fix_db.py                   # Utilitário: correcções na BD
│   ├── fix_user.py                 # Utilitário: correcções de utilizadores
│   ├── migrate_to_postgres.py      # Script de migração SQLite → PostgreSQL
│   ├── test_chat.py                # Testes do endpoint /chat
│   ├── test_irrigation.py          # Testes do endpoint /irrigar
│   ├── test_login.py               # Testes de autenticação
│   └── test_polygon.py             # Testes do endpoint de polígono
│
├── frontend/                       # Frontend React/TypeScript
│   ├── src/
│   │   ├── App.tsx                 # Raiz da aplicação — gestão de rotas/views
│   │   ├── main.tsx                # Ponto de entrada React
│   │   ├── index.css               # Estilos globais
│   │   ├── lib/
│   │   │   └── api.ts              # Cliente HTTP centralizado (fetch + auth)
│   │   ├── types/                  # Definições TypeScript (Zone, etc.)
│   │   ├── utils/                  # Utilitários de suporte
│   │   ├── pages/
│   │   │   ├── Landing.tsx         # Página inicial (marketing)
│   │   │   ├── Login.tsx           # Página de login
│   │   │   ├── Register.tsx        # Página de registo multi-empresa
│   │   │   ├── Dashboard.tsx       # Dashboard principal
│   │   │   ├── WeatherPage.tsx     # Página de previsão meteorológica detalhada
│   │   │   └── ReportsPage.tsx     # Página de relatórios e exportação PDF
│   │   └── components/
│   │       └── dashboard/
│   │           ├── Sidebar.tsx             # Navegação lateral
│   │           ├── Topbar.tsx              # Barra superior
│   │           ├── FarmMap.tsx             # Mapa interactivo (Leaflet)
│   │           ├── SectorsGrid.tsx         # Grid de gestão de sensores
│   │           ├── ChatBot.tsx             # Chatbot flutuante AgroIntel
│   │           ├── AiAnalysisModal.tsx     # Modal de análise IA completa
│   │           ├── WeatherWidget.tsx       # Widget meteo no dashboard
│   │           └── SystemMetricsSidebar.tsx # Métricas do sistema (sidebar direita)
│   ├── package.json
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   └── tsconfig.json
│
├── render.yaml                     # Configuração de deploy (Render.com)
├── ideiageral.md                   # Documento de conceito e escopo do MVP
├── backend.md                      # (Placeholder — vazio)
└── .gitignore                      # Regras de exclusão do Git
```

---

## 4. Backend — FastAPI

### 4.1 Tecnologias e Dependências

| Pacote | Versão | Papel |
|---|---|---|
| `fastapi` | 0.115.0 | Framework web assíncrono |
| `uvicorn[standard]` | 0.30.0 | Servidor ASGI |
| `pydantic` | 2.9.0 | Validação e serialização de dados |
| `sqlalchemy` | 2.0.36 | ORM para PostgreSQL |
| `psycopg2-binary` | 2.9.10 | Driver PostgreSQL |
| `scikit-learn` | 1.5.2 | Motor de Machine Learning |
| `joblib` | 1.4.2 | Serialização do modelo ML |
| `numpy` | >=2.1.0 | Computação numérica |
| `pandas` | >=2.2.3 | Manipulação de dados |
| `httpx` | 0.27.0 | Cliente HTTP assíncrono (Open-Meteo) |
| `openai` | 1.54.0 | SDK para DeepSeek via interface OpenAI |
| `python-dotenv` | 1.0.1 | Gestão de variáveis de ambiente |
| `passlib[bcrypt]` | 1.7.4 | Hashing de passwords (importado mas não activo no MVP) |
| `PyJWT` | 2.10.1 | Geração e validação de tokens JWT |
| `python-multipart` | 0.0.12 | Upload de ficheiros (FormData) |

### 4.2 Configuração da Base de Dados

O módulo `database.py` configura a ligação à base de dados exclusivamente via variável de ambiente `DATABASE_URL`. O sistema:

1. Lê `DATABASE_URL` do ficheiro `.env`
2. Corrige automaticamente o prefixo `postgres://` → `postgresql://` (necessário para SQLAlchemy 2.x com bases Render)
3. Cria o engine SQLAlchemy e a sessão local
4. Expõe `get_db()` como dependência injectável nos endpoints FastAPI

> **Nota importante:** Em desenvolvimento local, era usado SQLite (`hydrolean.db`, `hydrolean2.db`), mas a versão actual exige **PostgreSQL** (tanto em produção como em desenvolvimento, apontando para a URL externa do Render).

### 4.3 Modelos de Dados (ORM)

Definidos em `models.py`:

#### `Fazenda` (tabela: `fazendas`)
Representa uma empresa/exploração agrícola — é o **tenant** do sistema multi-empresa.

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | Integer PK | Identificador único |
| `nome` | String | Nome da fazenda |
| `nif` | String (único) | NIF da empresa |
| `endereco` | String (nullable) | Endereço da fazenda |
| `logo_url` | String (nullable) | Caminho para o logo carregado |
| `polygon_coordinates` | String (nullable) | JSON serializado das coordenadas do polígono da área da fazenda |
| `created_at` | DateTime | Data de criação |

Relações: `1 → N` com `Usuario` e `SensorZone`

#### `Usuario` (tabela: `usuarios`)
Representa um utilizador autenticado, sempre associado a uma `Fazenda`.

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | Integer PK | Identificador único |
| `nome` | String | Nome completo |
| `email` | String (único) | Email de acesso |
| `senha_hash` | String | Password (plain text no MVP — ver nota de segurança) |
| `role` | String | Perfil (`admin` por omissão) |
| `fazenda_id` | FK → `fazendas` | Associação ao tenant |
| `created_at` | DateTime | Data de criação |

#### `SensorZone` (tabela: `sensor_zones`)
Representa um ponto de interesse no mapa da fazenda — pode ser um sensor de humidade, reservatório, bomba, painel solar, etc.

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | Integer PK | Identificador único |
| `name` | String | Nome atribuído pelo utilizador |
| `lat` | Float | Latitude GPS |
| `lng` | Float | Longitude GPS |
| `type` | String | Tipo: `sensor`, `tank`, `pump`, `solar`, `warehouse` |
| `status` | String | Estado: `optimal`, `attention`, `critical`, `irrigating` |
| `crop` | String | Cultura associada ao sensor |
| `moisture` | Integer | Humidade do solo (%) |
| `temp` | Integer | Temperatura (°C) |
| `rainForecast` | String | Previsão de chuva textual |
| `battery` | Integer | Nível de bateria (%) |
| `signal` | String | Tipo de sinal (4G, ND, etc.) |
| `lastUpdate` | String | Texto de última actualização |
| `aiMode` | Boolean | Modo IA activado |
| `pumpOn` | Boolean | Bomba ligada |
| `level` | Integer (nullable) | Nível de reservatório (apenas para `type=tank`) |
| `fazenda_id` | FK → `fazendas` | Isolamento multi-tenant |

### 4.4 Módulo de Autenticação (`auth.py`)

O módulo implementa autenticação baseada em **JWT (JSON Web Tokens)** com os seguintes componentes:

#### Configuração
- **Algoritmo:** HS256
- **Duração do token:** 7 dias (60 × 24 × 7 minutos)
- **Secret Key:** lida de `JWT_SECRET` no `.env` (fallback hardcoded para desenvolvimento)

#### Endpoints de Auth

| Método | Rota | Descrição |
|---|---|---|
| `POST` | `/register` | Registo de nova empresa + utilizador admin. Aceita `multipart/form-data` com logo opcional. Cria `Fazenda` e `Usuario` numa transacção atómica. Retorna JWT imediatamente. |
| `POST` | `/login` | Autenticação via `application/x-www-form-urlencoded` (compatível OAuth2). Retorna JWT. |
| `GET` | `/me` | Retorna o perfil completo do utilizador autenticado (nome, email, fazenda, logo, polígono). |

#### Fluxo Multi-Tenant

O payload do JWT contém `fazenda_id`, que é injectado automaticamente em cada request via a dependência `get_current_user()`. Desta forma, **cada utilizador só consegue aceder e modificar dados da sua própria fazenda**, garantindo isolamento completo entre tenants.

> **Nota de Segurança — MVP:** As passwords são actualmente armazenadas em plain text (sem bcrypt). O código de hashing está presente mas desactivado. Esta situação deve ser corrigida antes de qualquer lançamento em produção.

### 4.5 Endpoints da API REST

Todos os endpoints estão definidos em `main.py`.

#### Saúde do Sistema

| Método | Rota | Autenticação | Descrição |
|---|---|---|---|
| `GET` | `/health` | Pública | Retorna estado da API, versão, modelo ML e estado do DeepSeek |

#### Previsão Meteorológica

| Método | Rota | Autenticação | Descrição |
|---|---|---|---|
| `GET` | `/meteo?latitude=&longitude=` | JWT | Previsão meteorológica de 48h via Open-Meteo: temperatura, humidade, precipitação por janelas de 6h/12h/24h/48h, evapotranspiração, próxima chuva significativa (>=1mm), humidade do solo satelital, vento, previsão diária 3 dias e decisão rápida de irrigação |

#### Machine Learning — Recomendação de Culturas

| Método | Rota | Autenticação | Descrição |
|---|---|---|---|
| `POST` | `/predict` | JWT | Recebe N, P, K, temperatura, humidade, pH, precipitação. Retorna cultura recomendada (EN + PT), confiança (%) e top-5 probabilidades |

#### Irrigação Inteligente

| Método | Rota | Autenticação | Descrição |
|---|---|---|---|
| `POST` | `/irrigar` | JWT | Combina dados do solo + previsão meteo + DeepSeek para gerar plano de irrigação detalhado em português |
| `POST` | `/analise-completa` | JWT | Pipeline combinado: `/predict` + `/irrigar` numa única chamada |

#### Gestão da Fazenda e Sensores (CRUD)

| Método | Rota | Autenticação | Descrição |
|---|---|---|---|
| `PUT` | `/fazenda/polygon` | JWT | Actualiza as coordenadas do polígono da área da fazenda (cria a fazenda se não existir) |
| `GET` | `/fazenda/zones` | JWT | Lista todos os sensores/equipamentos da fazenda do utilizador |
| `POST` | `/fazenda/zones` | JWT | Cria um novo sensor/equipamento no mapa |
| `PUT` | `/fazenda/zones/{id}` | JWT | Actualiza um sensor/equipamento existente |
| `DELETE` | `/fazenda/zones/{id}` | JWT | Remove um sensor/equipamento |

#### Chatbot AgroIntel

| Método | Rota | Autenticação | Descrição |
|---|---|---|---|
| `POST` | `/chat` | JWT | Chatbot conversacional com contexto completo da fazenda. Carrega sensores, polígono e meteo em tempo real. Suporta histórico de conversa multi-turno. |

---

## 5. Motor de Inteligência Artificial

### 5.1 Modelo de Machine Learning — GaussianNB

O sistema utiliza um modelo de **Naive Bayes Gaussiano** (`GaussianNB` do scikit-learn) treinado para classificar 22 culturas tropicais com base em 7 parâmetros agronómicos.

#### Dataset de Treino
- **Ficheiro:** `Crop_recommendation.csv` (150 KB, ~2200 registos)
- **Features de entrada:**

| Feature | Unidade | Intervalo válido |
|---|---|---|
| N (Nitrogénio) | mg/kg | 0 – 200 |
| P (Fósforo) | mg/kg | 0 – 200 |
| K (Potássio) | mg/kg | 0 – 300 |
| temperature | °C | 0 – 60 |
| humidity | % | 0 – 100 |
| ph | escala pH | 0 – 14 |
| rainfall | mm | 0 – 500 |

- **Target:** `label` — nome da cultura em inglês
- **Split:** 80% treino / 20% teste (`random_state=42`)

#### Culturas Suportadas (22)

| Inglês | Português |
|---|---|
| rice | Arroz |
| maize | Milho |
| chickpea | Grão-de-bico |
| kidneybeans | Feijão-vermelho |
| pigeonpeas | Feijão-guandu |
| mothbeans | Feijão-moth |
| mungbean | Feijão-mungo |
| blackgram | Feijão-preto |
| lentil | Lentilha |
| pomegranate | Romã |
| banana | Banana |
| mango | Manga |
| grapes | Uva |
| watermelon | Melancia |
| muskmelon | Melão |
| apple | Maçã |
| orange | Laranja |
| papaya | Papaia |
| coconut | Coco |
| cotton | Algodão |
| jute | Juta |
| coffee | Café |

#### Ficheiro Serializado
O modelo treinado é guardado em `backend/models/NAIVEBAYES.pkl` via `joblib`. É carregado uma única vez no arranque do servidor e reutilizado em todas as previsões.

### 5.2 Integração Meteorológica — Open-Meteo

O backend integra a **API Open-Meteo** (gratuita, sem chave de API) para obter previsões em tempo real para qualquer coordenada GPS.

#### Dados recolhidos (horários, 48h)
- `temperature_2m` — Temperatura a 2m de altitude
- `relative_humidity_2m` — Humidade relativa do ar
- `precipitation_probability` — Probabilidade de precipitação
- `precipitation` — Volume de precipitação
- `rain` — Chuva
- `evapotranspiration` — Evapotranspiração (ET)
- `wind_speed_10m` — Velocidade do vento
- `soil_moisture_0_to_1cm` — Humidade do solo superficial (satelital)
- `soil_moisture_1_to_3cm` — Humidade do solo profunda (satelital)

#### Dados diários (3 dias)
- Temperatura máx/mín
- Precipitação total
- Probabilidade máxima de precipitação
- Chuva total
- ET0 FAO (evapotranspiração de referência)

#### Lógica de Decisão Rápida
O sistema implementa uma decisão automática de irrigação com quatro níveis:
1. **NÃO irrigar** — chuva significativa (>=1mm) prevista em <=6h
2. **NÃO irrigar** — previsão de >=10mm nas próximas 24h
3. **Reduzir volume** — previsão de >=5mm e <10mm nas próximas 24h
4. **IRRIGAR** — sem previsão de chuva significativa

### 5.3 IA Generativa — DeepSeek

O sistema utiliza o modelo **`deepseek-chat`** da DeepSeek, acessível através do **OpenAI SDK** (apontando para `https://api.deepseek.com`).

#### Dois casos de uso

**1. Plano de Irrigação (`/irrigar`)**
O sistema constrói um prompt detalhado com:
- Todos os parâmetros do solo
- Cultura recomendada pelo modelo ML
- Previsão meteorológica completa (próximas 6h, 12h, 24h, 48h)
- Próxima chuva significativa
- Humidade do solo satelital

O DeepSeek responde com um plano estruturado em texto simples (sem Markdown) cobrindo: decisão imediata, plano de irrigação (volume, frequência, horário), método recomendado, análise do solo, alertas e dicas práticas.

**2. Chatbot AgroIntel (`/chat`)**
O chatbot recebe contexto completo da fazenda do utilizador antes de cada resposta:
- Nome da fazenda e do utilizador
- Estado de todos os sensores (humidade, temperatura, cultura, bomba, bateria, cultura ML prevista)
- Dados meteorológicos em tempo real (se coordenadas fornecidas)

Suporta **histórico de conversa multi-turno** — o frontend envia o histórico completo e recebe o histórico actualizado com a nova resposta.

---

## 6. Frontend — React + Vite

### 6.1 Tecnologias

| Tecnologia | Versão | Papel |
|---|---|---|
| React | 19.2 | Framework UI |
| TypeScript | ~5.9.3 | Tipagem estática |
| Vite | 7.3.1 | Bundler e dev server |
| TailwindCSS | 3.4.19 | Estilização utilitária |
| Leaflet | 1.9.4 | Mapas interactivos |
| react-leaflet | 5.0.0 | Bindings React para Leaflet |
| leaflet-draw | 1.0.4 | Ferramentas de desenho no mapa |
| Recharts | 3.7.0 | Gráficos e visualizações |
| lucide-react | 0.575.0 | Biblioteca de ícones |
| jsPDF | 4.2.0 | Exportação de relatórios PDF |
| html2canvas | 1.4.1 | Captura de ecrã para PDF |

### 6.2 Gestão de Estado e Rotas

O projecto **não utiliza React Router**. Em vez disso, usa um sistema simples de estado `currentView` no componente raiz `App.tsx`, com 4 vistas possíveis:

```
'landing' → 'login'    → 'dashboard'
          → 'register' → 'dashboard'
```

Na inicialização, o `App` verifica se existe um token válido no `localStorage` e redirige directamente para o dashboard se autenticado.

### 6.3 Cliente HTTP (`api.ts`)

O ficheiro `src/lib/api.ts` centraliza todas as chamadas à API com:

- **`authInfo`** — Helper para gerir o token JWT no `localStorage` (chave: `hydrosync_token`)
- **`fetchWithAuth()`** — Wrapper que injeta automaticamente o header `Authorization: Bearer <token>` em todos os pedidos e faz logout automático em caso de resposta 401

#### Funções disponíveis no cliente `api`

| Função | Método | Rota |
|---|---|---|
| `checkHealth()` | GET | `/health` |
| `getMeteo(lat, lng)` | GET | `/meteo` |
| `predictCrop(params)` | POST | `/predict` |
| `fullAnalysis(params)` | POST | `/analise-completa` |
| `updateFarmPolygon(polygon)` | PUT | `/fazenda/polygon` |
| `getZones()` | GET | `/fazenda/zones` |
| `createZone(data)` | POST | `/fazenda/zones` |
| `updateZone(id, data)` | PUT | `/fazenda/zones/{id}` |
| `deleteZone(id)` | DELETE | `/fazenda/zones/{id}` |
| `getMe()` | GET | `/me` |
| `login(data)` | POST | `/login` |
| `register(formData)` | POST | `/register` |

### 6.4 Páginas

#### `Landing.tsx` (~30 KB)
Página de marketing pública. Apresenta o produto, funcionalidades, diferenciais e call-to-action para registo/login. É a primeira página que o utilizador não autenticado vê.

#### `Login.tsx` (~8 KB)
Formulário de autenticação. Envia `application/x-www-form-urlencoded` (compatível com o schema OAuth2 do FastAPI). Após login com sucesso, guarda o token e navega para o dashboard.

#### `Register.tsx` (~12.5 KB)
Formulário de registo multi-empresa. Permite criar uma nova fazenda e utilizador admin simultaneamente. Suporta upload de logo via `multipart/form-data`. Após registo com sucesso, recebe e guarda o JWT automaticamente.

#### `Dashboard.tsx` (~15 KB)
Página central da aplicação. Gere 5 abas de navegação:

| Aba | Conteúdo |
|---|---|
| `visao-geral` | WeatherWidget + FarmMap + SystemMetricsSidebar |
| `mapa-interativo` | FarmMap em modo expandido (altura total) |
| `setores` | SectorsGrid |
| `previsao` | WeatherPage |
| `relatorios` | ReportsPage |

**Gestão de dados multi-tenant:** Os dados do polígono e sensores são guardados no `localStorage` com chaves únicas por `fazenda_id` (ex: `hydrolean_polygon_5`). Isto garante que:
1. Os dados são carregados instantaneamente sem esperar pela API
2. Múltiplos tenants no mesmo browser não se misturam
3. O backend é a fonte de verdade — os dados locais são actualizados após cada resposta

#### `WeatherPage.tsx` (~18.5 KB)
Dashboard meteorológico detalhado. Exibe previsão completa de 48h com gráficos Recharts (temperatura, precipitação, humidade), dados de ET0, decisão de irrigação e previsão diária para 3 dias.

#### `ReportsPage.tsx` (~11.5 KB)
Página de geração e exportação de relatórios. Permite exportar o estado da fazenda (sensores, métricas, análises) para PDF usando jsPDF + html2canvas.

### 6.5 Componentes do Dashboard

#### `FarmMap.tsx` (~18.8 KB)
O componente mais complexo do frontend. Implementa um mapa interactivo baseado em **Leaflet** com:

- **Mapa base:** OpenStreetMap (gratuito)
- **Polígono da fazenda:** Área da plantação desenhada e editável
- **Marcadores de sensores:** Ícones diferenciados por tipo de equipamento (sensor, reservatório, bomba, painel solar, armazém)
- **Ferramentas de desenho** (modo edição):
  - Desenhar polígono da área
  - Adicionar marcadores (colocação de equipamentos)
  - Editar elementos existentes
  - Apagar elementos
- **Modal de detalhes do sensor:** Clique num marcador abre um popup com dados completos
- **Paleta de equipamentos:** Interface para colocar novos equipamentos no mapa
- **Centro dinâmico:** O mapa centra-se automaticamente no polígono da fazenda ou no primeiro sensor

#### `SectorsGrid.tsx` (~15.2 KB)
Grid de gestão de todos os sensores e equipamentos. Para cada zona exibe:
- Estado (cor/badge): Ótimo, Atenção, Crítico, Irrigando
- Humidade e temperatura
- Cultura associada
- Estado da bomba e modo IA
- Botões de acção: Ligar/desligar bomba, Activar/desactivar modo IA
- Modal de análise IA (`AiAnalysisModal`)

#### `ChatBot.tsx` (~20.8 KB)
Chatbot flutuante disponível em todo o dashboard. Características:
- Janela flutuante colapsável (botão no canto inferior direito)
- Histórico de conversa completo (multi-turno)
- Envio de localização GPS (geolocalização do browser) para contexto meteo no chat
- Indicador de escrita ("a digitar...")
- Formatação automática de respostas em parágrafos

#### `AiAnalysisModal.tsx` (~11 KB)
Modal de análise completa via IA. Permite ao utilizador inserir parâmetros de solo manualmente e obter uma análise combinada (previsão de cultura + plano de irrigação) via `/analise-completa`.

#### `Sidebar.tsx` (~10 KB)
Navegação lateral com:
- Logo e nome da fazenda
- Menu de navegação entre abas
- Nome e email do utilizador logado
- Botão de logout (remove token do localStorage)

#### `WeatherWidget.tsx` (~4.2 KB)
Widget compacto de meteorologia exibido na aba "Visão Geral". Mostra temperatura actual, humidade, precipitação e decisão rápida de irrigação.

#### `SystemMetricsSidebar.tsx` (~6.7 KB)
Barra lateral direita com métricas agregadas do sistema:
- Total de sensores activos
- Percentagem de sensores em estado óptimo
- Bombas ligadas
- Economia estimada de água

---

## 7. Deploy e Infraestrutura

### 7.1 Plataforma: Render.com

O projecto está configurado para deploy automático na plataforma **Render**, utilizando o ficheiro `render.yaml` na raíz do repositório.

### 7.2 Serviços Configurados

#### Backend — `hydrosync-backend`
```yaml
type: web
runtime: python
region: frankfurt
plan: free
rootDir: backend
buildCommand: pip install -r requirements.txt
startCommand: uvicorn main:app --host 0.0.0.0 --port $PORT
```

**Variáveis de ambiente obrigatórias:**

| Variável | Descrição | Geração |
|---|---|---|
| `DEEPSEEK_API_KEY` | Chave da API DeepSeek | Manual no dashboard do Render |
| `SECRET_KEY` | Chave secreta da aplicação | Gerada automaticamente pelo Render |
| `DATABASE_URL` | URL do PostgreSQL | Manual (External URL do DB do Render) |

#### Frontend — `hydrosync-frontend`
```yaml
type: web
runtime: static
region: frankfurt
plan: free
rootDir: frontend
buildCommand: npm install && npm run build
staticPublishPath: dist
routes:
  - type: rewrite
    source: /*
    destination: /index.html   # SPA fallback
```

**Variáveis de ambiente:**

| Variável | Descrição |
|---|---|
| `VITE_API_URL` | URL pública do backend (ex: `https://hydrosync-backend.onrender.com`) |

### 7.3 Base de Dados

- **Tipo:** PostgreSQL gerido pelo Render
- **Região:** Frankfurt (coerente com os serviços)
- **Plano:** Free (com limitações de storage e ligações)

> **Nota:** No desenvolvimento local, foram usados ficheiros SQLite (`hydrolean.db`, `hydrolean2.db`) que estão excluídos do Git via `.gitignore`. O script `migrate_to_postgres.py` foi criado para facilitar a migração dos dados para PostgreSQL.

---

## 8. Segurança e Considerações

### 8.1 Estado Actual de Segurança

| Aspecto | Estado | Observação |
|---|---|---|
| Autenticação JWT | Implementado | HS256, 7 dias de validade |
| Isolamento multi-tenant | Implementado | `fazenda_id` no token, filtro em todos os queries |
| CORS | Permissivo | `allow_origins=["*"]` — aceitável em MVP, rever em produção |
| Hashing de passwords | Desactivado | Plain text no MVP — CRÍTICO para produção |
| HTTPS | Via Render | Render força HTTPS em produção |
| Secrets no .env | Correcto | `.env` excluído do Git |
| Rate limiting | Não implementado | Risco de abuso da API DeepSeek |
| Input validation | Pydantic | Todos os campos com `ge`/`le` bounds |

### 8.2 Notas sobre o MVP

O projecto está explicitamente numa fase de **MVP (Produto Mínimo Viável)**, com algumas simplificações documentadas no código:

- Passwords em plain text (`# pwd_context foi removido para o MVP`)
- Dados de sensores são simulados/inseridos manualmente pelo utilizador (sem hardware físico real)
- Os valores de N, P, K, pH no chatbot usam valores proxy padrão quando não há sensores com dados completos
- O modelo de ML usa dados de treino globais, não calibrados especificamente para solos angolanos

---

## 9. Fluxo de Utilização Típico

```
1. REGISTO
   Utilizador acede ao Landing Page
   → Clica em "Criar Conta"
   → Preenche dados da empresa (nome, NIF, endereço, logo)
   → Preenche dados de acesso (nome, email, password)
   → Sistema cria Fazenda + Usuario na BD
   → JWT emitido automaticamente → redirige para Dashboard

2. CONFIGURAÇÃO DA FAZENDA
   No Dashboard > Mapa Interactivo
   → Clica em "Editar Área"
   → Desenha o polígono da área de plantação no mapa
   → Adiciona sensores/equipamentos (marcadores) na localização correta
   → Salva → persiste no backend + localStorage

3. ANÁLISE DE SOLO
   No Dashboard > Setores
   → Selecciona um sensor → Clica em "Análise IA"
   → Insere parâmetros do solo (N, P, K, pH, temperatura, humidade, precipitação)
   → Sistema: prevê cultura ideal (ML) + obtém meteo em tempo real + gera plano via DeepSeek
   → Exibe recomendação detalhada em português

4. MONITORIZAÇÃO CONTÍNUA
   No Dashboard > Visão Geral
   → Widget meteo mostra estado actual
   → Sensores mostram estado (humidade, temperatura, bomba)
   → Utilizador pode ligar/desligar bombas manualmente
   → Activar Modo IA por sensor (automatiza decisão)

5. CONSULTA AO CHATBOT
   Clica no ícone do chatbot (canto inferior direito)
   → Chat AgroIntel tem acesso ao contexto da fazenda em tempo real
   → Responde em português com dados dos sensores e previsão meteo

6. RELATÓRIOS
   Dashboard > Relatórios
   → Exporta relatório da fazenda em PDF
```

---

## 10. Escopo do MVP — Estado Actual vs. Desejado

### Funcionalidades Implementadas

- [x] Registo multi-empresa (multi-tenant)
- [x] Autenticação JWT
- [x] Dashboard com mapa interactivo Leaflet
- [x] Desenho e edição do polígono da fazenda
- [x] Colocação de sensores e equipamentos no mapa
- [x] CRUD completo de zonas/sensores
- [x] Previsão de cultura via ML (GaussianNB, 22 culturas)
- [x] Integração meteorológica em tempo real (Open-Meteo)
- [x] Geração de plano de irrigação via DeepSeek
- [x] Análise completa combinada (ML + Meteo + IA)
- [x] Chatbot AgroIntel com contexto real
- [x] Exportação de relatórios em PDF
- [x] Persistência local (localStorage) como fallback offline
- [x] Deploy configurado no Render (backend + frontend + PostgreSQL)
- [x] Landing page de marketing

### Em Desenvolvimento / Por Implementar

- [ ] Hashing de passwords (bcrypt)
- [ ] Rate limiting na API
- [ ] Hardware físico real (sensores Arduino/ESP32)
- [ ] Alertas push / notificações (sobre-irrigação, sensores críticos)
- [ ] Histórico de dados dos sensores (time series)
- [ ] Gráficos de tendência por sensor
- [ ] Gestão de utilizadores (multi-admin por fazenda)
- [ ] Integração com sistemas de rega automatizados
- [ ] App mobile
- [ ] Calibração do modelo ML para solos angolanos específicos

---

## 11. Como Executar Localmente

### Backend

```bash
# 1. Entrar na pasta
cd backend

# 2. Criar ambiente virtual
python -m venv .venv
.venv\Scripts\activate  # Windows

# 3. Instalar dependências
pip install -r requirements.txt

# 4. Configurar variáveis de ambiente
# Criar ficheiro .env com:
# DATABASE_URL=postgresql://...
# DEEPSEEK_API_KEY=sk-...
# JWT_SECRET=uma_chave_segura

# 5. Treinar o modelo ML (se NAIVEBAYES.pkl não existir)
python train_model.py

# 6. Iniciar servidor
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

- API disponível em: `http://localhost:8000`
- Documentação interactiva: `http://localhost:8000/docs`

### Frontend

```bash
# 1. Entrar na pasta
cd frontend

# 2. Instalar dependências
npm install

# 3. Configurar variável de ambiente
# Criar ficheiro .env com:
# VITE_API_URL=http://localhost:8000

# 4. Iniciar servidor de desenvolvimento
npm run dev
```

- Aplicação disponível em: `http://localhost:5173`

---

## 12. Glossário

| Termo | Definição |
|---|---|
| **HydroSync** | Nome oficial do produto |
| **Hydrolean** | Nome inicial do projecto (pasta e repositório) |
| **Tenant** | Uma empresa/fazenda individual no sistema multi-empresa |
| **Fazenda** | Entidade principal no sistema — representa uma exploração agrícola |
| **SensorZone** | Ponto de interesse no mapa (sensor, bomba, reservatório, etc.) |
| **GaussianNB** | Classificador Naive Bayes Gaussiano — modelo de ML utilizado |
| **Open-Meteo** | API meteorológica gratuita usada para previsões em tempo real |
| **DeepSeek** | Modelo de linguagem (LLM) usado para geração de recomendações em linguagem natural |
| **ET0** | Evapotranspiração de referência FAO — métrica agronómica de perda de água |
| **MVP** | Minimum Viable Product — versão mínima funcional do produto |
| **JWT** | JSON Web Token — mecanismo de autenticação stateless |
| **CORS** | Cross-Origin Resource Sharing — política de segurança para pedidos cross-domain |

---

*Documento gerado automaticamente com base na análise do código-fonte do projecto em Junho 2026.*
