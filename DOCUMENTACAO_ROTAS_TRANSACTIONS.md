# Documentação API - Sistema de Transações Financeiras

## Visão Geral
Sistema completo para visualização e análise de transações financeiras dos usuários, incluindo créditos, débitos, comissões, saques e relatórios detalhados.

## Endpoints Disponíveis

### 1. Listar Transações
**GET** `/api/transactions`

**Autenticação:** Requerida

**Descrição:** Lista todas as transações do usuário com filtros avançados, paginação e estatísticas.

**Parâmetros de Query:**
- `page` (number, opcional): Página (padrão: 1)
- `limit` (number, opcional): Itens por página (padrão: 20)
- `type` (string, opcional): Tipo de transação (`credit`, `debit`, `commission`, `withdrawal`, `refund`)
- `status` (string, opcional): Status (`pending`, `completed`, `failed`, `cancelled`, `paid`)
- `startDate` (string, opcional): Data de início (formato ISO 8601)
- `endDate` (string, opcional): Data de fim (formato ISO 8601)
- `search` (string, opcional): Termo de busca

**Exemplo de Requisição:**
```
GET /api/transactions?page=1&limit=10&type=credit&status=completed&startDate=2024-01-01&endDate=2024-01-31
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "transactions": [
    {
      "id": "uuid-transacao",
      "amount": 497.00,
      "type": "credit",
      "status": "completed",
      "description": "Venda do produto: Curso de Marketing Digital",
      "paymentMethod": "credit_card",
      "product": {
        "id": "uuid-produto",
        "name": "Curso de Marketing Digital",
        "slug": "curso-marketing-digital",
        "price": 497.00
      },
      "order": {
        "id": "uuid-pedido",
        "orderNumber": "ORD-2024-001",
        "total": 497.00,
        "customerEmail": "cliente@email.com"
      },
      "user": {
        "id": "uuid-usuario",
        "name": "João Silva",
        "email": "joao@email.com"
      },
      "metadata": {
        "payment_id": "payment-123",
        "commission_rate": 0.1
      },
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-01-15T10:35:00Z",
      "displayAmount": "+R$ 497.00",
      "typeLabel": "Crédito",
      "statusLabel": "Concluído"
    }
  ],
  "stats": {
    "totalTransactions": 45,
    "totalCredits": 12450.00,
    "totalDebits": 2500.00,
    "pendingAmount": 750.00
  },
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 45,
    "totalPages": 5,
    "hasNextPage": true,
    "hasPreviousPage": false
  },
  "filters": {
    "availableTypes": ["credit", "debit", "commission", "withdrawal", "refund"],
    "availableStatuses": ["pending", "completed", "failed", "cancelled", "paid"]
  }
}
```

**Tipos de Transação:**
- `credit`: Receita de vendas próprias
- `commission`: Comissões de afiliados
- `debit`: Débitos/descontos
- `withdrawal`: Solicitações de saque
- `refund`: Reembolsos processados

**Status de Transação:**
- `pending`: Aguardando processamento
- `completed`: Concluída com sucesso
- `paid`: Pagamento confirmado
- `failed`: Falha no processamento
- `cancelled`: Cancelada

### 2. Detalhes de Transação Específica
**GET** `/api/transactions/:id`

**Autenticação:** Requerida

**Descrição:** Retorna detalhes completos de uma transação específica com timeline.

