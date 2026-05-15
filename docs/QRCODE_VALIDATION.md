# 🔐 Sistema de Validação via WhatsApp com QR Code - NexCash

## 📱 Fluxo Completo

```
┌─────────────────────────────────────────────────────────┐
│ 1. USUÁRIO ACESSA PÁGINA DE LOGIN                       │
│    └─ Vê QR Code + Código de Validação                 │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 2. ESCANEAMENTO DO QR CODE                              │
│    └─ Abre WhatsApp com mensagem pré-pronta            │
│    └─ Exibe logo e nome "NexCash"                      │
│    └─ Código de validação: A1B2C3D4                    │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 3. USUÁRIO CLICA "ENVIAR" NO WHATSAPP                  │
│    └─ Mensagem: "Validar: A1B2C3D4"                    │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 4. WEBHOOK RECEBE MENSAGEM                              │
│    POST /api/whatsapp/webhook                           │
│    └─ Extrai número do usuário                         │
│    └─ Extrai código de validação                       │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 5. SISTEMA VALIDA CÓDIGO                                │
│    └─ Verifica se código existe                        │
│    └─ Verifica se não expirou (15 min)                 │
│    └─ Valida número WhatsApp                           │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 6. CRIAR USUÁRIO E TOKEN JWT                            │
│    └─ Insere na tabela users                           │
│    └─ Gera token JWT válido por 7 dias                 │
│    └─ Salva session_id no Redis                        │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 7. ENVIAR RESPOSTA VIA WHATSAPP                         │
│    └─ Mensagem de sucesso                              │
│    └─ Instruções iniciais                              │
│    └─ Botões para próximas ações                       │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 8. FRONTEND DETECTA VALIDAÇÃO                           │
│    └─ Polling a cada 2 segundos                        │
│    └─ GET /api/qrcode/status/:id                       │
│    └─ Recebe resposta com token                        │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 9. ✅ USUÁRIO LOGADO AUTOMATICAMENTE                    │
│    └─ Token armazenado em localStorage                 │
│    └─ Redireciona para dashboard                       │
│    └─ Mensagem de boas-vindas                          │
└─────────────────────────────────────────────────────────┘
```

---

## 💬 Mensagem no WhatsApp

### Contato NexCash Aparece Assim:

```
NexCash
+55 11 98765-4321
Assistente Financeiro 💰

🟢 Online
```

### Mensagem Recebida:

```
🚀 Olá! Sou o NexCash

Seu código de validação é:

A1B2C3D4

Use este código para finalizar seu registro!

---

Dúvidas? Digite:
• help - Ajuda
• saldo - Ver saldo
• gastos - Registrar despesa
```

### Resposta Após Validação:

```
✅ VALIDAÇÃO REALIZADA COM SUCESSO!

Bem-vindo ao NexCash! 🎉

Sua conta foi criada:
📱 Número: +55 11 98765-4321
🕐 Data: 15/05/2026

Agora você pode:
💰 Registrar gastos
💵 Registrar receitas
📊 Ver relatórios
📅 Agendar compromissos
🎯 Criar metas

Envie uma mensagem para começar!
```

---

## 🎯 Endpoints

### 1. Gerar QR Code

```http
GET /api/qrcode/generate
Authorization: Bearer {token_opcional}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "qr_code_id": "abc123def456",
    "qr_code_url": "data:image/png;base64,iVBORw0KGgo...",
    "validation_code": "A1B2C3D4",
    "whatsapp_link": "https://wa.me/551198765432?text=Validar:%20A1B2C3D4",
    "expires_in": 900,
    "expires_at": "2026-05-15T14:30:00Z"
  }
}
```

### 2. Obter Link WhatsApp Direto

```http
GET /api/qrcode/link?code=A1B2C3D4
```

**Response:**
```json
{
  "success": true,
  "data": {
    "whatsapp_link": "https://wa.me/551198765432?text=Validar:%20A1B2C3D4",
    "phone_number": "551198765432"
  }
}
```

### 3. Verificar Validação (Webhook interno)

```http
POST /api/qrcode/verify
Content-Type: application/json

{
  "validation_code": "A1B2C3D4",
  "phone_number": "5511987654321",
  "user_name": "João Silva"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "validated": true,
    "user_id": "uuid-123",
    "token": "eyJhbGc...",
    "expires_in": 604800,
    "message": "User registered and validated successfully"
  }
}
```

### 4. Verificar Status em Tempo Real

```http
GET /api/qrcode/status/abc123def456
```

**Response:**
```json
{
  "success": true,
  "data": {
    "qr_code_id": "abc123def456",
    "status": "validated",
    "validated": true,
    "user_id": "uuid-123",
    "token": "eyJhbGc...",
    "created_at": "2026-05-15T14:15:00Z",
    "validated_at": "2026-05-15T14:18:30Z"
  }
}
```

Se não validado ainda:
```json
{
  "success": true,
  "data": {
    "qr_code_id": "abc123def456",
    "status": "pending",
    "validated": false,
    "expires_in": 750,
    "created_at": "2026-05-15T14:15:00Z"
  }
}
```

---

## 🔧 Configuração WhatsApp

### Passo 1: Obter Número de Negócio

1. Acesse https://www.whatsapp.com/business/
2. Faça login com sua conta Meta
3. Crie uma conta de negócio
4. Configure número de telefone

### Passo 2: Adicionar ao Contato

Para que apareça com logo e nome:

1. Meta Business → WhatsApp Business Accounts
2. Configure Display Name: "NexCash"
3. Adicione foto de perfil (logo)
4. Configure About: "Assistente Financeiro Inteligente"

### Passo 3: Habilitar Mensagens

1. Acesse Meta for Developers
2. Ative "Starting Messages"
3. Usuarios podem iniciar conversa com seu negócio

