# 📖 API Reference - NexCash

## Base URL

```
http://localhost:3001/api
```

## Authentication

Todas as requisições protegidas requerem header:

```
Authorization: Bearer {token}
```

## 🔐 Auth Endpoints

### POST `/auth/register`

Registrar novo usuário

**Request:**
```json
{
  "whatsappNumber": "5511999999999",
  "name": "João Silva",
  "email": "joao@example.com"
}
```

**Response:**
```json
{
  "success": true,
  "message": "User registered successfully",
  "token": "eyJhbGc...",
  "data": {
    "id": "uuid-string",
    "name": "João Silva",
    "whatsappNumber": "5511999999999",
    "email": "joao@example.com"
  }
}
```

### POST `/auth/login`

Fazer login

**Request:**
```json
{
  "whatsappNumber": "5511999999999"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Login successful",
  "token": "eyJhbGc...",
  "data": { ... }
}
```

### POST `/auth/verify-token`

Verificar validade do token

**Headers:**
```
Authorization: Bearer {token}
```

**Response:**
```json
{
  "success": true,
  "message": "Token is valid",
  "data": { ... }
}
```

## 👤 User Endpoints

### GET `/users/{userId}`

Obter perfil do usuário

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid-string",
    "name": "João Silva",
    "email": "joao@example.com",
    "whatsapp_number": "5511999999999",
    "preferences": {},
    "created_at": "2026-05-15T10:00:00Z"
  }
}
```

### PUT `/users/{userId}`

Atualizar perfil do usuário

**Request:**
```json
{
  "name": "João Silva Santos",
  "email": "joao.silva@example.com",
  "preferences": {
    "currency": "BRL",
    "timezone": "America/Sao_Paulo"
  }
}
```

**Response:**
```json
{
  "success": true,
  "message": "User updated successfully",
  "data": { ... }
}
```

### DELETE `/users/{userId}`

Deletar usuário

**Response:**
```json
{
  "success": true,
  "message": "User deleted successfully"
}
```

## 💳 Transaction Endpoints

### POST `/transactions`

Criar nova transação

**Request:**
```json
{
  "userId": "uuid-string",
  "type": "expense",
  "amount": 250.50,
  "category": "Alimentação",
  "description": "Compras no mercado"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Transaction created successfully",
  "data": {
    "id": "uuid-string",
    "user_id": "uuid-string",
    "type": "expense",
    "amount": 250.50,
    "category": "Alimentação",
    "description": "Compras no mercado",
    "transaction_date": "2026-05-15T10:00:00Z",
    "created_at": "2026-05-15T10:00:00Z"
  }
}
```

### GET `/transactions/{transactionId}`

Obter transação específica

**Response:**
```json
{
  "success": true,
  "data": { ... }
}
```

### GET `/transactions/user/{userId}`

Listar transações do usuário

**Query Parameters:**
```
?startDate=2026-05-01
&endDate=2026-05-31
&category=Alimentação
&type=expense
```

**Response:**
```json
{
  "success": true,
  "count": 15,
  "data": [ ... ]
}
```

### GET `/transactions/user/{userId}/summary`

Resumo de transações por categoria

**Query Parameters:**
```
?startDate=2026-05-01&endDate=2026-05-31
```

**Response:**
```json
{
  "success": true,
  "period": {
    "start": "2026-05-01T00:00:00Z",
    "end": "2026-05-31T23:59:59Z"
  },
  "data": [
    {
      "category": "Alimentação",
      "type": "expense",
      "count": 10,
      "total": "1200.50"
    }
  ]
}
```

### PUT `/transactions/{transactionId}`

Atualizar transação

**Request:**
```json
{
  "amount": 300.00,
  "category": "Alimentação",
  "description": "Mercado e restaurante"
}
```

### DELETE `/transactions/{transactionId}`

Deletar transação

## 📅 Appointment Endpoints

### POST `/appointments`

Criar compromisso

**Request:**
```json
{
  "userId": "uuid-string",
  "title": "Reunião com cliente",
  "description": "Discussão sobre novo projeto",
  "date": "2026-05-20",
  "time": "15:00",
  "reminder": true,
  "reminderMinutes": 30
}
```

### GET `/appointments/user/{userId}`

Listar compromissos

### GET `/appointments/user/{userId}/upcoming`

Compromissos próximos

### PUT `/appointments/{appointmentId}`

Atualizar compromisso

### DELETE `/appointments/{appointmentId}`

Deletar compromisso

## 🎯 Goal Endpoints

### POST `/goals`

Criar meta

**Request:**
```json
{
  "userId": "uuid-string",
  "name": "Economizar para férias",
  "targetAmount": 5000,
  "deadline": "2026-12-31",
  "category": "Férias"
}
```

### GET `/goals/user/{userId}`

Listar metas do usuário

**Response:**
```json
{
  "success": true,
  "count": 3,
  "data": [
    {
      "id": "uuid-string",
      "name": "Economizar para férias",
      "target_amount": 5000,
      "current_amount": 2000,
      "percentage": 40,
      "remaining": 3000,
      "deadline": "2026-12-31T00:00:00Z"
    }
  ]
}
```

### POST `/goals/{goalId}/add-amount`

Adicionar valor à meta

**Request:**
```json
{
  "amount": 500
}
```

### PUT `/goals/{goalId}`

Atualizar meta

### DELETE `/goals/{goalId}`

Deletar meta

## 📊 Report Endpoints

### GET `/reports/monthly/{userId}`

Relatório mensal

**Response:**
```json
{
  "success": true,
  "data": {
    "period": "Mensal",
    "startDate": "2026-05-01T00:00:00Z",
    "endDate": "2026-05-31T23:59:59Z",
    "summary": {
      "totalIncome": "10000.00",
      "totalExpense": "4200.00",
      "balance": "5800.00",
      "transactionCount": 45
    },
    "categoryBreakdown": {
      "Alimentação": {
        "amount": 850,
        "count": 10
      }
    }
  }
}
```

### GET `/reports/weekly/{userId}`

Relatório semanal

### GET `/reports/daily/{userId}`

Relatório diário

### GET `/reports/cash-flow/{userId}`

Fluxo de caixa

### GET `/reports/comparison/{userId}`

Comparação entre períodos

## 📈 Analytics Endpoints

### GET `/analytics/insights/{userId}`

Insights financeiros

**Response:**
```json
{
  "success": true,
  "data": {
    "insights": [
      {
        "title": "Taxa de Economia",
        "value": "22.5%",
        "description": "Você economiza 22.5% da sua renda"
      }
    ],
    "warnings": [
      {
        "level": "medium",
        "message": "Suas despesas estão próximas da sua renda!"
      }
    ],
    "achievements": [
      {
        "title": "💪 Grande Economista",
        "description": "Você está economizando mais de 20%!"
      }
    ]
  }
}
```

### GET `/analytics/trends/{userId}`

Tendências de gasto

**Query Parameters:**
```
?months=6
```

### GET `/analytics/alerts/{userId}`

Alertas financeiros

### GET `/analytics/recommendations/{userId}`

Recomendações financeiras

## 📱 WhatsApp Endpoints

### POST `/whatsapp/webhook`

Receber mensagens do WhatsApp (interno)

### POST `/whatsapp/send-test`

Enviar mensagem de teste

**Request:**
```json
{
  "to": "5511999999999",
  "message": "Olá! Sou o NexCash 🚀"
}
```

### POST `/whatsapp/send-report`

Enviar relatório

**Request:**
```json
{
  "userId": "uuid-string",
  "phoneNumber": "5511999999999",
  "reportType": "monthly"
}
```

### POST `/whatsapp/send-reminder`

Enviar lembrete

**Request:**
```json
{
  "appointmentId": "uuid-string",
  "phoneNumber": "5511999999999"
}
```

## Status Codes

| Código | Significado |
|--------|-------------|
| 200 | OK - Sucesso |
| 201 | Created - Recurso criado |
| 400 | Bad Request - Dados inválidos |
| 401 | Unauthorized - Token inválido |
| 403 | Forbidden - Sem permissão |
| 404 | Not Found - Recurso não encontrado |
| 409 | Conflict - Conflito de dados |
| 500 | Server Error - Erro interno |

## Rate Limiting

- **Limite:** 100 requisições por 15 minutos
- **Header:** `X-RateLimit-Remaining`

---

**NexCash API** - Documentação Completa 🚀
