# 📊 Documentação Completa - Rotas de Analytics

## Visão Geral
O arquivo `routes/analytics.js` contém todas as rotas para análise de dados, métricas de conversão, performance de produtos e dashboard de vendas. Todas as rotas são protegidas e filtradas pelo usuário logado para garantir privacidade dos dados.

## Middleware Utilizado
- **Autenticação**: `authenticateToken` obrigatório para todas as rotas
- **Segurança**: Filtro rigoroso por `seller_id` do usuário logado
- **Validação**: Validação de períodos e parâmetros de consulta

## Funcionalidades
- **Métricas de Funil**: Conversões em cada etapa do funil de vendas
- **Dashboard**: Resumo executivo com KPIs principais
- **Performance de Produtos**: Análise detalhada por produto
- **Resumo de Receita**: Análise financeira completa
- **Carrinhos Abandonados**: Tracking de oportunidades perdidas

---

## 📈 MÉTRICAS DE FUNIL DE VENDAS

### GET /api/analytics/funnel-metrics
**Descrição**: Métricas completas do funil de vendas do usuário

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Query Parameters**:
- `startDate` (opcional): Data de início (padrão: 30 dias atrás)
- `endDate` (opcional): Data de fim (padrão: hoje)
- `productId` (opcional): Filtrar por produto específico

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "metrics": {
      "productViews": 2850,
      "addedToCart": 142,
      "checkoutsStarted": 98,
      "completedPurchases": 67
    },
    "conversionRates": {
      "viewToCart": 4.98,
      "cartToCheckout": 69.01,
      "checkoutToSale": 68.37,
      "overall": 2.35
    },
    "period": {
      "startDate": "2024-12-01T00:00:00Z",
      "endDate": "2025-01-01T00:00:00Z"
    }
  }
}
```

**Métricas Calculadas**:
- `productViews`: Visualizações dos produtos (estimativa)
- `addedToCart`: Produtos adicionados ao carrinho
- `checkoutsStarted`: Checkouts iniciados
- `completedPurchases`: Compras finalizadas

**Taxas de Conversão**:
- `viewToCart`: % de visualizações que viraram carrinho
- `cartToCheckout`: % de carrinhos que iniciaram checkout
- `checkoutToSale`: % de checkouts que viraram venda
- `overall`: % geral de conversão (view → sale)

**Observações**:
- Dados filtrados apenas por produtos do usuário logado
- Se usuário não tem produtos, retorna zeros
- Estimativa de visualizações baseada em produtos criados

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 📊 DASHBOARD EXECUTIVO

### GET /api/analytics/dashboard
**Descrição**: Dashboard executivo com resumo de KPIs

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Query Parameters**:
- `period` (opcional): Período de análise
  - `7d`: 7 dias
  - `30d`: 30 dias (padrão)
  - `90d`: 90 dias
  - `1y`: 1 ano

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "summary": {
      "totalRevenue": 13245.50,
      "totalOrders": 67,
      "totalProducts": 8,
      "conversionRate": 68.37
    },
    "period": {
      "startDate": "2024-12-01T00:00:00Z",
      "endDate": "2025-01-01T00:00:00Z",
      "period": "30d"
    },
    "charts": {
      "dailySales": [
        {
          "date": "2024-12-25",
          "revenue": 1970.00,
          "orders": 10
        },
        {
          "date": "2024-12-26",
          "revenue": 2364.00,
          "orders": 12
        },
        {
          "date": "2024-12-27",
          "revenue": 985.00,
          "orders": 5
        }
      ]
    },
    "topProducts": [
      {
        "id": "uuid-produto-1",
        "name": "Curso de JavaScript Avançado",
        "price": 197.00,
        "imageUrl": "https://exemplo.com/js.jpg",
        "totalOrders": 25,
        "revenue": 4925.00
      },
      {
        "id": "uuid-produto-2",
        "name": "React do Zero ao Pro",
        "price": 247.00,
        "imageUrl": "https://exemplo.com/react.jpg",
        "totalOrders": 18,
        "revenue": 4446.00
      }
    ]
  }
}
```

**Campos do Resumo**:
- `totalRevenue`: Receita total no período
- `totalOrders`: Total de pedidos
- `totalProducts`: Total de produtos do usuário
- `conversionRate`: Taxa de conversão média

**Gráficos**:
- `dailySales`: Vendas diárias dos últimos 7 dias
  - `date`: Data da venda
  - `revenue`: Receita do dia
  - `orders`: Número de pedidos

**Top Produtos**:
- Listagem dos 5 produtos com maior receita
- Ordenados por receita total
- Inclui imagem e estatísticas

**Observações**:
- Dashboard personalizado por usuário
- Cálculos em tempo real
- Suporte a múltiplos períodos

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 🏆 PERFORMANCE DE PRODUTOS

