# ⚙️ Documentação Completa - Rotas de Configurações

## Visão Geral
O arquivo `routes/settings.js` contém o sistema de configurações globais da plataforma, permitindo gerenciar configurações do sistema, notificações, pagamentos e outras funcionalidades. Todas as configurações são armazenadas de forma persistente e podem ser atualizadas dinamicamente.

## Funcionalidades Principais
- **Configurações Globais**: Gerenciamento de todas as configurações do sistema
- **Configurações por Categoria**: Sistema, notificações, pagamentos
- **Valores Padrão**: Fallback automático para configurações não definidas
- **Transações Atômicas**: Atualizações seguras de múltiplas configurações
- **Controle de Acesso**: Diferenciação entre admin e usuários comuns

## Categorias de Configuração
- **System**: Configurações gerais do sistema
- **Notifications**: Configurações de notificações
- **Payment**: Configurações de pagamento e taxas

---

## 🔧 CONFIGURAÇÕES GLOBAIS

### GET /api/settings
**Descrição**: Busca todas as configurações do sistema

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "platformFee": 5,
  "payoutDelayDays": 7,
  "enableNewUserRegistration": true,
  "cardFeePercent": 3.5,
  "transactionFixedFee": 1.0,
  "withdrawalFixedFee": 5.0,
  "pixFeePercent": 1.5,
  "pixFixedFee": 0.50,
  "installmentFees": {
    "2x": 0.5,
    "3x": 0.8,
    "4x": 1.0,
    "5x": 1.2,
    "6x": 1.5,
    "7x": 1.7,
    "8x": 2.0,
    "9x": 2.2,
    "10x": 2.5,
    "11x": 2.7,
    "12x": 3.0
  }
}
```

**Configurações Padrão**:
- `platformFee`: Taxa da plataforma (%)
- `payoutDelayDays`: Dias para liberação de pagamento
- `enableNewUserRegistration`: Permite novos cadastros
- `cardFeePercent`: Taxa do cartão de crédito (%)
- `transactionFixedFee`: Taxa fixa por transação
- `withdrawalFixedFee`: Taxa fixa de saque
- `pixFeePercent`: Taxa do PIX (%)
- `pixFixedFee`: Taxa fixa do PIX
- `installmentFees`: Taxas por parcela

**Uso**: Inicialização do sistema e cálculos de taxas

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### PUT /api/settings
**Descrição**: Atualiza múltiplas configurações do sistema

**Autenticação**: Obrigatória

**Body Parameters**:
```json
{
  "platformFee": 6,
  "payoutDelayDays": 5,
  "cardFeePercent": 4.0,
  "pixFeePercent": 2.0,
  "installmentFees": {
    "2x": 0.6,
    "3x": 0.9,
    "4x": 1.2,
    "5x": 1.5,
    "6x": 1.8
  }
}
```

**Resposta de Sucesso (200)**:
```json
{
  "platformFee": 6,
  "payoutDelayDays": 5,
  "cardFeePercent": 4.0,
  "pixFeePercent": 2.0,
  "installmentFees": {
    "2x": 0.6,
    "3x": 0.9,
    "4x": 1.2,
    "5x": 1.5,
    "6x": 1.8
  }
}
```

**Funcionalidades**:
- **Transação Atômica**: Todas as configurações são salvas ou nenhuma
- **Upsert**: Cria nova configuração ou atualiza existente
- **Tipo Flexível**: Suporta strings, números, objetos e arrays
- **Serialização JSON**: Objetos complexos são salvos como JSON

**Uso**: Painel administrativo para ajustar configurações

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor ou falha na transação

---

## 🔍 CONFIGURAÇÃO ESPECÍFICA

### GET /api/settings/:key
**Descrição**: Busca configuração específica por chave

**Parâmetros**:
- `key` (string): Chave da configuração

**Autenticação**: Obrigatória

**Exemplo**: `/api/settings/system`

**Resposta de Sucesso (200)**:
```json
{
  "status": "success",
  "key": "system",
  "value": {
    "maintenanceMode": false,
    "enableNewRegistrations": true,
    "maxUsersPerIP": 5,
    "sessionTimeout": 24
  },
  "timestamp": "2024-12-31T21:30:00.000Z"
}
```

**Valores Padrão por Chave**:

**System**:
```json
{
  "maintenanceMode": false,
  "enableNewRegistrations": true,
  "maxUsersPerIP": 5,
  "sessionTimeout": 24
}
```

**Notifications**:
```json
{
  "emailNotifications": true,
  "smsNotifications": false,
  "pushNotifications": true,
  "marketingEmails": false
}
```

**Payment**:
```json
{
  "enablePix": true,
  "enableCreditCard": true,
  "pixFeePercent": 2.5,
  "cardFeePercent": 4.5,
  "minWithdrawAmount": 50
}
```

**Uso**: Buscar configurações específicas para funcionalidades

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### DELETE /api/settings/:key
**Descrição**: Remove configuração específica (retorna ao valor padrão)

**Parâmetros**:
- `key` (string): Chave da configuração

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "status": "success",
  "message": "Configuração 'system' deletada com sucesso",
  "timestamp": "2024-12-31T21:35:00.000Z"
}
```