---

## 💻 Frontend (React)

```jsx
import QRCodeValidation from './components/QRCodeValidation';

function LoginPage() {
  return (
    <div className="login-container">
      <QRCodeValidation />
    </div>
  );
}
```

### Componente Exibe:

```
┌──────────────────────────────────┐
│     🚀 NexCash - Login           │
├──────────────────────────────────┤
│                                  │
│      ┌────────────────────┐     │
│      │                    │     │
│      │   [QR CODE HERE]   │     │
│      │                    │     │
│      └────────────────────┘     │
│                                  │
│   Código: A1B2C3D4  [Copiar]    │
│                                  │
│   [Abrir WhatsApp]              │
│                                  │
│   ⏱️ Expira em: 14 minutos      │
│                                  │
│   Status: ⏳ Aguardando...      │
│                                  │
│   💡 Escaneie com seu celular   │
│      ou clique "Abrir WhatsApp" │
│                                  │
└──────────────────────────────────┘
```

---

## 🔒 Segurança

### ✅ O que é validado:

1. ✓ Código expira em 15 minutos
2. ✓ Código é único e aleatório
3. ✓ Número WhatsApp é verificado
4. ✓ Máximo 3 tentativas falhadas
5. ✓ IP é registrado para análise
6. ✓ JWT com assinatura segura

### 🛡️ Proteções:

- Rate limiting: 5 tentativas/min
- Códigos case-sensitive
- Dados sensíveis criptografados
- Logs de validação para auditoria
- HTTPS obrigatório em produção

---

## 📊 Banco de Dados

### Tabela: qr_codes

```sql
CREATE TABLE qr_codes (
  id UUID PRIMARY KEY,
  validation_code VARCHAR(8) UNIQUE NOT NULL,
  whatsapp_number VARCHAR(20),
  status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP NOT NULL,
  validated_at TIMESTAMP,
  user_id UUID REFERENCES users(id),
  ip_address VARCHAR(45),
  attempt_count INT DEFAULT 0
);
```

### Tabela: validation_sessions (Redis)

```
Key: qr_code:abc123def456
Value: {
  "validation_code": "A1B2C3D4",
  "status": "pending|validated",
  "created_at": "2026-05-15T14:15:00Z",
  "expires_at": "2026-05-15T14:30:00Z",
  "user_id": "uuid-123",
  "token": "eyJhbGc...",
  "attempts": 0
}
TTL: 900 segundos (15 minutos)
```

---

## 🧪 Testando Localmente

### 1. Com Ngrok

```bash
# Terminal 1: Ngrok
ngrok http 3001

# Terminal 2: Backend
cd backend
npm run dev

# Terminal 3: Testar
curl http://localhost:3001/api/qrcode/generate
```

### 2. Simular WhatsApp

```bash
# Enviar validação simulada
curl -X POST http://localhost:3001/api/qrcode/verify \
  -H "Content-Type: application/json" \
  -d '{
    "validation_code": "A1B2C3D4",
    "phone_number": "5511987654321",
    "user_name": "João Silva"
  }'
```

### 3. Verificar Status

```bash
curl http://localhost:3001/api/qrcode/status/abc123def456
```

---

## 🚀 Deploy em Produção

### 1. Variáveis Obrigatórias no `.env`

```env
WHATSAPP_BUSINESS_PHONE=551140411234
WHATSAPP_PHONE_NUMBER_ID=102234567890123
WHATSAPP_ACCESS_TOKEN=seu_token_acesso
WHATSAPP_WEBHOOK_VERIFY_TOKEN=seu_token_webhook
JWT_SECRET=sua_chave_super_secreta
REDIS_HOST=sua_url_redis
DB_HOST=sua_url_banco
WEBHOOK_URL=https://seu-dominio.com/api/whatsapp/webhook
```

### 2. Configurar Webhook no Meta Dashboard

- URL: `https://seu-dominio.com/api/whatsapp/webhook`
- Token: Mesmo do `WHATSAPP_WEBHOOK_VERIFY_TOKEN`
- Eventos: `messages`, `message_template_status_update`

### 3. Testar Webhook

```bash
curl -X GET "https://seu-dominio.com/api/whatsapp/webhook?hub.mode=subscribe&hub.challenge=test_challenge&hub.verify_token=seu_token"
```

Deve retornar: `test_challenge`

---

## 📞 Suporte

### Problemas Comuns

**P: QR Code não aparece**
- A: Verifique se a API de QR Code está rodando
- Cache do navegador pode estar velho (Ctrl+Shift+R)

**P: Link WhatsApp não funciona**
- A: Verifique o número de telefone (formato correto)
- Certifique-se que WhatsApp está instalado no celular

**P: Código não valida**
- A: Pode ter expirado (15 min)
- Gere um novo QR Code
- Verifique se o número WhatsApp está correto

**P: Webhook não recebe mensagens**
- A: URL deve ser HTTPS
- Verifique firewall/proxy
- Teste com https://webhook.site

---

## ✅ Checklist Final

- [ ] Backend rodando em `localhost:3001`
- [ ] Redis conectado e funcionando
- [ ] PostgreSQL com tabelas criadas
- [ ] OpenAI API key configurada
- [ ] Meta WhatsApp credenciais obtidas
- [ ] Ngrok ativo e URL configurada
- [ ] Webhook registrado no Meta Dashboard
- [ ] Frontend rodando em `localhost:3000`
- [ ] QR Code gerando corretamente
- [ ] Validação de código funcionando
- [ ] JWT sendo gerado após validação
- [ ] Usuário logado automaticamente

**Tudo pronto! 🎉 Seu sistema de QR Code com validação WhatsApp está funcionando!**