### GET /api/analytics/product-performance
**Descrição**: Performance detalhada dos produtos do usuário

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Query Parameters**:
- `limit` (opcional): Número de produtos (padrão: 10)
- `period` (opcional): Período de análise (padrão: 30d)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-produto-1",
      "name": "Curso de JavaScript Avançado",
      "price": 197.00,
      "imageUrl": "https://exemplo.com/js.jpg",
      "totalOrders": 25,
      "revenue": 4925.00
    },
    {
      "id": "uuid-produto-2",
      "name": "React do Zero ao Pro",
      "price": 247.00,
      "imageUrl": "https://exemplo.com/react.jpg",
      "totalOrders": 18,
      "revenue": 4446.00
    },
    {
      "id": "uuid-produto-3",
      "name": "Node.js para Iniciantes",
      "price": 127.00,
      "imageUrl": "https://exemplo.com/node.jpg",
      "totalOrders": 12,
      "revenue": 1524.00
    }
  ]
}
```

**Campos por Produto**:
- `id`: UUID do produto
- `name`: Nome do produto
- `price`: Preço unitário
- `imageUrl`: URL da imagem
- `totalOrders`: Total de pedidos no período
- `revenue`: Receita total gerada

**Ordenação**: Por receita total (maior primeiro)

**Uso**: Identificar produtos de maior sucesso e oportunidades

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 💰 RESUMO DE RECEITA

### GET /api/analytics/revenue-summary
**Descrição**: Resumo financeiro detalhado do usuário

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Query Parameters**:
- `period` (opcional): Período de análise (7d, 30d, 90d, 1y)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "totalRevenue": 13245.50,
    "completedOrders": 67,
    "pendingRevenue": 1576.00,
    "cancelledRevenue": 788.00,
    "averageOrderValue": 197.68,
    "period": {
      "startDate": "2024-12-01T00:00:00Z",
      "endDate": "2025-01-01T00:00:00Z",
      "period": "30d"
    }
  }
}
```

**Campos Financeiros**:
- `totalRevenue`: Receita confirmada (pedidos completed)
- `completedOrders`: Número de pedidos confirmados
- `pendingRevenue`: Receita de pedidos pendentes
- `cancelledRevenue`: Receita perdida por cancelamentos
- `averageOrderValue`: Ticket médio (AOV)

**Cálculos**:
- Receita total = Soma de pedidos com status "completed"
- Receita pendente = Soma de pedidos "pending" + "processing"
- Ticket médio = Receita total ÷ Pedidos confirmados

**Uso**: Análise financeira e planejamento de crescimento

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 📦 ESTATÍSTICAS POR PRODUTO

### GET /api/analytics/products/:id/stats
**Descrição**: Estatísticas detalhadas de um produto específico

**Autenticação**: Obrigatória

**Parâmetros**:
- `id` (string): UUID do produto

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "views": 1245,
    "conversions": 25,
    "conversion_rate": 2.01,
    "revenue": 4925.00,
    "top_traffic_sources": [
      {
        "name": "facebook.com",
        "count": 456
      },
      {
        "name": "google.com",
        "count": 234
      },
      {
        "name": "Direto",
        "count": 198
      },
      {
        "name": "instagram.com",
        "count": 156
      },
      {
        "name": "youtube.com",
        "count": 89
      }
    ],
    "abandoned_carts": 12
  }
}
```

**Métricas do Produto**:
- `views`: Total de visualizações
- `conversions`: Conversões (pedidos completed)
- `conversion_rate`: Taxa de conversão (%)
- `revenue`: Receita total gerada
- `abandoned_carts`: Carrinhos abandonados

**Top Traffic Sources**:
- Lista das 5 principais fontes de tráfego
- Baseado no campo `referrer` dos analytics
- "Direto" para tráfego sem referrer

**Segurança**: Verifica se produto pertence ao usuário logado

**Erros Possíveis**:
- `404`: Produto não encontrado ou sem permissão
- `500`: Erro interno do servidor

---

## 📈 ESTATÍSTICAS GERAIS DO DASHBOARD

### GET /api/analytics/dashboard/stats
**Descrição**: Estatísticas gerais para dashboard principal

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "total_products": 8,
    "total_orders": 125,
    "total_revenue": 24650.00,
    "pending_orders": 8,
    "conversion_rate": 2.35,
    "monthly_growth": 15.3
  }
}
```

**Estatísticas Calculadas**:
- `total_products`: Total de produtos do usuário
- `total_orders`: Total de pedidos (todos os status)
- `total_revenue`: Receita total confirmada
- `pending_orders`: Pedidos pendentes
- `conversion_rate`: Taxa de conversão geral
- `monthly_growth`: Crescimento mensal (%)