**Resposta de Erro - Não Encontrada (404)**:
```json
{
  "status": "error",
  "message": "Configuração 'inexistente' não encontrada",
  "timestamp": "2024-12-31T21:35:00.000Z"
}
```

**Uso**: Reset de configurações para valores padrão

**Erros Possíveis**:
- `404`: Configuração não encontrada
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 🖥️ CONFIGURAÇÕES DO SISTEMA

### GET /api/settings/system
**Descrição**: Busca configurações específicas do sistema

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "settings": {
    "maintenanceMode": false,
    "enableNewRegistrations": true,
    "maxUsersPerIP": 5,
    "sessionTimeout": 24
  }
}
```

**Configurações do Sistema**:
- `maintenanceMode`: Modo de manutenção ativo
- `enableNewRegistrations`: Permite novos cadastros
- `maxUsersPerIP`: Máximo de usuários por IP
- `sessionTimeout`: Timeout da sessão em horas

**Uso**: Verificar estado geral do sistema

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### PUT /api/settings/system
**Descrição**: Atualiza configurações do sistema (apenas admin)

**Autenticação**: Obrigatória (Admin)

**Body Parameters**:
```json
{
  "settings": {
    "maintenanceMode": true,
    "enableNewRegistrations": false,
    "maxUsersPerIP": 3,
    "sessionTimeout": 12
  }
}
```

**Ou diretamente**:
```json
{
  "maintenanceMode": true,
  "enableNewRegistrations": false,
  "maxUsersPerIP": 3,
  "sessionTimeout": 12
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Configurações do sistema atualizadas com sucesso"
}
```

**Controle de Acesso**: Apenas usuários com role "admin"

**Resposta de Erro - Acesso Negado (403)**:
```json
{
  "success": false,
  "error": "Acesso negado"
}
```

**Uso**: Painel administrativo para controle do sistema

**Erros Possíveis**:
- `403`: Usuário não é administrador
- `400`: Configurações não fornecidas ou vazias
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 🔔 CONFIGURAÇÕES DE NOTIFICAÇÕES

### GET /api/settings/notifications
**Descrição**: Busca configurações de notificações

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "settings": {
    "emailEnabled": true,
    "smsEnabled": false,
    "pushEnabled": true,
    "orderConfirmation": true,
    "paymentNotifications": true,
    "kycUpdates": true
  }
}
```

**Configurações de Notificações**:
- `emailEnabled`: Notificações por email ativas
- `smsEnabled`: Notificações por SMS ativas
- `pushEnabled`: Push notifications ativas
- `orderConfirmation`: Confirmar pedidos por notificação
- `paymentNotifications`: Notificar sobre pagamentos
- `kycUpdates`: Notificar sobre updates de KYC

**Uso**: Controlar tipos de notificações enviadas

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### PUT /api/settings/notifications
**Descrição**: Atualiza configurações de notificações (apenas admin)

**Autenticação**: Obrigatória (Admin)

**Body Parameters**:
```json
{
  "settings": {
    "emailEnabled": true,
    "smsEnabled": true,
    "pushEnabled": false,
    "orderConfirmation": true,
    "paymentNotifications": true,
    "kycUpdates": false
  }
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Configurações de notificação atualizadas com sucesso"
}
```

**Controle de Acesso**: Apenas usuários com role "admin"

**Uso**: Ajustar canais e tipos de notificação

**Erros Possíveis**:
- `403`: Usuário não é administrador
- `400`: Configurações não fornecidas ou vazias
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 💳 CONFIGURAÇÕES DE PAGAMENTO

