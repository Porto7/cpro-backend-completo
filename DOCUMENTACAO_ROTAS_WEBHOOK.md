# 🔗 Documentação Completa - Rotas de Webhook

## Visão Geral
O arquivo `routes/webhook.js` contém webhooks para processamento automático de pagamentos e matrículas em cursos. Sistema responsável por receber notificações de gateways de pagamento e executar ações automáticas no sistema.

## Funcionalidades Principais
- **Pagamento Aprovado**: Matrícula automática em cursos
- **Pagamento Rejeitado**: Atualização de status de pedidos
- **Renovação de Assinatura**: Extensão automática de acesso
- **Sistema de Logs**: Rastreamento completo de operações

## Gateways Suportados
- Mercado Pago
- Stripe
- Hotmart
- Outros (configuráveis)

---

## 💰 PAGAMENTO APROVADO

### POST /api/webhook/payment-approved
**Descrição**: Processa pagamentos aprovados e executa matrícula automática em cursos

**Autenticação**: Não obrigatória (endpoint público para webhooks)

**Content-Type**: `application/json`

**Body Parameters**:
```json
{
  "orderId": "uuid-pedido-123",
  "transactionId": "txn_abc123def456",
  "paymentMethod": "credit_card",
  "amount": 197.50,
  "currency": "BRL",
  "status": "approved",
  "customer": {
    "id": "customer_123",
    "email": "cliente@email.com",
    "name": "João Silva"
  },
  "metadata": {
    "source": "checkout_page",
    "campaign": "black_friday"
  },
  "gateway": "mercadopago"
}
```

**Parâmetros Obrigatórios**:
- `orderId` (string): ID do pedido no sistema
- `transactionId` (string): ID da transação no gateway
- `status` (string): Status do pagamento (deve ser "approved")

**Parâmetros Opcionais**:
- `paymentMethod` (string): Método de pagamento usado
- `amount` (number): Valor pago
- `currency` (string): Moeda (padrão: BRL)
- `customer` (object): Dados do cliente
- `metadata` (object): Metadados adicionais
- `gateway` (string): Gateway utilizado

**Resposta de Sucesso - Nova Matrícula (200)**:
```json
{
  "success": true,
  "message": "Matrícula criada com sucesso",
  "data": {
    "enrollmentId": "uuid-matricula-456",
    "courseId": "uuid-curso-789",
    "courseTitle": "Curso de JavaScript Avançado",
    "studentId": "uuid-usuario-321",
    "studentName": "João Silva",
    "enrolledAt": "2024-12-31T18:30:00.000Z",
    "expiresAt": "2025-12-31T18:30:00.000Z",
    "accessUrl": "https://plataforma.com/area-membro/curso/uuid-curso-789"
  }
}
```

**Resposta de Sucesso - Já Processado (200)**:
```json
{
  "success": true,
  "message": "Pagamento já processado",
  "alreadyProcessed": true
}
```

**Resposta de Sucesso - Produto (200)**:
```json
{
  "success": true,
  "message": "Pagamento processado (produto não é curso)",
  "isProduct": true
}
```

**Fluxo de Processamento**:

1. **Validação Inicial**:
   ```javascript
   // Verifica dados obrigatórios
   if (!orderId || !transactionId || status !== 'approved') {
     return res.status(400).json({ error: 'Dados inválidos' });
   }
   ```

2. **Busca do Pedido**:
   ```javascript
   // Busca por ID ou transaction_id
   let order = await Order.findOne({
     where: { 
       [Op.or]: [
         { id: orderId },
         { transaction_id: transactionId }
       ]
     },
     include: [Product, User, Course]
   });
   ```

3. **Verificação de Duplicação**:
   ```javascript
   // Evita processamento duplicado
   if (order.status === 'completed') {
     return res.json({ alreadyProcessed: true });
   }
   ```

4. **Atualização do Pedido**:
   ```javascript
   await order.update({
     status: 'completed',
     payment_method: paymentMethod,
     transaction_id: transactionId,
     payment_data: {
       gateway,
       approvedAt: new Date(),
       webhookData: req.body
     }
   });
   ```

