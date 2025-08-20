# 🔐 Documentação Completa - Rotas de Autenticação

## Visão Geral
O arquivo `routes/auth.js` contém todas as rotas relacionadas à autenticação, verificação de email, recuperação de senha, 2FA e notificações por email.

## Configurações e Middleware
- **Rate Limiting**: Aplicado para emails e login com limites diferentes para desenvolvimento e produção
- **JWT Secret**: Configurado via variável de ambiente `JWT_SECRET`
- **Middleware de autenticação**: `authenticateToken` para rotas protegidas

---

## 🔐 ROTAS DE AUTENTICAÇÃO PRINCIPAL

### POST /api/auth/login
**Descrição**: Realiza login do usuário com email e senha

**Dados Esperados**:
```json
{
  "email": "usuario@email.com",
  "password": "senha123"
}
```

**Validações**:
- Email e senha obrigatórios
- Formato de email válido
- Verificação de hash da senha com bcrypt

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "user": {
    "id": "uuid",
    "email": "usuario@email.com",
    "name": "Nome Completo",
    "role": "user",
    "emailVerified": true,
    "profileCompleted": true,
    "createdAt": "2025-01-01T00:00:00Z",
    "updatedAt": "2025-01-01T00:00:00Z"
  },
  "token": "jwt_token_aqui",
  "dashboardData": {
    "wallet": {
      "balance": 1500.50,
      "availableBalance": 1400.50,
      "totalEarnings": 2000.00,
      "totalWithdrawals": 500.00,
      "pendingWithdrawals": 100.00,
      "transactions": [...]
    },
    "sales": {
      "totalRevenue": 5000.00,
      "totalOrders": 25,
      "completedOrders": 20,
      "pendingOrders": 5,
      "uniqueCustomers": 18,
      "conversionRate": 80.00
    },
    "products": {
      "total": 8,
      "list": [...],
      "topSelling": [...]
    },
    "recovery": {
      "abandonedCarts": 12,
      "potentialRevenue": 1200
    },
    "analytics": {
      "recentEvents": [...],
      "topUtmSources": {...},
      "topUtmMediums": {...}
    },
    "profile": {...},
    "metadata": {
      "lastLogin": "2025-01-01T00:00:00Z",
      "accountType": "user",
      "memberSince": "2024-01-01T00:00:00Z"
    }
  }
}
```

**Resposta com 2FA (200)**:
```json
{
  "success": false,
  "requires2FA": true,
  "message": "Login parcial realizado. É necessário fornecer o código 2FA.",
  "type": "2fa_required",
  "instructions": "Use o endpoint POST /api/auth/login-2fa com email, senha e token 2FA"
}
```

**Erros Possíveis**:
- `400`: Email/senha obrigatórios, formato inválido
- `401`: Email ou senha incorretos
- `429`: Rate limit atingido
- `500`: Erro interno

---

### POST /api/auth/register
**Descrição**: Registro completo de usuário com dados LGPD e KYC

**Dados Esperados**:
```json
{
  "firstName": "João",
  "lastName": "Silva",
  "email": "joao@email.com",
  "password": "senha123",
  "confirmPassword": "senha123",
  "cpf": "123.456.789-00",
  "cnpj": "12.345.678/0001-90",
  "phone": "(11) 99999-9999",
  "birthDate": "1990-01-01",
  "userType": "pf",
  "address": {
    "street": "Rua das Flores, 123",
    "city": "São Paulo",
    "state": "SP",
    "zipCode": "01234-567"
  },
  "companyData": {
    "name": "Empresa LTDA",
    "cnpj": "12.345.678/0001-90"
  },
  "selectedPlan": "basic",
  "selectedPaymentPlan": "monthly",
  "termsAccepted": true,
  "lgpdConsent": {
    "dataProcessing": true,
    "marketing": false
  },
  "registrationSource": "web",
  "registrationIp": "192.168.1.1",
  "registrationUserAgent": "Mozilla/5.0..."
}
```

**Validações**:
- Dados pessoais básicos obrigatórios
- `userType` deve ser "pf" ou "pj"
- CPF obrigatório para PF, CNPJ para PJ
- Consentimentos LGPD obrigatórios
- Senha mínima de 8 caracteres
- Confirmação de senha
- Email único no sistema
- CPF/CNPJ único

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Usuário registrado com sucesso. Verifique seu email para continuar.",
  "user": {
    "id": "uuid",
    "email": "joao@email.com",
    "name": "João Silva",
    "firstName": "João",
    "lastName": "Silva",
    "userType": "pf",
    "role": "user",
    "emailVerified": false,
    "profileCompleted": false,
    "kycStatus": "not_started",
    "selectedPlan": "basic",
    "selectedPaymentPlan": "monthly",
    "createdAt": "2025-01-01T00:00:00Z",
    "updatedAt": "2025-01-01T00:00:00Z"
  },
  "token": "jwt_token_aqui",
  "nextStep": "email_verification"
}
```