### GET /api/settings/payment
**Descrição**: Busca configurações de pagamento

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "settings": {
    "pixEnabled": true,
    "cardEnabled": true,
    "boletoEnabled": false,
    "installmentsEnabled": true,
    "maxInstallments": 12,
    "minInstallmentAmount": 10
  }
}
```

**Configurações de Pagamento**:
- `pixEnabled`: PIX habilitado
- `cardEnabled`: Cartão de crédito habilitado
- `boletoEnabled`: Boleto bancário habilitado
- `installmentsEnabled`: Parcelamento habilitado
- `maxInstallments`: Número máximo de parcelas
- `minInstallmentAmount`: Valor mínimo por parcela

**Uso**: Controlar métodos de pagamento disponíveis

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### PUT /api/settings/payment
**Descrição**: Atualiza configurações de pagamento (apenas admin)

**Autenticação**: Obrigatória (Admin)

**Body Parameters**:
```json
{
  "settings": {
    "pixEnabled": true,
    "cardEnabled": true,
    "boletoEnabled": true,
    "installmentsEnabled": true,
    "maxInstallments": 10,
    "minInstallmentAmount": 15
  }
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Configurações de pagamento atualizadas com sucesso"
}
```

**Controle de Acesso**: Apenas usuários com role "admin"

**Impacto das Configurações**:
- **pixEnabled**: Mostra/oculta PIX no checkout
- **cardEnabled**: Habilita/desabilita cartão de crédito
- **maxInstallments**: Limita opções de parcelamento
- **minInstallmentAmount**: Calcula parcelas mínimas permitidas

**Uso**: Configurar métodos de pagamento da plataforma

**Erros Possíveis**:
- `403`: Usuário não é administrador
- `400`: Configurações não fornecidas ou vazias
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 🔧 Estrutura de Dados

### Modelo Setting
```javascript
{
  id: 'uuid',
  key: 'string',        // Chave única da configuração
  value: 'text',        // Valor serializado (JSON ou string)
  created_at: 'timestamp',
  updated_at: 'timestamp'
}
```

### Serialização de Valores
```javascript
// Objetos e arrays são serializados como JSON
const objectValue = { foo: 'bar', array: [1, 2, 3] };
await Setting.upsert({
  key: 'config',
  value: JSON.stringify(objectValue)
});

// Strings e números são salvos como string
const stringValue = 'simple value';
await Setting.upsert({
  key: 'simple',
  value: String(stringValue)
});
```

### Deserialização na Leitura
```javascript
// Tenta fazer parse JSON, fallback para string
try {
  settings[row.key] = JSON.parse(row.value);
} catch (e) {
  settings[row.key] = row.value;
}
```

---

## 🔒 Controle de Acesso

### Níveis de Permissão
- **Leitura**: Todos os usuários autenticados
- **Escrita**: Apenas administradores (para rotas específicas)
- **Global**: Administradores podem alterar qualquer configuração

### Middleware de Admin
```javascript
if (req.user.role !== 'admin') {
  return res.status(403).json({
    success: false,
    error: 'Acesso negado'
  });
}
```

### Validação de Dados
```javascript
if (!settingsData || Object.keys(settingsData).length === 0) {
  return res.status(400).json({
    success: false,
    error: 'Configurações não fornecidas ou vazias'
  });
}
```

---

## 📊 Casos de Uso

### Para Administradores
- **Manutenção**: Ativar modo de manutenção
- **Controle de Registro**: Habilitar/desabilitar novos usuários
- **Ajuste de Taxas**: Modificar taxas de pagamento
- **Configuração de Canais**: Controlar notificações

### Para o Sistema
- **Cálculo de Taxas**: Usar configurações para calcular fees
- **Validação de Limites**: Verificar limites de parcelamento
- **Controle de Funcionalidades**: Habilitar/desabilitar recursos
- **Fallback**: Valores padrão quando configuração não existe

### Para Desenvolvimento
- **Feature Flags**: Controlar funcionalidades em produção
- **A/B Testing**: Testar diferentes configurações
- **Configuração Dinâmica**: Ajustar sem deploy
- **Debug**: Configurações específicas para debug

---

## 🚀 Configurações Futuras

### Configurações de Usuário
```javascript
const userSettings = {
  theme: 'dark',
  language: 'pt-BR',
  timezone: 'America/Sao_Paulo',
  emailFrequency: 'daily'
};
```

### Configurações de Marketing
```javascript
const marketingSettings = {
  enableCoupons: true,
  defaultDiscountPercent: 10,
  maxCouponUses: 100,
  loyaltyProgram: true
};
```

### Configurações de Segurança
```javascript
const securitySettings = {
  require2FA: false,
  sessionTimeout: 24,
  maxLoginAttempts: 5,
  passwordExpiryDays: 90
};
```

### Feature Flags
```javascript
const featureFlags = {
  newCheckoutFlow: true,
  aiRecommendations: false,
  advancedAnalytics: true,
  mobileApp: false
};
```

---

## 📈 Monitoramento

### Logs de Configuração
```javascript
console.log('✅ Configurações carregadas:', Object.keys(finalSettings));
console.log('💾 Atualizando configurações do sistema...');
console.log('✅ Configurações atualizadas:', Object.keys(settingsData));
```

### Auditoria de Mudanças
- Registrar quem alterou configurações
- Quando foram alteradas
- Valores anteriores vs novos
- Impacto das mudanças

### Alertas Automáticos
- Modo de manutenção ativado
- Registros desabilitados
- Taxas alteradas significativamente
- Métodos de pagamento desabilitados

---

**Próxima documentação**: `routes/tracking.js` - Sistema de tracking e analytics
