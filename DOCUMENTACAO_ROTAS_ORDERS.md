# 🛒 Documentação Completa - Rotas de Pedidos

## Visão Geral
O arquivo `routes/orders.js` contém todas as rotas para gerenciamento CRUD de pedidos, incluindo integração com cursos, matrícula automática, webhooks de pagamento e sistema de tracking. O sistema inclui funcionalidades avançadas para processamento de pagamentos e notificações.

## Middleware Utilizado
- **Autenticação**: `authenticateToken` para rotas protegidas
- **Segurança**: Filtro por produtos do vendedor logado
- **Validação**: Validação de dados obrigatórios e status

## Funcionalidades Especiais
- **Matrícula Automática em Cursos**: Quando pedido de curso é confirmado
- **Geração de Senhas Temporárias**: Para novos usuários de curso
- **Envio de Emails**: Credenciais de acesso e notificações
- **Webhooks**: Integração com provedores de pagamento

---

## 🛒 CRUD DE PEDIDOS

### GET /api/orders
**Descrição**: Lista todos os pedidos de produtos do usuário logado

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "orders": [
    {
      "id": "uuid-pedido-1",
      "customer_id": "uuid-cliente",
      "product_id": "uuid-produto",
      "status": "completed",
      "amount": 197.00,
      "payment_method": "credit_card",
      "customer_info": {
        "name": "João Silva",
        "email": "joao@email.com",
        "phone": "(11) 99999-9999"
      },
      "utm_params": {
        "source": "facebook",
        "medium": "cpc",
        "campaign": "lancamento"
      },
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2025-01-01T00:00:00Z",
      "product": {
        "id": "uuid-produto",
        "name": "Curso de JavaScript Avançado",
        "slug": "abc123def-curso-javascript-avancado",
        "price": 197.00,
        "image_url": "https://exemplo.com/imagem.jpg",
        "seller": {
          "id": "uuid-vendedor",
          "name": "Professor João",
          "email": "professor@email.com"
        }
      },
      "customer": {
        "id": "uuid-cliente",
        "name": "João Silva",
        "email": "joao@email.com"
      },
      "productName": "Curso de JavaScript Avançado",
      "productSlug": "abc123def-curso-javascript-avancado",
      "productImage": "https://exemplo.com/imagem.jpg",
      "customerName": "João Silva",
      "customerEmail": "joao@email.com"
    }
  ]
}
```

**Observações**:
- Lista apenas pedidos de produtos do usuário logado (filtro por `seller_id`)
- Inclui dados do produto, vendedor e cliente
- Ordenação por data de criação (mais recentes primeiro)
- Campos adicionais para facilitar exibição

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

### GET /api/orders/customer/:email
**Descrição**: Busca pedidos por email do cliente (para área de membros)

**Autenticação**: Obrigatória

**Parâmetros**:
- `email` (string): Email do cliente (URL encoded)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "orders": [
    {
      "id": "uuid-pedido",
      "status": "completed",
      "amount": 197.00,
      "payment_method": "pix",
      "created_at": "2024-01-01T00:00:00Z",
      "product": {
        "id": "uuid-produto",
        "name": "Curso de React",
        "slug": "def456ghi-curso-react",
        "price": 197.00,
        "image_url": "https://exemplo.com/react.jpg",
        "media_type": "digital"
      },
      "productName": "Curso de React",
      "productSlug": "def456ghi-curso-react",
      "productImage": "https://exemplo.com/react.jpg",
      "mediaType": "digital",
      "customerName": "João Silva",
      "customerEmail": "joao@email.com"
    }
  ],
  "count": 1
}
```

**Funcionalidades**:
- Busca por email no relacionamento `User` ou no campo `customer_info`
- Útil para área de membros e validação de acesso
- Inclui dados do produto para verificação de tipo de mídia

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

### GET /api/orders/:order_id
**Descrição**: Busca um pedido específico por ID

**Autenticação**: Não obrigatória (pública)