**Crescimento Mensal**:
- Compara pedidos completados do mês atual vs anterior
- Percentual de crescimento ou declínio
- Útil para análise de tendências

**Uso**: Cards principais do dashboard administrativo

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 🛒 CARRINHOS ABANDONADOS

### GET /api/analytics/abandoned-carts
**Descrição**: Lista carrinhos abandonados com dados de recuperação

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Query Parameters**:
- `page` (opcional): Página (padrão: 1)
- `limit` (opcional): Itens por página (padrão: 20)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "abandoned_carts": [
      {
        "id": "uuid-carrinho-1",
        "session_id": "sess_abc123",
        "customer_email": "joao@email.com",
        "customer_data": {
          "name": "João Silva",
          "phone": "(11) 99999-9999"
        },
        "recovery_token": "rec_token_xyz789",
        "product": {
          "id": "uuid-produto-1",
          "name": "Curso de JavaScript Avançado",
          "slug": "abc123def-curso-javascript-avancado",
          "price": 197.00
        },
        "created_at": "2024-12-31T15:30:00Z",
        "time_since": 45
      },
      {
        "id": "uuid-carrinho-2",
        "session_id": "sess_def456",
        "customer_email": "maria@email.com",
        "customer_data": {
          "name": "Maria Santos",
          "phone": "(11) 88888-8888"
        },
        "recovery_token": "rec_token_abc456",
        "product": {
          "id": "uuid-produto-2",
          "name": "React do Zero ao Pro",
          "slug": "def456ghi-react-zero-pro",
          "price": 247.00
        },
        "created_at": "2024-12-31T14:15:00Z",
        "time_since": 120
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 12,
      "pages": 1
    }
  }
}
```

**Dados do Carrinho Abandonado**:
- `id`: UUID do carrinho abandonado
- `session_id`: ID da sessão do usuário
- `customer_email`: Email do cliente (se fornecido)
- `customer_data`: Dados extras do cliente
- `recovery_token`: Token para recuperação
- `product`: Dados do produto abandonado
- `created_at`: Data/hora do abandono
- `time_since`: Tempo desde abandono (em minutos)

**Paginação**:
- Suporte completo a paginação
- Ordenação por data (mais recentes primeiro)
- Apenas carrinhos não recuperados

**Uso**: Campanhas de recuperação de carrinho via email

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 🔄 Funcionalidades Avançadas

### Filtros Automáticos
- Todas as consultas filtram por `seller_id` do usuário logado
- Impossível ver dados de outros usuários
- Proteção total de privacidade

### Cálculos em Tempo Real
- Estatísticas calculadas dinamicamente
- Não há cache, dados sempre atualizados
- Performance otimizada com índices

### Suporte a Períodos Flexíveis
- Períodos predefinidos: 7d, 30d, 90d, 1y
- Datas customizadas via parâmetros
- Comparações mês a mês

### Tratamento de Dados Ausentes
- Retorna zeros quando usuário não tem dados
- Evita divisões por zero
- Respostas consistentes sempre

---

## 📊 Métricas de Negócio

### KPIs Principais
- **Receita Total**: Soma de todas as vendas confirmadas
- **Taxa de Conversão**: % de visitantes que compram
- **Ticket Médio (AOV)**: Valor médio por pedido
- **Crescimento Mensal**: % de crescimento vs mês anterior

### Métricas de Funil
- **Visualizações**: Tráfego nos produtos
- **Add to Cart**: Produtos adicionados ao carrinho
- **Checkout Started**: Checkouts iniciados
- **Purchase**: Compras finalizadas

### Análise de Tráfego
- **Top Sources**: Principais fontes de visitantes
- **Conversão por Source**: Performance de cada canal
- **Abandono**: Identificação de gargalos

---

## 🎯 Casos de Uso

### Para Vendedores
- Monitorar performance de produtos
- Identificar produtos de maior sucesso
- Acompanhar crescimento de vendas
- Otimizar funil de conversão

### Para Recuperação
- Listar carrinhos abandonados
- Criar campanhas de email marketing
- Analisar motivos de abandono
- Medir eficácia de recuperação

### Para Otimização
- Identificar fontes de tráfego
- Melhorar taxas de conversão
- Ajustar preços baseado em performance
- Planejar novos produtos

---

## 🚀 Próximas Melhorias

### Analytics Avançados
- Segmentação de clientes
- Análise de coorte
- Lifetime Value (LTV)
- Churn rate

### Dashboards Visuais
- Gráficos interativos
- Comparações período a período
- Alertas automáticos
- Relatórios exportáveis

### Integrações
- Google Analytics
- Facebook Insights
- Hotjar/Mixpanel
- Webhooks personalizados

---

**Próxima documentação**: `routes/email.js` - Sistema de emails e notificações