5. **Matrícula no Curso**:
   ```javascript
   // Se produto é um curso
   if (order.product?.course) {
     const enrollment = await CourseEnrollment.create({
       course_id: course.id,
       student_id: user.id,
       order_id: order.id,
       status: 'active',
       enrolled_at: new Date(),
       expires_at: calculateExpiryDate(course.access_duration_days)
     });
   }
   ```

**Funcionalidades Automáticas**:

- **Matrícula Inteligente**: Cria nova matrícula ou reativa existente
- **Cálculo de Expiração**: Baseado na duração do curso
- **Dados de Acesso**: Metadados completos para auditoria
- **Prevenção de Duplicação**: Verificação de status antes do processamento

**Logs Detalhados**:
```javascript
console.log('🔔 Webhook recebido - Pagamento aprovado:', {
  orderId, transactionId, amount, gateway
});
console.log('💰 Pedido atualizado como pago:', orderId);
console.log('✅ Nova matrícula criada:', enrollment.id);
```

**Erros Possíveis**:
- `400`: Dados obrigatórios ausentes ou inválidos
- `404`: Pedido não encontrado
- `500`: Erro interno do servidor

---

## ❌ PAGAMENTO REJEITADO

### POST /api/webhook/payment-failed
**Descrição**: Processa pagamentos rejeitados ou falhados

**Body Parameters**:
```json
{
  "orderId": "uuid-pedido-123",
  "transactionId": "txn_abc123def456",
  "reason": "insufficient_funds",
  "gateway": "mercadopago"
}
```

**Parâmetros**:
- `orderId` (string): ID do pedido
- `transactionId` (string): ID da transação
- `reason` (string): Motivo da rejeição
- `gateway` (string): Gateway utilizado

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Pagamento rejeitado processado"
}
```

**Atualização do Pedido**:
```javascript
await order.update({
  status: 'failed',
  payment_data: {
    ...order.payment_data,
    failureReason: reason,
    failedAt: new Date(),
    gateway
  }
});
```

**Motivos de Rejeição Comuns**:
- `insufficient_funds`: Fundos insuficientes
- `expired_card`: Cartão expirado
- `invalid_card`: Cartão inválido
- `fraud_detected`: Fraude detectada
- `gateway_error`: Erro no gateway

**Uso**: Tracking de tentativas fracassadas e análise de conversão

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 🔄 RENOVAÇÃO DE ASSINATURA

### POST /api/webhook/subscription-renewed
**Descrição**: Processa renovações automáticas de assinaturas

**Body Parameters**:
```json
{
  "subscriptionId": "sub_123abc456def",
  "orderId": "uuid-pedido-456",
  "transactionId": "txn_renewal_789",
  "nextBillingDate": "2025-01-31T18:30:00.000Z",
  "amount": 197.50
}
```

**Parâmetros**:
- `subscriptionId` (string): ID da assinatura
- `orderId` (string): ID do pedido de renovação
- `transactionId` (string): ID da transação de renovação
- `nextBillingDate` (string): Próxima data de cobrança
- `amount` (number): Valor da renovação

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Assinatura renovada"
}
```

**Funcionalidades**:

1. **Busca por Assinatura**:
   ```javascript
   const enrollment = await CourseEnrollment.findOne({
     where: {
       'access_data.subscriptionId': subscriptionId
     }
   });
   ```

2. **Extensão de Acesso**:
   ```javascript
   await enrollment.update({
     expires_at: new Date(nextBillingDate),
     status: 'active',
     access_data: {
       ...enrollment.access_data,
       lastRenewal: new Date(),
       renewalTransactionId: transactionId
     }
   });
   ```

**Logs**:
```javascript
console.log('🔄 Webhook - Assinatura renovada:', subscriptionId);
console.log('📅 Acesso estendido até:', newExpiryDate);
```

**Uso**: Manter acesso contínuo para assinaturas recorrentes

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 🧪 TESTE DE CONECTIVIDADE

### GET /api/webhook/test
**Descrição**: Endpoint para testar conectividade e status dos webhooks

**Autenticação**: Não obrigatória

**Resposta (200)**:
```json
{
  "status": "online",
  "timestamp": "2024-12-31T18:30:00.000Z",
  "version": "1.0.0",
  "endpoints": [
    "POST /payment-approved",
    "POST /payment-failed",
    "POST /subscription-renewed"
  ]
}
```