**Erros Possíveis**:
- `400`: Dados obrigatórios ausentes, formato inválido
- `409`: Email/CPF/CNPJ já em uso
- `500`: Erro interno

---

### POST /api/auth/logout
**Descrição**: Realiza logout do usuário

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Logout realizado com sucesso"
}
```

---

## 📧 ROTAS DE VERIFICAÇÃO DE EMAIL

### POST /api/auth/send-verification-email
**Descrição**: Envia código de verificação por email

**Rate Limit**: 3 tentativas por 5 min (prod) / 20 por 1 min (dev)

**Dados Esperados**:
```json
{
  "email": "usuario@email.com"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Código de verificação enviado com sucesso",
  "email": "usuario@email.com",
  "expiresIn": 300
}
```

**Erros Possíveis**:
- `400`: Email obrigatório, formato inválido, já verificado
- `404`: Usuário não encontrado
- `429`: Rate limit atingido
- `500`: Erro no envio

---

### POST /api/auth/verify-email-code
**Descrição**: Verifica código de verificação de email

**Dados Esperados**:
```json
{
  "email": "usuario@email.com",
  "code": "123456"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Email verificado com sucesso",
  "email": "usuario@email.com",
  "nextStep": "kyc_upload"
}
```

**Erros Possíveis**:
- `400`: Email/código obrigatórios, já verificado, código inválido
- `404`: Usuário não encontrado
- `500`: Erro interno

---

## 🔑 ROTAS DE RECUPERAÇÃO DE SENHA

### POST /api/auth/forgot-password
**Descrição**: Envia link de recuperação de senha

**Rate Limit**: Aplicado

**Dados Esperados**:
```json
{
  "email": "usuario@email.com",
  "userName": "João Silva"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Link de recuperação enviado com sucesso",
  "email": "usuario@email.com"
}
```

---

### POST /api/auth/verify-reset-token
**Descrição**: Verifica validade do token de reset

**Dados Esperados**:
```json
{
  "token": "reset_token_here"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Token válido",
  "email": "usuario@email.com"
}
```

---

### POST /api/auth/reset-password
**Descrição**: Redefine senha com token válido

**Dados Esperados**:
```json
{
  "token": "reset_token_here",
  "newPassword": "nova_senha123"
}
```

**Validações**:
- Senha mínima de 6 caracteres
- Token válido e não usado

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Senha redefinida com sucesso",
  "email": "usuario@email.com"
}
```

---

## 📧 ROTAS DE NOTIFICAÇÃO POR EMAIL

### POST /api/auth/send-welcome
**Descrição**: Envia email de boas-vindas

**Dados Esperados**:
```json
{
  "email": "usuario@email.com",
  "userName": "João Silva"
}
```

---

### POST /api/auth/send-product-notification
**Descrição**: Notifica vendedor sobre produto criado

**Dados Esperados**:
```json
{
  "sellerEmail": "vendedor@email.com",
  "sellerName": "João Vendedor",
  "product": {
    "name": "Curso de JavaScript",
    "price": 199.99,
    "id": "product_id"
  }
}
```

---

### POST /api/auth/send-order-confirmation
**Descrição**: Confirma pedido para cliente

**Dados Esperados**:
```json
{
  "customerEmail": "cliente@email.com",
  "customerName": "Maria Cliente",
  "order": {
    "id": "order_123",
    "amount": 199.99
  },
  "product": {
    "name": "Curso de JavaScript"
  }
}
```

---

### POST /api/auth/send-sale-notification
**Descrição**: Notifica vendedor sobre nova venda

**Dados Esperados**:
```json
{
  "sellerEmail": "vendedor@email.com",
  "sellerName": "João Vendedor",
  "order": {
    "id": "order_123",
    "amount": 199.99
  },
  "product": {
    "name": "Curso de JavaScript"
  }
}
```

---

## 🔐 ROTAS DE TWO-FACTOR AUTHENTICATION (2FA)