**Parâmetros de Rota:**
- `id` (string): ID da transação

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "transaction": {
    "id": "uuid-transacao",
    "amount": 497.00,
    "type": "credit",
    "status": "completed",
    "description": "Venda do produto: Curso de Marketing Digital",
    "paymentMethod": "credit_card",
    "product": {
      "id": "uuid-produto",
      "name": "Curso de Marketing Digital",
      "slug": "curso-marketing-digital",
      "price": 497.00,
      "description": "Aprenda marketing digital do zero ao avançado"
    },
    "order": {
      "id": "uuid-pedido",
      "orderNumber": "ORD-2024-001",
      "total": 497.00,
      "customerEmail": "cliente@email.com",
      "customerName": "Maria Silva",
      "paymentMethod": "credit_card"
    },
    "metadata": {
      "payment_id": "payment-123",
      "transaction_id": "txn-456",
      "commission_rate": 0.1,
      "payment_gateway": "stripe"
    },
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:35:00Z",
    "timeline": [
      {
        "status": "created",
        "timestamp": "2024-01-15T10:30:00Z",
        "description": "Transação criada"
      },
      {
        "status": "completed",
        "timestamp": "2024-01-15T10:35:00Z",
        "description": "Transação concluída"
      }
    ]
  }
}
```

**Erros Possíveis:**
- `404` - Transação não encontrada ou sem permissão

### 3. Resumo Financeiro
**GET** `/api/transactions/summary`

**Autenticação:** Requerida

**Descrição:** Gera resumo estatístico das transações com comparação de períodos e tendências.

**Parâmetros de Query:**
- `period` (string, opcional): Período de análise (`7d`, `30d`, `90d`, `1y`) (padrão: `30d`)

**Exemplo de Requisição:**
```
GET /api/transactions/summary?period=30d
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "summary": {
    "period": "30d",
    "startDate": "2023-12-16T00:00:00Z",
    "endDate": "2024-01-15T23:59:59Z",
    "totalTransactions": 85,
    "totalRevenue": 24850.00,
    "totalCommissions": 3725.00,
    "totalWithdrawals": 5000.00,
    "pendingAmount": 1250.00,
    "byStatus": {
      "pending": 5,
      "completed": 75,
      "failed": 3,
      "cancelled": 2
    },
    "byType": {
      "credit": 45,
      "commission": 25,
      "withdrawal": 12,
      "refund": 3
    },
    "trends": {
      "revenue": {
        "current": 24850.00,
        "previous": 18500.00,
        "change": 34.32,
        "trend": "up"
      }
    }
  }
}
```

**Métricas Explicadas:**
- `totalRevenue`: Receita total (créditos + comissões confirmadas)
- `totalCommissions`: Total de comissões recebidas
- `totalWithdrawals`: Valor total sacado
- `pendingAmount`: Valor em transações pendentes
- `change`: Variação percentual em relação ao período anterior
- `trend`: Tendência (`up` para crescimento, `down` para queda)

## Análise de Dados

### Filtros Avançados
```javascript
// Buscar transações por múltiplos critérios
const params = new URLSearchParams({
  page: 1,
  limit: 20,
  type: 'credit',
  status: 'completed',
  startDate: '2024-01-01T00:00:00Z',
  endDate: '2024-01-31T23:59:59Z'
});

const response = await fetch(`/api/transactions?${params}`, {
  headers: { 'Authorization': `Bearer ${token}` }
});
```

### Estatísticas Automáticas
```javascript
// As estatísticas são calculadas automaticamente para cada consulta
{
  "stats": {
    "totalTransactions": 45,        // Total de transações no período
    "totalCredits": 12450.00,       // Soma dos créditos confirmados
    "totalDebits": 2500.00,         // Soma dos débitos
    "pendingAmount": 750.00         // Valor em transações pendentes
  }
}
```

### Timeline de Transação
```javascript
// Cada transação inclui uma timeline detalhada
{
  "timeline": [
    {
      "status": "created",
      "timestamp": "2024-01-15T10:30:00Z",
      "description": "Transação criada"
    },
    {
      "status": "completed", 
      "timestamp": "2024-01-15T10:35:00Z",
      "description": "Transação concluída"
    }
  ]
}
```

## Formatação para Interface

### Display de Valores
```javascript
// Valores são formatados automaticamente para exibição
{
  "amount": 497.00,                    // Valor numérico para cálculos
  "displayAmount": "+R$ 497.00",      // Formatação para interface
  "type": "credit",                    // Tipo interno
  "typeLabel": "Crédito",             // Label amigável
  "status": "completed",               // Status interno  
  "statusLabel": "Concluído"          // Label amigável
}
```

### Mapeamento de Labels
```javascript
const typeLabels = {
  'credit': 'Crédito',
  'debit': 'Débito',
  'commission': 'Comissão',
  'withdrawal': 'Saque',
  'refund': 'Reembolso'
};

