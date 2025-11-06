# 👀 Projeto X9 - Sistema de Monitoramento de Atendimento

<div align="center">

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![n8n](https://img.shields.io/badge/n8n-1.109.2-red.svg)
![Status](https://img.shields.io/badge/status-active-success.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

**Sistema automatizado de monitoramento e gamificação de atendimento ao cliente para grupos do WhatsApp**

[Arquitetura](#-arquitetura) • [Instalação](#-instalação) • [Uso](#-uso) • [Roadmap V2](#-roadmap-v2)

</div>

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Características Principais](#-características-principais)
- [Arquitetura](#-arquitetura)
- [Workflows](#-workflows)
- [Tecnologias](#-tecnologias)
- [Instalação](#-instalação)
- [Uso](#-uso)
- [Estrutura de Dados](#-estrutura-de-dados)
- [Roadmap V2](#-roadmap-v2)
- [Problemas Conhecidos](#-problemas-conhecidos)
- [Contribuindo](#-contribuindo)
- [Licença](#-licença)

---

## 🎯 Sobre o Projeto

O **X9** é um sistema de monitoramento automatizado que acompanha o atendimento de gestores da Rugido aos grupos de clientes no WhatsApp. Ele analisa diariamente se os grupos foram atendidos, gera rankings de performance e alimenta relatórios semanais.

### 🎮 Gamificação

O sistema implementa mecânicas de gamificação para incentivar a equipe:
- 🏆 Rankings diários e semanais
- 🥇 Reconhecimento público dos gestores 100%
- 📊 Métricas de performance transparentes
- 🎯 Metas claras (3+ mensagens = atendimento)

### 👥 Stakeholders

- **Gestores**: 4 squads de atendimento
- **Time de Projetos**: Grupo principal que recebe os relatórios
- **Clientes**: 40+ grupos monitorados ativamente

---

## ✨ Características Principais

### 📊 Monitoramento Automático
- Verifica **2x por dia** (17h e 18h) se grupos foram atendidos
- Considera atendimento válido quando há **3+ mensagens** do gestor no dia
- Filtra automaticamente finais de semana (não executa aos sábados e domingos)

### 📈 Relatórios Inteligentes
- **17h**: Relatório parcial mostrando grupos pendentes por gestor
- **18h**: Ranking final com gestores que atingiram 100%
- **Sexta-feira 18h05**: Relatório semanal consolidado com medalhas 🥇🥈🥉

### 🔄 Auto-Cadastro
- Quando o bot é adicionado a um novo grupo, ele se cadastra automaticamente na planilha
- Grupos inativos podem ser marcados e não serão monitorados

### 💾 Histórico Completo
- Armazena dados diários em planilhas separadas (100% vs Incompleto)
- Permite análises retroativas e geração de relatórios customizados

---

## 🏗️ Arquitetura

```
┌─────────────────────────────────────────────────────────────────────┐
│                          PROJETO X9 - V1                             │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────────┐
│   CHATWOOT       │
│   (Webhook)      │──────┐
└──────────────────┘      │
                          │
                          ▼
                 ┌─────────────────┐
                 │  Workflow 1:    │
                 │  Auto-Cadastro  │
                 │  de Grupos      │
                 └────────┬────────┘
                          │
                          ▼
              ┌──────────────────────┐
              │  Google Sheets       │
              │  "Grupos Projetos"   │◄────────────┐
              └──────────┬───────────┘             │
                         │                         │
                         │                         │
              ┌──────────▼────────────┐            │
              │  Workflow 2:          │            │
              │  Resumidor Diário     │            │
              │  (17h e 18h)          │            │
              └──────────┬────────────┘            │
                         │                         │
                ┌────────┴────────┐                │
                ▼                 ▼                │
     ┌──────────────────┐  ┌─────────────────┐    │
     │  Chatwoot API    │  │  Google Sheets  │────┘
     │  (Buscar msgs)   │  │  "Atendimento"  │
     └──────────┬───────┘  └─────────────────┘
                │
                ▼
     ┌──────────────────────┐
     │  Grupo WhatsApp      │
     │  "Time de Projetos"  │
     └──────────────────────┘
                ▲
                │
     ┌──────────┴────────────┐
     │  Workflow 3:          │
     │  Relatório Semanal    │
     │  (Sexta 18h05)        │
     └───────────────────────┘
```

### 🔄 Fluxo de Dados

1. **Bot é adicionado ao grupo** → Webhook Chatwoot → Workflow 1 → Cadastro automático
2. **17h (dias úteis)** → Workflow 2 busca mensagens do dia → Gera relatório parcial
3. **18h (dias úteis)** → Workflow 2 gera ranking 100% → Envia parabenizações
4. **Sexta 18h05** → Workflow 3 consolida semana → Envia ranking com medalhas

---

## 🔧 Workflows

### 1️⃣ Auto-Cadastro de Grupos
**Arquivo**: `[Projetos] X9 - Adiciona grupo na planilha quando número é adicionado.json`

**Trigger**: Webhook do Chatwoot (evento `message_created` com conteúdo "joined group")

**Funcionalidade**: Quando o número cadastrado do X9 é adicionado a um novo grupo:
- Extrai ID e nome do grupo
- Insere na planilha "Grupos Projetos Rugido"
- Marca como ativo automaticamente

**Planilha de Saída**: `Grupos Projetos Rugido`

---

### 2️⃣ Resumidor Diário (Principal)
**Arquivo**: `[Projetos] X9 - Resumidor de grupos CHATWOOT.json`

**Triggers**: 
- Schedule 17h (dias úteis)
- Schedule 18h (dias úteis)

**Funcionalidade Detalhada**:

#### 📥 Etapa 1: Coleta de Dados
1. Busca todos os grupos ativos na planilha
2. Para cada grupo, consulta Chatwoot API:
   - Busca contato pelo ID do grupo
   - Puxa conversations
   - Extrai mensagens do dia

#### 🔍 Etapa 2: Análise
3. Filtra mensagens do dia atual (00:00 até 23:59)
4. Conta total de mensagens por grupo
5. Classifica: **>2 mensagens = Atendido** | **≤2 mensagens = Não atendido**

#### 📊 Etapa 3: Agrupamento
6. Agrupa grupos por gestor responsável
7. Calcula percentual de atendimento de cada gestor
8. Remove emojis dos nomes para melhor visualização

#### 📤 Etapa 4: Envio e Armazenamento

**Às 17h**:
```
📊 RELATÓRIO DO X9👀:
📊 RELATÓRIO DE ATENDIMENTO - 06/11/2025

👤 Lucas Santana e Abigayl: 18/22
   • Cliente A: ✅ Atendido
   • Cliente B: ❌ Sem atendimento
   ...
```

**Às 18h** (se houver gestores 100%):
```
🎉 PARABÉNS GESTORES 100% - 06/11/2025

🏆 Eduardo
   ✅ 10/10 grupos atendidos (100%)

🏆 Thainá
   ✅ 12/12 grupos atendidos (100%)

👏 Excelente trabalho! Continuem assim!
```

**Caso todos sejam atendidos**:
```
*BORAAAAAAAAA* *É O DREAM TEAM PÔ* 🤑🤑🤑🤑🤑
*PARABÉNS GALERA 100% HOJE GERAL*
```

#### 💾 Armazenamento
- **Gestores 100%** → Aba "Atendidos 100%"
- **Gestores <100%** → Aba "Não atenderam 100%"

**Planilhas Usadas**:
- 📖 Leitura: `Grupos Projetos Rugido`
- 💾 Escrita: `Gestores Atendimento - RUGIDO` (2 abas)

---

### 3️⃣ Relatório Semanal
**Arquivo**: `[Projetos] X9 - Gerar resumo relatório da semana.json`

**Trigger**: Schedule Sexta-feira 18h05

**Funcionalidade**:
1. Define período: Segunda 00:00 até Sexta 23:59
2. Lê dados das duas abas da planilha de atendimento
3. Consolida atendimentos por gestor na semana
4. Calcula percentual: `(feitos / devidos) * 100`
5. Ordena por performance
6. Atribui medalhas:
   - 🥇 Todos com 100% (em ordem alfabética)
   - 🥈 Segundo lugar
   - 🥉 Terceiro lugar
   - 🔹 Demais posições

**Exemplo de Saída**:
```
🏆 Ranking Semanal de Gestores 🏆
(Período: 04/11/2025 a 08/11/2025)

🥇 Eduardo - 100.00% de conclusão (50/50 atendimentos)

🥇 Thainá - 100.00% de conclusão (60/60 atendimentos)

🥈 Lucas Santana e Abigayl - 95.45% de conclusão (105/110 atendimentos)

🥉 Marcos - 91.67% de conclusão (44/48 atendimentos)
```

**Planilha de Entrada**: `Gestores Atendimento - RUGIDO` (2 abas)

---

## 💻 Tecnologias

### Core
- **n8n** `v1.109.2` - Self-hosted workflow automation
- **Chatwoot** - CRM e gerenciamento de conversas
- **Google Sheets API** - Armazenamento e histórico

### Integrações
- **Chatwoot API** `v1` - Busca de mensagens e contatos
- **Google Sheets OAuth2** - Leitura/escrita de dados
- **WhatsApp** (via Chatwoot) - Envio de notificações

### Linguagens
- **JavaScript** (Node.js) - Lógica de negócio nos nós Code
- **n8n Expressions** - Manipulação de dados

---

## 🚀 Instalação

### Pré-requisitos

```bash
# n8n instalado e rodando
n8n --version  # Deve ser >= 1.109.2

# Chatwoot configurado
# Google Cloud Project com Sheets API habilitada
```

### 1. Clonar Workflows

```bash
# Importar os 3 arquivos .json no n8n:
1. [Projetos] X9 - Adiciona grupo na planilha quando número é adicionado.json
2. [Projetos] X9 - Resumidor de grupos CHATWOOT.json
3. [Projetos] X9 - Gerar resumo relatório da semana.json
```

### 2. Configurar Credenciais

#### Google Sheets
1. No n8n, acesse **Credentials**
2. Adicione **Google Sheets OAuth2 API**
3. Autorize a conta (Eduardo no caso da Rugido)

#### Chatwoot
1. Acesse Chatwoot → **Settings** → **Applications** → **API Access Tokens**
2. Crie um token com permissões de leitura de mensagens
3. Anote o token para usar nos workflows

### 3. Configurar Planilhas

#### Planilha 1: Grupos Projetos Rugido
**ID**: `1r2jE8qP67k4Ws74aC1ftWbsz84gIA73oqlntx04qSqs`

**Estrutura** (Aba "Página1"):
```
| Nome do grupo | id do grupo | Está ativo? | Gestor Responsável |
|---------------|-------------|-------------|--------------------|
```

#### Planilha 2: Gestores Atendimento - RUGIDO
**ID**: `1pdjbDpebIKFhtEQnRCEdLMcwQ614S_0-9IVJ56XDqk4`

**Aba "Atendidos 100%"**:
```
| Dia do atendimeto | Nome do Gestor | Quantidade de atendimentos |
|-------------------|----------------|----------------------------|
```

**Aba "Não atenderam 100%"**:
```
| Dia do atendimeto | Nome do Gestor | Quantidade de atendimentos | Precisava ter feito |
|-------------------|----------------|----------------------------|---------------------|
```

### 4. Configurar Webhook do Chatwoot

```bash
# Webhook URL do n8n (Workflow 1)
https://seu-n8n.com/webhook/170e3d80-104d-4db0-8753-1e9914e75b87

# Configurar no Chatwoot:
# Settings → Webhooks → Add Webhook
# Event: message_created
```

### 5. Atualizar Variáveis

Nos 3 workflows, ajuste:
- **ID do grupo WhatsApp** de destino (Time de Projetos)
- **Token Chatwoot API**
- **URLs das planilhas** (se diferentes)

### 6. Ativar Workflows

No n8n, ative os 3 workflows:
- ✅ Auto-Cadastro de Grupos
- ✅ Resumidor Diário
- ✅ Relatório Semanal

---

## 📖 Uso

### Adicionar Novo Grupo ao Monitoramento

1. Adicione o número do bot Chatwoot ao grupo do cliente
2. O sistema detecta automaticamente e cadastra
3. Verifique na planilha se apareceu com status "SIM"
4. Atribua um gestor responsável manualmente na planilha

### Desativar Monitoramento de um Grupo

1. Acesse a planilha "Grupos Projetos Rugido"
2. Mude "Está ativo?" de "SIM" para "NÃO"
3. O grupo será ignorado nos próximos relatórios

### Interpretar Relatórios

#### Relatório das 17h
- Mostra grupos **pendentes** de atendimento
- Permite que gestores se mobilizem até as 18h
- Agrupa por gestor para facilitar visualização

#### Ranking das 18h
- **Aparece apenas se houver gestores 100%**
- Lista em ordem alfabética (todos têm o mesmo peso)
- Serve como reconhecimento público

#### Relatório Semanal
- Considera de **Segunda a Sexta**
- Medalhas por ordem de performance
- Múltiplos 100% recebem medalha de ouro

---

## 📊 Estrutura de Dados

### Webhook Payload (Auto-Cadastro)

```json
{
  "body": {
    "messages": [{
      "content": "joined group",
      "sender": {
        "identifier": "120363422844879974@g.us",
        "name": "Teste 2 (GRUPO)"
      }
    }]
  }
}
```

### Chatwoot API Response (Mensagens)

```json
{
  "payload": [
    {
      "id": 1502837,
      "content": "Olá, como posso ajudar?",
      "created_at": 1762432407,
      "message_type": 0,
      "sender_type": "Contact"
    }
  ]
}
```

### Dados Processados (Workflow 2)

```json
{
  "id_grupo": "120363422844879974@g.us",
  "nome_grupo": "Cliente XYZ",
  "nome_do_gestor": "Eduardo",
  "total_mensagens_hoje": 5,
  "tem_mensagens_hoje": true,
  "data_filtro": "2025-11-06"
}
```

---

## 🚧 Roadmap V2

### 🎯 Objetivo Principal
**Substituir a lógica simples de "3 mensagens" por análise qualitativa real com LLMs**

### 🆕 Novas Funcionalidades

#### 1. Análise de Qualidade com IA
- ✅ Implementar LLMs para avaliar **qualidade** das mensagens
- ✅ Detectar tentativas de "burlar" com mensagens vazias
  - ❌ "Oi", "Bom dia", "Tudo bem?" (sem contexto)
  - ✅ Resolver dúvidas, avançar atendimento, feedback real
- ✅ Critérios baseados no treinamento Fathom da Rugido
- ✅ Transcrição de áudios para análise completa

#### 2. Alerta de Clientes Abandonados
- ✅ Detectar clientes **2 dias consecutivos sem atendimento**
- ✅ Enviar alerta automático para gestor responsável
- ✅ Escalar para liderança se não resolvido

#### 3. Feedback Automático Contextual
- ✅ Análise individual de cada atendimento
- ✅ Identificar pontos de melhoria específicos
- ✅ Enviar feedback no **grupo do Squad** (não polui grupo principal)
- ✅ Incluir trechos de exemplo para embasar

#### 4. Banco de Dados Dedicado
- ✅ Migrar de Google Sheets para banco relacional (PostgreSQL?)
- ✅ Histórico completo de conversas
- ✅ Análises temporais e tendências

### 🔧 Melhorias Técnicas

- ✅ Arquitetura multi-agente (RAG + LLMs especializados)
- ✅ Pipeline de transcrição de áudio
- ✅ Cache inteligente para reduzir chamadas à API
- ✅ Dashboard web para visualização (Grafana?)

### 📝 Critérios de Sucesso V2

1. **Nenhum cliente fica 2 dias sem atendimento** sem alerta
2. **Zero falsos positivos** (mensagens reais classificadas como inválidas)
3. **Feedbacks acionáveis** recebidos pelos gestores
4. **Redução de 50%** em tentativas de burlar o sistema

---

## ⚠️ Problemas Conhecidos

### 1. Gestores Burlando o Sistema
**Problema**: Envio de 3 mensagens genéricas ("Oi", "Bom dia", "Tudo bem?") sem atendimento real

**Impacto**: Grupo conta como atendido sem valor agregado ao cliente

**Solução V2**: Análise com LLM para detectar tentativas de burla

---

### 2. Limitação de Mensagens na API
**Problema**: Chatwoot API retorna apenas últimas 50 mensagens por padrão

**Impacto**: Grupos muito ativos podem ter dados incompletos

**Workaround Atual**: Variável `mensagens_maxima` configurada para 50

**Solução Futura**: Paginação ou webhook em tempo real

---

### 3. Delay nas Planilhas
**Problema**: Google Sheets pode ter latência em leitura/escrita simultânea

**Impacto**: Raramente causa erro de execução no n8n

**Workaround**: Nó "Wait" de 30s entre leituras

**Solução V2**: Banco de dados dedicado

---

### 4. Sem Monitoramento Proativo
**Problema**: Se o workflow falhar, não há alerta automático

**Impacto**: Relatórios podem não ser enviados sem que ninguém perceba

**Solução Atual**: Verificação manual no n8n

**Solução Futura**: Healthcheck com Dead Man's Snitch ou similar

---

## 🤝 Contribuindo

### Fluxo de Trabalho

1. Clone o repositório
2. Importe workflows no n8n de desenvolvimento
3. Teste mudanças com grupos de teste
4. Documente alterações
5. Exporte workflows atualizados
6. Commit e PR

### Guidelines

- ✅ **Sempre** testar em ambiente de staging antes de produção
- ✅ **Sempre** atualizar a documentação junto com o código
- ✅ **Sempre** adicionar logs em nós Code para debug
- ✅ Usar nomes descritivos em nós e variáveis
- ✅ Comentar lógicas complexas no JavaScript

---

## 📞 Contato

**Time de Desenvolvimento**
* **LinkedIn:** `https://www.linkedin.com/in/eduardo-sousa-dev12`
* **E-mail:** `eduardodesousasilva12@gmail.com`

**Documentação Completa**
- [ARCHITECTURE.md](./ARCHITECTURE.md) - Detalhes técnicos da arquitetura
- [WORKFLOWS.md](./WORKFLOWS.md) - Documentação node-by-node
- [DEPLOYMENT.md](./DEPLOYMENT.md) - Guia completo de instalação
- [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) - Resolução de problemas

---

<div align="center">
</div>
