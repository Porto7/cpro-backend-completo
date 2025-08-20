# Documentação API - Gerenciamento de Planos de Usuário

## Visão Geral
Sistema completo para gerenciamento de planos de assinatura, incluindo upgrade, downgrade, cancelamento, reativação e controle de uso por recursos.

## Endpoints Disponíveis

### 1. Listar Planos Disponíveis
**GET** `/api/user-plans/available`

**Autenticação:** Não requerida

**Descrição:** Lista todos os planos disponíveis para assinatura.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "plan-basic",
      "name": "Plano Básico",
      "description": "Ideal para quem está começando",
      "price": 29.90,
      "billing_cycle": "monthly",
      "features": [
        "Até 10 produtos",
        "Checkout personalizado",
        "Suporte por email",
        "Analytics básico"
      ],
      "limits": {
        "products": 10,
        "transactions_per_month": 500,
        "storage_gb": 5,
        "custom_domain": true,
        "advanced_analytics": false,
        "priority_support": false
      },
      "popular": false,
      "created_at": "2024-01-01T00:00:00Z"
    },
    {
      "id": "plan-pro",
      "name": "Plano Pro",
      "description": "Para profissionais e pequenas empresas",
      "price": 79.90,
      "billing_cycle": "monthly",
      "features": [
        "Produtos ilimitados",
        "Checkout avançado",
        "Suporte prioritário",
        "Analytics avançado",
        "Domínio personalizado",
        "Integrações avançadas"
      ],
      "limits": {
        "products": -1,
        "transactions_per_month": 5000,
        "storage_gb": 50,
        "custom_domain": true,
        "advanced_analytics": true,
        "priority_support": true
      },
      "popular": true,
      "created_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

**Características dos Planos:**
- `price`: Valor em reais
- `billing_cycle`: `monthly` ou `annual`
- `features`: Lista de funcionalidades incluídas
- `limits`: Limites quantitativos (valor -1 = ilimitado)
- `popular`: Se é o plano mais popular (destaque)

### 2. Plano Atual do Usuário
**GET** `/api/user-plans/current`

**Autenticação:** Requerida

**Descrição:** Retorna o plano atual do usuário com informações de uso e limites.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": {
    "plan": {
      "id": "plan-pro",
      "name": "Plano Pro",
      "features": [
        "Produtos ilimitados",
        "Checkout avançado",
        "Suporte prioritário",
        "Analytics avançado"
      ],
      "limits": {
        "products": -1,
        "transactions_per_month": 5000,
        "storage_gb": 50,
        "custom_domain": true,
        "advanced_analytics": true,
        "priority_support": true
      }
    },
    "usage": {
      "products": {
        "used": 25,
        "limit": -1
      },
      "transactions_this_month": {
        "used": 340,
        "limit": 5000
      },
      "storage_gb": {
        "used": 2.5,
        "limit": 50
      },
      "period": {
        "start": "2024-01-01T00:00:00Z",
        "end": "2024-01-31T23:59:59Z"
      }
    },
    "expires_at": "2024-02-15T10:30:00Z",
    "status": "active",
    "is_trial": false,
    "subscription": {
      "id": "sub_123456789",
      "payment_method": "credit_card",
      "auto_renewal": true,
      "next_billing": "2024-02-15T10:30:00Z"
    }
  }
}
```

**Resposta Usuário Sem Plano (200):**
```json
{
  "success": true,
  "data": {
    "plan": {
      "id": "free",
      "name": "Plano Gratuito",
      "features": [
        "Até 3 produtos",
        "Checkout básico",
        "Suporte por email"
      ],
      "limits": {
        "products": 3,
        "transactions_per_month": 100,
        "storage_gb": 1,
        "custom_domain": false,
        "advanced_analytics": false,
        "priority_support": false
      }
    },
    "usage": {
      "products": {
        "used": 2,
        "limit": 3
      },
      "transactions_this_month": {
        "used": 15,
        "limit": 100
      },
      "storage_gb": {
        "used": 0.2,
        "limit": 1
      }
    },
    "expires_at": null,
    "status": "free",
    "is_trial": false
  }
}
```

### 3. Fazer Upgrade de Plano
**POST** `/api/user-plans/upgrade`

**Autenticação:** Requerida

**Rate Limit:** 30 requests por 15 minutos

**Descrição:** Inicia processo de upgrade para um plano superior.

**Body da Requisição:**
```json
{
  "plan_id": "plan-pro",
  "billing_cycle": "monthly",
  "payment_method": "credit_card"
}
```

**Parâmetros:**
- `plan_id` (string, obrigatório): ID do plano de destino
- `billing_cycle` (string, opcional): `monthly` ou `annual` (padrão: `monthly`)
- `payment_method` (string, opcional): Método de pagamento preferido

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Upgrade iniciado com sucesso",
  "data": {
    "user_plan_id": "user-plan-123",
    "plan": {
      "id": "plan-pro",
      "name": "Plano Pro",
      "price": 79.90,
      "billing_cycle": "monthly"
    },
    "payment": {
      "payment_url": "https://checkout.pro/payment/checkout/user-plan-123",
      "payment_id": "pay_1704963600000",
      "amount": 79.90,
      "currency": "BRL"
    },
    "status": "pending_payment"
  }
}
```