**Parâmetros**:
- `order_id` (string): UUID do pedido

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "order": {
    "id": "uuid-pedido",
    "customer_id": "uuid-cliente",
    "product_id": "uuid-produto",
    "status": "completed",
    "amount": 197.00,
    "payment_method": "credit_card",
    "customer_info": {
      "name": "João Silva",
      "email": "joao@email.com",
      "phone": "(11) 99999-9999"
    },
    "utm_params": {
      "source": "google",
      "medium": "organic"
    },
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z",
    "product": {
      "id": "uuid-produto",
      "name": "Curso de JavaScript Avançado",
      "description": "Aprenda JavaScript do básico ao avançado",
      "price": 197.00,
      "seller": {
        "id": "uuid-vendedor",
        "name": "Professor João",
        "email": "professor@email.com"
      }
    },
    "customer": {
      "id": "uuid-cliente",
      "name": "João Silva",
      "email": "joao@email.com"
    }
  }
}
```

**Erros Possíveis**:
- `404`: Pedido não encontrado
- `500`: Erro interno do servidor

---

### POST /api/orders
**Descrição**: Cria novo pedido

**Autenticação**: Não obrigatória (pública)

**Dados Esperados**:
```json
{
  "customerId": "uuid-cliente",
  "productId": "uuid-produto",
  "amount": 197.00,
  "status": "pending",
  "paymentMethod": "credit_card",
  "customerInfo": {
    "name": "João Silva",
    "email": "joao@email.com",
    "phone": "(11) 99999-9999",
    "document": "123.456.789-00"
  },
  "utmParams": {
    "source": "facebook",
    "medium": "cpc",
    "campaign": "lancamento",
    "content": "video_1"
  }
}
```

**Campos Obrigatórios**:
- `customerId`: UUID do cliente
- `productId`: UUID do produto
- `amount`: Valor do pedido (número)

**Campos Opcionais**:
- `status`: Status inicial (padrão: "pending")
- `paymentMethod`: Método de pagamento
- `customerInfo`: Dados extras do cliente
- `utmParams`: Parâmetros de UTM para tracking

**Validações**:
- Produto deve existir
- Cliente deve existir
- Amount deve ser numérico válido

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Pedido criado com sucesso",
  "order": {
    "id": "uuid-novo-pedido",
    "customer_id": "uuid-cliente",
    "product_id": "uuid-produto",
    "status": "pending",
    "amount": 197.00,
    "payment_method": "credit_card",
    "customer_info": {
      "name": "João Silva",
      "email": "joao@email.com"
    },
    "utm_params": {
      "source": "facebook",
      "medium": "cpc"
    },
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z",
    "product": {
      "id": "uuid-produto",
      "name": "Curso de JavaScript Avançado",
      "slug": "abc123def-curso-javascript-avancado",
      "price": 197.00
    },
    "customer": {
      "id": "uuid-cliente",
      "name": "João Silva",
      "email": "joao@email.com"
    }
  }
}
```

**Erros Possíveis**:
- `400`: Campos obrigatórios ausentes
- `404`: Produto ou cliente não encontrado
- `500`: Erro interno do servidor

---

### PUT /api/orders/:order_id
**Descrição**: Atualiza pedido existente

**Autenticação**: Não obrigatória (pública)

**Parâmetros**:
- `order_id` (string): UUID do pedido

**Dados Esperados**:
```json
{
  "status": "completed",
  "amount": 226.90,
  "paymentMethod": "pix",
  "customerInfo": {
    "name": "João Silva Santos",
    "email": "joao@email.com",
    "phone": "(11) 99999-9999"
  },
  "utmParams": {
    "source": "facebook",
    "medium": "cpc",
    "campaign": "lancamento"
  }
}
```

**Campos Atualizáveis**:
- `status`: Status do pedido
- `amount`: Valor do pedido
- `paymentMethod`: Método de pagamento
- `customerInfo`: Informações do cliente
- `utmParams`: Parâmetros UTM

**Funcionalidade Especial - Matrícula Automática**:
Quando status muda para "completed", o sistema:
1. Verifica se o produto é um curso (`category: 'curso'`)
2. Busca o curso associado ao produto
3. Cria matrícula automática no curso
4. Gera senha temporária se cliente não tiver
5. Envia email com credenciais de acesso

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Pedido atualizado com sucesso",
  "order": {
    "id": "uuid-pedido",
    "status": "completed",
    "amount": 226.90,
    // ... dados atualizados
  }
}
```

**Erros Possíveis**:
- `404`: Pedido não encontrado
- `500`: Erro interno do servidor

---

### DELETE /api/orders/:order_id
**Descrição**: Remove pedido permanentemente

**Autenticação**: Não obrigatória (pública)

**Parâmetros**:
- `order_id` (string): UUID do pedido

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Pedido excluído com sucesso"
}
```

