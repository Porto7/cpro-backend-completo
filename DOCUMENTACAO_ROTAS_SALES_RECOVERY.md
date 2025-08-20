# Documentação API - Sistema de Sales Recovery (Carrinho Abandonado)

## Visão Geral
Sistema completo de recuperação de vendas abandonadas que permite rastrear carrinhos abandonados, enviar emails de recuperação automatizados e analisar estatísticas de conversão.

## Endpoints Disponíveis

### 1. Listar Carrinhos Abandonados
**GET** `/api/sales-recovery/abandoned-carts`

**Autenticação:** Requerida

**Descrição:** Lista os carrinhos abandonados dos produtos do usuário com paginação e filtros.

**Parâmetros de Query:**
- `page` (number, opcional): Número da página (padrão: 1)
- `limit` (number, opcional): Itens por página (padrão: 20)
- `recovered` (boolean, opcional): Filtrar por status de recuperação
- `days` (number, opcional): Período em dias (padrão: 30)

**Exemplo de Requisição:**
```
GET /api/sales-recovery/abandoned-carts?page=1&limit=10&recovered=false&days=7
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-cart",
      "customer_email": "cliente@email.com",
      "product_id": "uuid-produto",
      "product": {
        "id": "uuid-produto",
        "name": "Curso de Marketing Digital",
        "price": 497.00,
        "slug": "curso-marketing-digital"
      },
      "abandoned_at": "2024-01-15T10:30:00Z",
      "recovered": false,
      "recovered_at": null,
      "emails_sent": 1,
      "last_email_sent": "2024-01-15T12:00:00Z",
      "recovery_token": "recovery-token-123"
    }
  ],
  "pagination": {
    "current_page": 1,
    "total_pages": 3,
    "total_items": 25,
    "items_per_page": 10
  }
}
```

### 2. Registrar Carrinho Abandonado
**POST** `/api/sales-recovery/track`

**Autenticação:** Não requerida

**Descrição:** Registra um novo carrinho abandonado ou atualiza um existente das últimas 24 horas.

**Body da Requisição:**
```json
{
  "product_id": "uuid-produto",
  "customer_email": "cliente@email.com",
  "customer_data": {
    "name": "João Silva",
    "phone": "11999999999",
    "utm_source": "google",
    "utm_campaign": "marketing"
  },
  "session_id": "session-123"
}
```

**Parâmetros:**
- `product_id` (string, obrigatório): ID do produto
- `customer_email` (string, obrigatório): Email do cliente
- `customer_data` (object, opcional): Dados adicionais do cliente
- `session_id` (string, opcional): ID da sessão

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Carrinho abandonado registrado",
  "data": {
    "id": "uuid-cart",
    "recovery_token": "recovery-token-123"
  }
}
```

**Erros Possíveis:**
- `400` - Product ID e customer email obrigatórios
- `404` - Produto não encontrado

### 3. Enviar Email de Recuperação
**POST** `/api/sales-recovery/send-recovery/:cartId`

**Autenticação:** Requerida

**Descrição:** Envia email de recuperação para um carrinho abandonado com templates personalizados.

**Rate Limit:** 20 requests por 5 minutos por IP

**Parâmetros de Rota:**
- `cartId` (string): ID do carrinho abandonado

**Body da Requisição:**
```json
{
  "template": "urgent"
}
```

**Parâmetros:**
- `template` (string, opcional): Template do email (`default` ou `urgent`)

**Templates Disponíveis:**

#### Template "default"
- Assunto: "Você esqueceu algo! - [Nome do Produto]"
- Estilo: Amigável e informativo
- Call-to-action: "Finalizar Compra"

#### Template "urgent"
- Assunto: "⚠️ ÚLTIMAS HORAS - [Nome do Produto]"
- Estilo: Urgência e escassez
- Call-to-action: "FINALIZAR AGORA"

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Email de recuperação enviado com sucesso",
  "data": {
    "id": "uuid-cart",
    "emails_sent": 2,
    "template_used": "urgent"
  }
}
```