**Fluxo de Upgrade:**
1. Validação do plano e usuário
2. Cancelamento do plano atual (se existir)
3. Criação do novo plano com status `pending`
4. Geração de link de pagamento
5. Ativação após confirmação do pagamento

**Erros Possíveis:**
- `400` - Plan ID obrigatório
- `404` - Plano não encontrado ou inativo
- `429` - Rate limit excedido

### 4. Cancelar Plano
**POST** `/api/user-plans/cancel`

**Autenticação:** Requerida

**Descrição:** Cancela o plano atual, com opção de cancelamento imediato ou no final do período.

**Body da Requisição:**
```json
{
  "reason": "Não uso mais o serviço",
  "immediate": false
}
```

**Parâmetros:**
- `reason` (string, opcional): Motivo do cancelamento
- `immediate` (boolean, opcional): Se deve cancelar imediatamente (padrão: false)

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Plano será cancelado no final do período",
  "data": {
    "status": "active",
    "expires_at": "2024-02-15T10:30:00Z",
    "will_cancel_at": "2024-02-15T10:30:00Z"
  }
}
```

**Resposta Cancelamento Imediato (200):**
```json
{
  "success": true,
  "message": "Plano cancelado imediatamente",
  "data": {
    "status": "cancelled",
    "expires_at": "2024-02-15T10:30:00Z",
    "will_cancel_at": "2024-01-15T14:30:00Z"
  }
}
```

**Tipos de Cancelamento:**
- **Normal**: Mantém acesso até o final do período pago
- **Imediato**: Remove acesso instantaneamente

### 5. Reativar Plano
**POST** `/api/user-plans/reactivate`

**Autenticação:** Requerida

**Descrição:** Reativa um plano cancelado que ainda não expirou.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Plano reativado com sucesso",
  "data": {
    "status": "active",
    "expires_at": "2024-02-15T10:30:00Z",
    "auto_renewal": true
  }
}
```

**Erros Possíveis:**
- `404` - Nenhum plano cancelado encontrado
- `400` - Plano já expirado, não pode ser reativado

### 6. Uso Detalhado
**GET** `/api/user-plans/usage`

**Autenticação:** Requerida

**Descrição:** Retorna informações detalhadas sobre o uso atual dos recursos.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": {
    "products": {
      "used": 25,
      "limit": -1
    },
    "transactions_this_month": {
      "used": 340,
      "limit": 5000
    },
    "storage_gb": {
      "used": 2.5,
      "limit": 50
    },
    "period": {
      "start": "2024-01-01T00:00:00Z",
      "end": "2024-01-31T23:59:59Z"
    }
  }
}
```

**Métricas Calculadas:**
- `products.used`: Número total de produtos criados
- `transactions_this_month.used`: Transações no mês corrente
- `storage_gb.used`: Estimativa de armazenamento usado
- `period`: Período de cálculo (mês corrente)

### 7. Histórico de Planos
**GET** `/api/user-plans/history`

**Autenticação:** Requerida

**Descrição:** Histórico completo de todos os planos do usuário.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "user-plan-current",
      "plan": {
        "id": "plan-pro",
        "name": "Plano Pro",
        "price": 79.90
      },
      "status": "active",
      "billing_cycle": "monthly",
      "created_at": "2024-01-15T10:30:00Z",
      "expires_at": "2024-02-15T10:30:00Z",
      "cancelled_at": null,
      "is_trial": false
    },
    {
      "id": "user-plan-previous",
      "plan": {
        "id": "plan-basic",
        "name": "Plano Básico",
        "price": 29.90
      },
      "status": "cancelled",
      "billing_cycle": "monthly",
      "created_at": "2023-12-01T10:30:00Z",
      "expires_at": "2024-01-15T10:30:00Z",
      "cancelled_at": "2024-01-15T10:30:00Z",
      "is_trial": false
    }
  ]
}
```

## Status de Planos

### Estados Possíveis
- `active`: Plano ativo e funcional
- `pending`: Aguardando confirmação de pagamento
- `cancelled`: Cancelado pelo usuário
- `expired`: Expirado por falta de pagamento
- `suspended`: Suspenso por violação de termos
- `free`: Usuário no plano gratuito

### Transições de Estado
```javascript
// Fluxo normal
free → pending → active → expired/cancelled

// Upgrade/Downgrade
active → pending → active (novo plano)

// Cancelamento
active → cancelled → expired

// Reativação
cancelled → active (se não expirou)
```

## Controle de Limites