**Uso**: 
- Verificar se webhooks estão funcionando
- Monitoramento de uptime
- Documentação de endpoints disponíveis

---

## 🔧 Configuração de Webhooks

### URLs de Webhook para Gateways

**Mercado Pago**:
```
POST https://api.seudominio.com/api/webhook/payment-approved
POST https://api.seudominio.com/api/webhook/payment-failed
```

**Stripe**:
```
POST https://api.seudominio.com/api/webhook/payment-approved
POST https://api.seudominio.com/api/webhook/subscription-renewed
```

**Hotmart**:
```
POST https://api.seudominio.com/api/webhook/payment-approved
```

### Configuração de Ambiente
```env
# Frontend URL para links de acesso
FRONTEND_URL=https://seudominio.com

# Database para queries otimizadas
DATABASE_URL=postgresql://...
```

### Headers Recomendados
```javascript
// Para verificação de assinatura (opcional)
const signature = req.headers['x-signature'];
const timestamp = req.headers['x-timestamp'];
```

---

## 📊 Fluxo Completo de Matrícula

### 1. Evento de Pagamento
```
Gateway → Webhook → Sistema
```

### 2. Processamento
```
Validação → Busca Pedido → Verifica Duplicação → Atualiza Status
```

### 3. Matrícula
```
Verifica Curso → Cria/Reativa Matrícula → Calcula Expiração
```

### 4. Dados Criados
```javascript
CourseEnrollment {
  id: 'uuid-matricula-123',
  course_id: 'uuid-curso-456',
  student_id: 'uuid-usuario-789',
  order_id: 'uuid-pedido-321',
  status: 'active',
  enrolled_at: '2024-12-31T18:30:00Z',
  expires_at: '2025-12-31T18:30:00Z',
  access_data: {
    enrolledVia: 'payment',
    paymentMethod: 'credit_card',
    transactionId: 'txn_abc123',
    gateway: 'mercadopago',
    amount: 197.50,
    processedAt: '2024-12-31T18:30:00Z'
  }
}
```

---

## 🚨 Tratamento de Erros

### Logs Detalhados
```javascript
console.error('❌ Erro no webhook de pagamento:', error);
console.error('Dados do webhook:', req.body);
console.error('Stack trace:', error.stack);
```

### Resposta de Erro (500)
```json
{
  "error": "Erro interno do servidor",
  "message": "Database connection failed",
  "timestamp": "2024-12-31T18:30:00.000Z"
}
```

### Estratégias de Recuperação
- **Retry Automático**: Gateways reenviam webhooks
- **Logs Centralizados**: Para debug e auditoria
- **Alertas**: Notificações para erros críticos
- **Fallback**: Processamento manual quando necessário

---

## 📈 Monitoramento e Analytics

### Métricas Importantes
- **Taxa de Sucesso**: % de webhooks processados com sucesso
- **Tempo de Resposta**: Latência de processamento
- **Duplicações**: Webhooks processados múltiplas vezes
- **Falhas**: Erros por gateway e tipo

### Logs para Análise
```javascript
// Sucesso
console.log('✅ Nova matrícula criada:', enrollment.id);

// Duplicação
console.log('✅ Pagamento já processado anteriormente:', orderId);

// Falha
console.error('❌ Pedido não encontrado:', { orderId, transactionId });
```

---

## 🔮 Funcionalidades Futuras (TODOs)

### Implementações Planejadas
```javascript
// 1. Email de Boas-vindas
await sendWelcomeEmail(user, course, enrollment);

// 2. Grupo WhatsApp
if (course.whatsapp_group_url) {
  await addToWhatsAppGroup(user, course);
}

// 3. Sistema de Gamificação
await createGamificationEntry(user.id, 'course_enrolled', course.id);
```

### Melhorias Técnicas
- **Assinatura de Webhooks**: Verificação de autenticidade
- **Rate Limiting**: Proteção contra spam
- **Fila de Processamento**: Para alta demanda
- **Webhooks Bidirecionais**: Notificações de volta para gateways

---

**Próxima documentação**: `routes/courses.js` - Sistema de cursos e aulas