**Observações**:
- Remove permanentemente o pedido
- Operação irreversível

**Erros Possíveis**:
- `404`: Pedido não encontrado
- `500`: Erro interno do servidor

---

## 📊 ESTATÍSTICAS DE PEDIDOS

### GET /api/orders/stats
**Descrição**: Estatísticas de pedidos do usuário logado

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "stats": {
    "totalOrders": 125,
    "totalRevenue": 24650.00,
    "pendingOrders": 8,
    "completedOrders": 117,
    "averageOrderValue": 197.20
  }
}
```

**Campos das Estatísticas**:
- `totalOrders`: Total de pedidos de todos os produtos do usuário
- `totalRevenue`: Receita total gerada
- `pendingOrders`: Pedidos pendentes
- `completedOrders`: Pedidos confirmados
- `averageOrderValue`: Valor médio por pedido

**Observações**:
- Calcula apenas produtos do usuário logado
- Se usuário não tem produtos, retorna zeros
- Receita baseada no campo `amount` dos pedidos

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 🔗 WEBHOOKS E INTEGRAÇÕES

### POST /api/orders/webhook/payment-status
**Descrição**: Webhook para atualização de status de pagamento

**Autenticação**: Não obrigatória (pública com validação)

**Dados Esperados**:
```json
{
  "order_id": "uuid-pedido",
  "status": "completed",
  "payment_id": "pay_123456789",
  "provider": "stripe",
  "metadata": {
    "transaction_id": "txn_abc123",
    "fee": 5.85,
    "net_amount": 191.15
  }
}
```

**Campos Obrigatórios**:
- `order_id`: UUID do pedido
- `status`: Novo status do pedido

**Campos Opcionais**:
- `payment_id`: ID do pagamento no provedor
- `provider`: Nome do provedor de pagamento
- `metadata`: Dados adicionais do pagamento

**Status Válidos**:
- `pending`: Aguardando pagamento
- `processing`: Processando pagamento  
- `completed`: Pagamento confirmado
- `failed`: Pagamento falhou
- `cancelled`: Pedido cancelado

**Funcionalidades Automáticas**:
- Atualiza status do pedido
- Dispara webhooks para vendedor (se status = completed)
- Registra dados do pagamento

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Status do pedido atualizado",
  "data": {
    "order_id": "uuid-pedido",
    "old_status": "pending",
    "new_status": "completed",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

**Erros Possíveis**:
- `400`: Dados obrigatórios ausentes ou status inválido
- `404`: Pedido não encontrado
- `500`: Erro interno do servidor

---

### GET /api/orders/public/status/:orderId
**Descrição**: Verifica status de pedido (público)

**Autenticação**: Não obrigatória (pública)

**Parâmetros**:
- `orderId` (string): ID público do pedido

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "order_id": "ORD123456789",
    "status": "completed",
    "status_label": "Pagamento confirmado",
    "total": 197.00,
    "product": {
      "name": "Curso de JavaScript Avançado",
      "image": "https://exemplo.com/imagem.jpg"
    },
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

**Labels de Status**:
- `pending`: "Aguardando pagamento"
- `processing`: "Processando pagamento"
- `completed`: "Pagamento confirmado"
- `failed`: "Pagamento falhou"
- `cancelled`: "Pedido cancelado"

**Uso**: Página de tracking de pedidos, emails de confirmação

**Erros Possíveis**:
- `404`: Pedido não encontrado
- `500`: Erro interno do servidor

---

### POST /api/orders/simulate-payment
**Descrição**: Simula pagamento para testes (apenas desenvolvimento)

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Dados Esperados**:
```json
{
  "order_id": "uuid-pedido",
  "should_succeed": true
}
```

**Campos**:
- `order_id`: UUID do pedido
- `should_succeed`: Se deve simular sucesso (padrão: true)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Pagamento simulado: Sucesso",
  "data": {
    "order_id": "uuid-pedido",
    "status": "completed",
    "payment_id": "sim_1704067200000",
    "simulated": true
  }
}
```

