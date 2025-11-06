# 🚀 Guia de Setup - Projeto X9

## ⚡ Quick Start (5 minutos)

### 1. Pré-requisitos
- ✅ n8n v1.109.2+ instalado e rodando
- ✅ Chatwoot configurado
- ✅ Google Cloud Project com Sheets API habilitada
- ✅ Credenciais Google Sheets OAuth2

### 2. Importar Workflows no n8n

1. Acesse seu n8n
2. Clique em **"Import from File"**
3. Importe na ordem:
   - `workflows/01-auto-cadastro-grupos.json`
   - `workflows/02-resumidor-diario.json`
   - `workflows/03-relatorio-semanal.json`

### 3. Configurar Credenciais

#### Google Sheets OAuth2
```
n8n → Credentials → Add Credential → Google Sheets OAuth2 API
```
- Autorize a conta com acesso às planilhas
- Nome sugerido: `Google Sheets account (Seu Nome)`

#### Chatwoot API Token
```
Chatwoot → Settings → Applications → API Access Tokens
```
- Crie um token com permissão de leitura
- Anote o token para o próximo passo

### 4. Atualizar Variáveis nos Workflows

Em **cada um dos 3 workflows**, localize o nó **"Map Infos Iniciais"** ou similar e atualize:

```javascript
{
  "chatwoot_api_token": "SEU_TOKEN_AQUI",
  "chatwoot_account_id": "2",  // ou seu account_id
  "grupo_whatsapp_destino": "SEU_GRUPO_ID@g.us",
  "sheet_id_grupos": "SEU_ID_PLANILHA_1",
  "sheet_id_gestores": "SEU_ID_PLANILHA_2"
}
```

### 5. Criar Planilhas no Google Sheets

#### Planilha 1: "Grupos Projetos Rugido"
Criar com as colunas:
| Nome do grupo | id do grupo | Está ativo? | Gestor Responsável |
|---------------|-------------|-------------|--------------------|

#### Planilha 2: "Gestores Atendimento - RUGIDO"

**Aba "Atendidos 100%":**
| Dia do atendimeto | Nome do Gestor | Quantidade de atendimentos |
|-------------------|----------------|----------------------------|

**Aba "Não atenderam 100%":**
| Dia do atendimeto | Nome do Gestor | Quantidade de atendimentos | Precisava ter feito |
|-------------------|----------------|----------------------------|---------------------|

### 6. Configurar Webhook do Chatwoot

```
Chatwoot → Settings → Webhooks → Add Webhook
```

- **Event**: `message_created`
- **Webhook URL**: 
  ```
  https://seu-n8n.com/webhook/SEU-UUID-AQUI
  ```
  *(copie do nó Webhook do workflow 01-auto-cadastro)*

### 7. Ativar Workflows

No n8n, ative os 3 workflows:
- ✅ 01 - Auto-Cadastro de Grupos
- ✅ 02 - Resumidor Diário
- ✅ 03 - Relatório Semanal

### 8. Testar

1. Adicione o bot do Chatwoot em um grupo de teste
2. Verifique se apareceu na planilha "Grupos Projetos Rugido"
3. Atribua um gestor responsável
4. Aguarde os horários programados ou execute manualmente

---

## 🔧 Configuração Avançada

### Alterar Horários de Execução

No workflow **02-resumidor-diario.json**:
```
Nó "1x Dia as 17:0" → triggerAtHour: 17
Nó "1x dia as 18:00" → triggerAtHour: 18
```

No workflow **03-relatorio-semanal.json**:
```
Nó "Schedule Trigger" → 
  triggerAtDay: [5]  # Sexta-feira
  triggerAtHour: 18
  triggerAtMinute: 5
```

### Alterar Critério de Atendimento

No workflow **02-resumidor-diario.json**, localize o nó:
```javascript
"Separa grupos que não tiveram mensagem hoje"
```

E altere a linha:
```javascript
tem_atendimento: totalMensagensHoje > 2  // Mudar o "2" para o valor desejado
```

---

## 🐛 Troubleshooting

### Workflow não executa
- ✅ Verifique se está **ativo** (toggle verde)
- ✅ Verifique logs de execução no n8n
- ✅ Confirme credenciais Google Sheets

### Relatório não chega no grupo
- ✅ Verifique ID do grupo WhatsApp
- ✅ Confirme que bot está no grupo
- ✅ Teste conexão Chatwoot API

### Grupos não aparecem na planilha
- ✅ Webhook configurado corretamente?
- ✅ Bot foi realmente adicionado ao grupo?
- ✅ Verifique execuções do workflow 01

---

## 📚 Documentação Completa

Para mais detalhes, consulte:
- [README.md](./README.md) - Visão geral do projeto
- [ARCHITECTURE.md](./docs/ARCHITECTURE.md) - Arquitetura técnica (futuro)
- [TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) - Problemas comuns (futuro)

---

## 💬 Suporte

Dúvidas? Entre em contato:
- **Email**: eduardo@gruporugido.com
- **GitHub Issues**: [Abrir issue](https://github.com/eduardosousa12-dev/projeto-x9/issues)