### Implementação de Verificação
```javascript
const checkLimit = async (userId, resource, amount = 1) => {
  const userPlan = await getCurrentUserPlan(userId);
  const usage = await calculateUserUsage(userId);
  
  const limit = userPlan.plan.limits[resource];
  const used = usage[resource].used;
  
  // -1 = ilimitado
  if (limit === -1) return true;
  
  return (used + amount) <= limit;
};

// Exemplo de uso
const canCreateProduct = await checkLimit(userId, 'products', 1);
if (!canCreateProduct) {
  throw new Error('Limite de produtos atingido');
}
```

### Recursos Controlados
- **Produtos**: Número máximo de produtos
- **Transações**: Transações por mês
- **Armazenamento**: Espaço em GB
- **Domínio Personalizado**: Booleano
- **Analytics Avançado**: Booleano
- **Suporte Prioritário**: Booleano

## Integração com Pagamentos

### Fluxo de Pagamento
```javascript
// 1. Usuário solicita upgrade
POST /api/user-plans/upgrade
{
  "plan_id": "plan-pro",
  "billing_cycle": "monthly"
}

// 2. Sistema cria UserPlan com status "pending"
// 3. Retorna URL de pagamento

// 4. Usuário completa pagamento
// 5. Webhook confirma pagamento

// 6. Sistema atualiza status para "active"
UserPlan.update({
  status: 'active',
  subscription_id: webhookData.subscription_id
});
```

### Webhooks de Pagamento
```javascript
// Webhook para ativar plano após pagamento
app.post('/webhook/payment-confirmed', async (req, res) => {
  const { user_plan_id, subscription_id } = req.body;
  
  await UserPlan.update({
    status: 'active',
    subscription_id,
    activated_at: new Date()
  }, {
    where: { id: user_plan_id }
  });
});
```

## Exemplos de Uso

### Verificar Plano do Usuário
```javascript
const getUserPlan = async () => {
  const response = await fetch('/api/user-plans/current', {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  const data = await response.json();
  
  if (data.data.status === 'free') {
    console.log('Usuário no plano gratuito');
  } else {
    console.log(`Plano: ${data.data.plan.name}`);
    console.log(`Expira em: ${data.data.expires_at}`);
  }
};
```

### Upgrade de Plano
```javascript
const upgradePlan = async (planId) => {
  const response = await fetch('/api/user-plans/upgrade', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      plan_id: planId,
      billing_cycle: 'monthly'
    })
  });
  
  const result = await response.json();
  
  if (result.success) {
    // Redirecionar para página de pagamento
    window.location.href = result.data.payment.payment_url;
  }
};
```

### Dashboard de Uso
```javascript
const createUsageDashboard = async () => {
  const [planData, usage] = await Promise.all([
    fetch('/api/user-plans/current').then(r => r.json()),
    fetch('/api/user-plans/usage').then(r => r.json())
  ]);
  
  const plan = planData.data.plan;
  const currentUsage = usage.data;
  
  // Calcular percentuais de uso
  Object.keys(currentUsage).forEach(resource => {
    const used = currentUsage[resource].used;
    const limit = plan.limits[resource];
    
    if (limit > 0) {
      const percentage = (used / limit) * 100;
      console.log(`${resource}: ${percentage.toFixed(1)}% usado`);
    } else if (limit === -1) {
      console.log(`${resource}: ${used} (ilimitado)`);
    }
  });
};
```

### Controle de Acesso por Funcionalidade
```javascript
const checkFeatureAccess = (plan, feature) => {
  return plan.limits[feature] === true || plan.limits[feature] === -1;
};

// Exemplo de uso
const userPlan = await getCurrentUserPlan();

if (checkFeatureAccess(userPlan.plan, 'advanced_analytics')) {
  // Mostrar analytics avançado
  renderAdvancedAnalytics();
} else {
  // Mostrar upgrade prompt
  showUpgradePrompt('analytics');
}
```

## Considerações Técnicas

### Rate Limiting
- Limite de 30 requests por 15 minutos para operações de plano
- Proteção contra abuso de tentativas de upgrade

### Segurança
- Verificação rigorosa de propriedade do plano
- Validação de estados válidos para transições
- Logs de auditoria para todas as mudanças

### Performance
- Cache de informações de plano por sessão
- Consultas otimizadas com joins apropriados
- Índices adequados para consultas frequentes

### Escalabilidade
- Sistema preparado para múltiplos gateways de pagamento
- Arquitetura extensível para novos tipos de plano
- Suporte a promoções e descontos

## Observações Importantes

1. **Estados Temporários**: Planos em estado `pending` devem ser verificados periodicamente
2. **Graceful Degradation**: Usuários mantêm acesso durante período de carência
3. **Billing Cycles**: Sistema suporta mensalidade e anuidade
4. **Trials**: Estrutura preparada para períodos de teste
5. **Compliance**: Logs detalhados para auditoria e compliance
6. **Internacionalização**: Estrutura preparada para múltiplas moedas