**Restrições**:
- Funciona apenas em `NODE_ENV !== 'production'`
- Útil para testes de integração

**Erros Possíveis**:
- `400`: Order ID obrigatório
- `403`: Não disponível em produção
- `404`: Pedido não encontrado
- `500`: Erro interno do servidor

---

## 🎓 SISTEMA DE MATRÍCULA AUTOMÁTICA

### Fluxo de Matrícula em Cursos
Quando um pedido de produto tipo "curso" é marcado como "completed":

1. **Verificação**: Confirma se produto é categoria "curso"
2. **Busca do Curso**: Localiza curso associado ao produto
3. **Verificação de Matrícula**: Checa se cliente já está matriculado
4. **Geração de Senha**: Cria senha temporária se necessário
5. **Criação da Matrícula**: Registra matrícula no sistema
6. **Envio de Email**: Envia credenciais de acesso

### Geração de Senha Segura
```javascript
// Características da senha gerada:
- 12 caracteres
- Pelo menos 1 maiúscula
- Pelo menos 1 minúscula  
- Pelo menos 1 número
- Pelo menos 1 símbolo especial
- Caracteres embaralhados aleatoriamente
```

### Dados da Matrícula
```json
{
  "id": "uuid-matricula",
  "course_id": "uuid-curso",
  "student_id": "uuid-cliente",
  "order_id": "uuid-pedido",
  "enrolled_at": "2025-01-01T00:00:00Z",
  "status": "active",
  "progress_percentage": 0,
  "access_data": {
    "enrolledVia": "purchase",
    "email": "joao@email.com",
    "tempPasswordGenerated": true,
    "purchaseDate": "2025-01-01T00:00:00Z",
    "productId": "uuid-produto",
    "productName": "Curso de JavaScript Avançado"
  }
}
```

### Tipos de Email Enviados
- **Com Senha Nova**: Email com credenciais completas de acesso
- **Usuário Existente**: Email de notificação de matrícula

---

## 🔧 Funções Auxiliares

### processCourseEnrollment()
- Processa matrícula automática quando pedido é confirmado
- Gera senha temporária se necessário
- Cria registro de matrícula
- Envia emails de acesso

### generateSecurePassword()
- Gera senhas seguras de 12 caracteres
- Inclui todos os tipos de caracteres
- Embaralha resultado aleatoriamente

---

## 🛡️ Segurança e Validações

### Autenticação
- Rotas protegidas requerem JWT válido
- Filtro por produtos do usuário logado
- Webhooks públicos com validação de dados

### Validações
- Campos obrigatórios verificados
- Status válidos para atualizações
- Existência de produto e cliente
- Unicidade de matrículas em cursos

### Logs e Auditoria
- Logs detalhados de todas as operações
- Rastreamento de erros
- Monitoramento de webhooks

---

## 📈 Campos de Tracking

### UTM Parameters
```json
{
  "source": "facebook",
  "medium": "cpc",
  "campaign": "lancamento",
  "content": "video_1",
  "term": "javascript"
}
```

### Customer Info
```json
{
  "name": "João Silva",
  "email": "joao@email.com", 
  "phone": "(11) 99999-9999",
  "document": "123.456.789-00",
  "address": {
    "street": "Rua das Flores, 123",
    "city": "São Paulo",
    "state": "SP",
    "zipCode": "01234-567"
  }
}
```

### Metadata de Pagamento
```json
{
  "transaction_id": "txn_abc123",
  "fee": 5.85,
  "net_amount": 191.15,
  "installments": 3,
  "card_brand": "visa",
  "card_last4": "1234"
}
```

---

## 🚀 Integrações Suportadas

### Provedores de Pagamento
- Stripe
- PayPal
- PagSeguro
- Mercado Pago
- PIX
- Boleto

### Notificações
- Email automático de confirmação
- Webhooks para vendedores
- Notificações push (se configurado)

### Analytics
- Tracking de conversões
- Métricas de pedidos
- Relatórios de vendas

---

**Próxima documentação**: `routes/analytics.js` - Sistema de analytics avançado