**Erros Possíveis:**
- `404` - Carrinho não encontrado ou sem permissão
- `400` - Carrinho já foi recuperado
- `400` - Limite de emails atingido (máximo 3)
- `429` - Rate limit excedido

### 4. Marcar Carrinho como Recuperado
**POST** `/api/sales-recovery/mark-recovered/:cartId`

**Autenticação:** Não requerida (usa recovery token)

**Descrição:** Marca um carrinho abandonado como recuperado quando o cliente finaliza a compra.

**Parâmetros de Rota:**
- `cartId` (string): ID do carrinho abandonado

**Body da Requisição:**
```json
{
  "recovery_token": "recovery-token-123",
  "order_id": "uuid-pedido"
}
```

**Parâmetros:**
- `recovery_token` (string, obrigatório): Token de recuperação
- `order_id` (string, opcional): ID do pedido finalizado

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Carrinho marcado como recuperado",
  "data": {
    "id": "uuid-cart",
    "recovered": true,
    "recovered_at": "2024-01-15T14:30:00Z"
  }
}
```

**Erros Possíveis:**
- `404` - Carrinho não encontrado ou token inválido

### 5. Estatísticas de Recuperação
**GET** `/api/sales-recovery/stats`

**Autenticação:** Requerida

**Descrição:** Retorna estatísticas detalhadas sobre recuperação de vendas abandonadas.

**Parâmetros de Query:**
- `days` (number, opcional): Período em dias para análise (padrão: 30)

**Exemplo de Requisição:**
```
GET /api/sales-recovery/stats?days=7
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": {
    "period_days": 7,
    "total_abandoned": 50,
    "total_recovered": 12,
    "recovery_rate": 24.00,
    "revenue_recovered": 5964.00,
    "emails_sent": 85,
    "avg_emails_per_cart": 1.70,
    "product_breakdown": [
      {
        "product_id": "uuid-produto-1",
        "abandoned": 25,
        "recovered": 8,
        "revenue": 3976.00
      },
      {
        "product_id": "uuid-produto-2",
        "abandoned": 25,
        "recovered": 4,
        "revenue": 1988.00
      }
    ],
    "summary": {
      "potential_revenue": 24850.00,
      "lost_revenue": 18886.00
    }
  }
}
```

**Métricas Explicadas:**
- `recovery_rate`: Percentual de carrinhos recuperados
- `revenue_recovered`: Receita total recuperada
- `avg_emails_per_cart`: Média de emails enviados por carrinho
- `potential_revenue`: Receita total possível (todos os carrinhos)
- `lost_revenue`: Receita perdida (carrinhos não recuperados)

### 6. Página de Recuperação via Token
**GET** `/api/sales-recovery/recovery/:token`

**Autenticação:** Não requerida

**Descrição:** Página de recuperação acessada via link do email. Redireciona para o checkout com dados pré-preenchidos.

**Parâmetros de Rota:**
- `token` (string): Token de recuperação

**Comportamento:**
- Valida o token de recuperação
- Verifica se não expirou (7 dias)
- Redireciona para checkout com dados pré-preenchidos

**Resposta de Sucesso (302 - Redirect):**
```
Location: https://frontend.com/checkout/produto-slug?recovery=token&email=cliente@email.com&prefill=true
```

**Erros Possíveis:**
- `404` - Token inválido ou expirado
- `400` - Token expirado (mais de 7 dias)

## Fluxo de Recuperação de Vendas

### 1. Rastreamento de Abandono
```javascript
// Cliente inicia checkout mas não finaliza
await fetch('/api/sales-recovery/track', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    product_id: 'uuid-produto',
    customer_email: 'cliente@email.com',
    customer_data: { name: 'João Silva' }
  })
});
```

### 2. Campanhas de Email
```javascript
// Enviar primeiro email (após 1 hora)
setTimeout(async () => {
  await fetch(`/api/sales-recovery/send-recovery/${cartId}`, {
    method: 'POST',
    headers: { 
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json' 
    },
    body: JSON.stringify({ template: 'default' })
  });
}, 3600000); // 1 hora