### POST /api/auth/2fa/setup-google
**Descrição**: Configura Google Authenticator (protegida)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "status": "success",
  "message": "Secret 2FA gerado com sucesso",
  "data": {
    "secret": "JBSWY3DPEHPK3PXP",
    "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...",
    "manualEntryKey": "JBSWY3DPEHPK3PXP",
    "instructions": {
      "step1": "Baixe o Google Authenticator ou similar",
      "step2": "Escaneie o QR Code ou digite a chave manualmente",
      "step3": "Use o endpoint /api/auth/2fa/verify-google para confirmar"
    }
  }
}
```

---

### POST /api/auth/2fa/verify-google
**Descrição**: Verifica código e ativa 2FA (protegida)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Dados Esperados**:
```json
{
  "token": "123456"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "status": "success",
  "message": "2FA ativado com sucesso",
  "data": {
    "backupCodes": [
      "ABC123DEF", "GHI456JKL", "MNO789PQR",
      "STU012VWX", "YZ345ABC", "DEF678GHI",
      "JKL901MNO", "PQR234STU", "VWX567YZ", "ABC890DEF"
    ],
    "warning": "Guarde estes códigos de backup em local seguro. Eles são a única forma de acessar sua conta se você perder o acesso ao Google Authenticator."
  }
}
```

---

### POST /api/auth/login-2fa
**Descrição**: Login com verificação 2FA

**Dados Esperados**:
```json
{
  "email": "usuario@email.com",
  "password": "senha123",
  "token": "123456"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "status": "success",
  "message": "Login com 2FA realizado com sucesso",
  "data": {
    "token": "jwt_token_here",
    "user": {
      "id": "user_id",
      "name": "João Silva",
      "email": "usuario@email.com",
      "role": "user"
    },
    "authMethod": "totp",
    "remainingBackupCodes": 8
  }
}
```

**Observações**:
- Aceita códigos TOTP (Google Authenticator) ou códigos de backup
- Códigos de backup são removidos após uso
- `authMethod` pode ser "totp" ou "backup_code"

---

### DELETE /api/auth/2fa/disable
**Descrição**: Desabilita 2FA (protegida)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Dados Esperados**:
```json
{
  "password": "senha_atual"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "status": "success",
  "message": "2FA desabilitado com sucesso",
  "data": {
    "warning": "Sua conta agora está menos segura. Recomendamos reativar o 2FA."
  }
}
```

---

## 👤 ROTAS DE PERFIL DO USUÁRIO

### GET /api/auth/me
**Descrição**: Obtém informações do usuário atual (protegida)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "user": {
    "id": "user_id",
    "email": "usuario@email.com",
    "firstName": "João",
    "lastName": "Silva",
    "role": "user",
    "profileCompleted": true,
    "hasFullProfile": true,
    "avatar": "url_do_avatar",
    "phone": "(11) 99999-9999",
    "bio": "Desenvolvedor apaixonado por tecnologia",
    "emailVerified": true,
    "twoFactorEnabled": true,
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2025-01-01T00:00:00Z"
  }
}
```

---

### PUT /api/auth/profile
**Descrição**: Atualiza perfil do usuário (protegida)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Dados Esperados**:
```json
{
  "firstName": "João",
  "lastName": "Silva Santos",
  "phone": "(11) 98888-8888",
  "bio": "Nova biografia",
  "avatar": "nova_url_avatar"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Perfil atualizado com sucesso",
  "user": {
    "id": "user_id",
    "email": "usuario@email.com",
    "firstName": "João",
    "lastName": "Silva Santos",
    "role": "user",
    "profileCompleted": true,
    "hasFullProfile": true,
    "avatar": "nova_url_avatar",
    "phone": "(11) 98888-8888",
    "bio": "Nova biografia",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2025-01-01T00:00:00Z"
  }
}
```

**Observações**:
- Atualiza apenas campos fornecidos
- Recalcula `profile_completed` automaticamente
- Perfil considerado completo com: firstName, lastName e phone

---

## 🧪 ROTAS DE TESTE

### GET /api/auth/test-email
**Descrição**: Testa configuração de email (apenas desenvolvimento)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "emailConfigured": true,
  "environment": "development",
  "smtpHost": "smtp.gmail.com",
  "smtpUser": "user@gmail.com",
  "fromEmail": "noreply@app.com",
  "fromName": "CheckoutPro"
}
```

**Erros**:
- `403`: Endpoint não disponível em produção

---

## 📊 Rate Limiting

### Configuração por Ambiente

**Desenvolvimento**:
- Email: 20 tentativas por 1 minuto
- Login: 50 tentativas por 5 minutos

**Produção**:
- Email: 3 tentativas por 5 minutos  
- Login: 5 tentativas por 15 minutos

### Resposta de Rate Limit (429):
```json
{
  "error": "Muitas tentativas de login. Tente novamente em 15 minutos.",
  "type": "login_rate_limit"
}
```

---

## 🔒 Segurança e Boas Práticas

1. **Hash de Senha**: bcrypt com salt rounds 12
2. **JWT**: Expiração de 7 dias para login normal, 24h para 2FA
3. **Rate Limiting**: Proteção contra ataques de força bruta
4. **Validação**: Sanitização e validação de todos os inputs
5. **2FA**: Suporte completo com códigos de backup
6. **LGPD**: Coleta e armazenamento de consentimentos
7. **Logs**: Registros detalhados para auditoria
8. **Ambiente**: Configurações diferentes para dev/prod

## 🚨 Códigos de Erro Comuns

- `validation_error`: Dados inválidos ou ausentes
- `auth_error`: Falha na autenticação
- `user_not_found`: Usuário não existe
- `already_verified`: Email já verificado
- `2fa_required`: Necessário código 2FA
- `invalid_2fa_code`: Código 2FA inválido
- `email_error`: Falha no envio de email
- `server_error`: Erro interno do servidor
- `rate_limit`: Limite de tentativas atingido

---

**Próxima documentação**: `routes/users.js` - Gestão de usuários e perfis avançados