const statusLabels = {
  'pending': 'Pendente',
  'completed': 'Concluído',
  'failed': 'Falhou',
  'cancelled': 'Cancelado',
  'paid': 'Pago'
};
```

## Relacionamentos de Dados

### Transação com Produto
```json
{
  "product": {
    "id": "uuid-produto",
    "name": "Curso de Marketing Digital",
    "slug": "curso-marketing-digital",
    "price": 497.00,
    "description": "Descrição completa do produto"
  }
}
```

### Transação com Pedido
```json
{
  "order": {
    "id": "uuid-pedido",
    "orderNumber": "ORD-2024-001",
    "total": 497.00,
    "customerEmail": "cliente@email.com",
    "customerName": "Maria Silva",
    "paymentMethod": "credit_card"
  }
}
```

### Transação com Usuário
```json
{
  "user": {
    "id": "uuid-usuario",
    "name": "João Silva",
    "email": "joao@email.com"
  }
}
```

## Exemplos de Uso

### Dashboard Financeiro
```javascript
// Carregar resumo para dashboard
const getSummary = async (period = '30d') => {
  const response = await fetch(`/api/transactions/summary?period=${period}`, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  return await response.json();
};

// Exibir estatísticas principais
const summary = await getSummary('30d');
console.log(`Receita: R$ ${summary.summary.totalRevenue.toFixed(2)}`);
console.log(`Crescimento: ${summary.summary.trends.revenue.change.toFixed(1)}%`);
```

### Lista de Transações com Filtros
```javascript
const getTransactions = async (filters = {}) => {
  const params = new URLSearchParams({
    page: filters.page || 1,
    limit: filters.limit || 20,
    ...(filters.type && { type: filters.type }),
    ...(filters.status && { status: filters.status }),
    ...(filters.startDate && { startDate: filters.startDate }),
    ...(filters.endDate && { endDate: filters.endDate })
  });

  const response = await fetch(`/api/transactions?${params}`, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  
  return await response.json();
};

// Buscar apenas créditos dos últimos 7 dias
const recentCredits = await getTransactions({
  type: 'credit',
  status: 'completed',
  startDate: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString()
});
```

### Detalhes de Transação
```javascript
const getTransactionDetails = async (transactionId) => {
  const response = await fetch(`/api/transactions/${transactionId}`, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  
  if (!response.ok) {
    throw new Error('Transação não encontrada');
  }
  
  return await response.json();
};

// Exibir timeline da transação
const details = await getTransactionDetails('uuid-transacao');
details.transaction.timeline.forEach(event => {
  console.log(`${event.timestamp}: ${event.description}`);
});
```

### Relatório de Comissões
```javascript
// Relatório específico de comissões
const getCommissionsReport = async (period) => {
  const transactions = await getTransactions({
    type: 'commission',
    status: 'completed',
    limit: 100 // Ajustar conforme necessário
  });
  
  return {
    totalCommissions: transactions.stats.totalCredits,
    transactionCount: transactions.transactions.length,
    averageCommission: transactions.stats.totalCredits / transactions.transactions.length,
    topProducts: transactions.transactions
      .reduce((acc, t) => {
        if (t.product) {
          acc[t.product.id] = (acc[t.product.id] || 0) + t.amount;
        }
        return acc;
      }, {})
  };
};
```

### Análise de Tendências
```javascript
// Comparar múltiplos períodos
const compareRevenue = async () => {
  const [week, month, quarter] = await Promise.all([
    getSummary('7d'),
    getSummary('30d'), 
    getSummary('90d')
  ]);
  
  return {
    week: week.summary.totalRevenue,
    month: month.summary.totalRevenue,
    quarter: quarter.summary.totalRevenue,
    weeklyGrowth: week.summary.trends.revenue.change,
    monthlyGrowth: month.summary.trends.revenue.change,
    quarterlyGrowth: quarter.summary.trends.revenue.change
  };
};
```

## Segurança e Performance

### Controle de Acesso
- Usuários só veem suas próprias transações
- Verificação de propriedade em todas as consultas
- Logs de auditoria para operações sensíveis

### Otimizações
- Consultas com includes otimizados
- Paginação para listas grandes
- Índices apropriados no banco de dados
- Cache de estatísticas quando apropriado

### Rate Limiting
- Limites aplicados para evitar sobrecarga
- Consultas complexas com throttling
- Proteção contra consultas abusivas

## Observações Técnicas

1. **Compatibilidade**: Endpoint `/api/transactions` é alias de `/api/wallet/transactions` para compatibilidade
2. **Precisão**: Cálculos monetários usando parseFloat com formatação adequada
3. **Filtros**: Sistema flexível de filtros por data, tipo, status
4. **Tendências**: Comparação automática com período anterior
5. **Relacionamentos**: Joins otimizados com modelos relacionados
6. **Paginação**: Sistema robusto com metadados completos