// Enviar email urgente (após 24 horas)
setTimeout(async () => {
  await fetch(`/api/sales-recovery/send-recovery/${cartId}`, {
    method: 'POST',
    headers: { 
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json' 
    },
    body: JSON.stringify({ template: 'urgent' })
  });
}, 86400000); // 24 horas
```

### 3. Recuperação via Link
```javascript
// Cliente clica no link do email
// URL: /api/sales-recovery/recovery/recovery-token-123
// Sistema redireciona para checkout com dados pré-preenchidos
```

### 4. Finalização da Recuperação
```javascript
// Ao finalizar a compra, marcar como recuperado
await fetch(`/api/sales-recovery/mark-recovered/${cartId}`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    recovery_token: 'recovery-token-123',
    order_id: 'uuid-pedido-finalizado'
  })
});
```

## Regras de Negócio

### Limite de Emails
- Máximo 3 emails por carrinho abandonado
- Rate limit: 20 requests por 5 minutos por IP

### Expiração de Token
- Tokens de recuperação expiram em 7 dias
- Links de email tornam-se inválidos após expiração

### Carrinho Duplicado
- Se existe carrinho das últimas 24h para mesmo produto+email, atualiza ao invés de criar novo
- Evita spam de carrinhos abandonados

### Templates de Email
- **Default**: Abordagem amigável para primeira tentativa
- **Urgent**: Abordagem de urgência para tentativas subsequentes

## Segurança

### Rate Limiting
```javascript
const recoveryLimiter = rateLimit({
  windowMs: 5 * 60 * 1000, // 5 minutos
  max: 20, // máximo 20 requests por minuto por IP
  message: {
    error: 'Muitas tentativas de recuperação. Tente novamente em 5 minutos.',
    type: 'recovery_rate_limit'
  }
});
```

### Validação de Permissões
- Apenas proprietários dos produtos podem acessar carrinhos abandonados
- Tokens de recuperação são únicos e expiram automaticamente

### Logs de Auditoria
- Todas as tentativas de recuperação são logadas
- Rastreamento completo do funil de recuperação

## Exemplos de Integração

### Integração Frontend (JavaScript)
```javascript
// Rastrear abandono no checkout
window.addEventListener('beforeunload', async () => {
  if (isCheckoutPage && hasProductData && !isPurchaseComplete) {
    await fetch('/api/sales-recovery/track', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        product_id: productId,
        customer_email: customerEmail,
        customer_data: customerData
      })
    });
  }
});
```

### Dashboard de Recuperação
```javascript
// Buscar estatísticas para dashboard
const getRecoveryStats = async (days = 30) => {
  const response = await fetch(`/api/sales-recovery/stats?days=${days}`, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  return await response.json();
};

// Buscar carrinhos abandonados
const getAbandonedCarts = async (page = 1) => {
  const response = await fetch(`/api/sales-recovery/abandoned-carts?page=${page}`, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  return await response.json();
};
```

### Automação de Campanhas
```javascript
// Sistema automatizado de emails
const automateRecoveryEmails = async () => {
  // Buscar carrinhos abandonados há 1 hora sem emails enviados
  const recentCarts = await getCartsAbandonedHoursAgo(1);
  
  for (const cart of recentCarts) {
    if (cart.emails_sent === 0) {
      await sendRecoveryEmail(cart.id, 'default');
    }
  }
  
  // Buscar carrinhos abandonados há 24 horas com apenas 1 email
  const olderCarts = await getCartsAbandonedHoursAgo(24);
  
  for (const cart of olderCarts) {
    if (cart.emails_sent === 1) {
      await sendRecoveryEmail(cart.id, 'urgent');
    }
  }
};

// Executar a cada hora
setInterval(automateRecoveryEmails, 3600000);
```

## Observações Técnicas

1. **Performance**: Consultas otimizadas com índices apropriados
2. **Escalabilidade**: Sistema preparado para alto volume de carrinhos
3. **Personalização**: Templates de email customizáveis
4. **Analytics**: Métricas detalhadas para otimização
5. **Flexibilidade**: Suporte a diferentes estratégias de recuperação
6. **Compliance**: Respeita limites de email e preferências do usuário
