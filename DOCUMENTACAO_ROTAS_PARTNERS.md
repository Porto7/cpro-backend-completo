# Documentação - Sistema de Parceiros (routes/partners.js)

## Visão Geral
Sistema completo para gerenciamento de co-produtores e afiliados. Permite criação de parcerias, gestão de comissões, rastreamento de vendas e pagamentos automáticos aos parceiros.

## Características Principais
- **Co-produtores e Afiliados**: Dois tipos de parceria distintos
- **Comissões Automáticas**: Cálculo e processamento automático de comissões
- **Códigos Únicos**: Sistema de códigos para rastreamento de vendas
- **Multi-produto**: Parceiros podem ter produtos diferentes
- **Gestão Financeira**: Aprovação e pagamento de comissões
- **Estatísticas Avançadas**: Analytics completas de performance
- **Auditoria Completa**: Logs detalhados de todas as operações

## Tipos de Parceiros

| Tipo | Descrição | Uso Comum |
|------|-----------|-----------|
| `co_producer` | Co-produtor com participação no produto | Especialistas que ajudam na criação |
| `affiliate` | Afiliado que promove o produto | Influenciadores e promotores |

## Status de Parcerias

| Status | Descrição |
|--------|-----------|
| `active` | Parceria ativa e funcionando |
| `inactive` | Parceria pausada temporariamente |
| `suspended` | Parceria suspensa por violações |
| `cancelled` | Parceria cancelada definitivamente |

## Rotas Implementadas

### 1. Gestão de Parcerias

#### `POST /api/partners`
**Descrição**: Criar nova parceria entre usuário e produto
**Acesso**: Autenticado

```javascript
// Payload de exemplo
{
  "productId": "product-123",
  "partnerId": "user-456",
  "partnerType": "affiliate", // ou "co_producer"
  "commissionPercentage": 15.5,
  "settings": {
    "autoApprove": true,
    "paymentMethod": "pix",
    "minPayoutAmount": 50.00
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Parceiro adicionado com sucesso",
  "partnership": {
    "id": "partnership-789",
    "product_id": "product-123",
    "partner_id": "user-456",
    "partner_type": "affiliate",
    "commission_percentage": 15.5,
    "partner_code": "AF8B2C1D",
    "status": "active",
    "settings": {
      "autoApprove": true,
      "paymentMethod": "pix",
      "minPayoutAmount": 50.00
    },
    "created_at": "2024-01-01T12:00:00Z",
    "product": {
      "id": "product-123",
      "name": "Curso de JavaScript",
      "slug": "curso-javascript"
    },
    "partner": {
      "id": "user-456",
      "name": "João Afiliado",
      "email": "joao@afiliado.com"
    }
  }
}
```

**Response Error**:
```json
{
  "error": "Já existe uma parceria entre este produto e usuário",
  "type": "conflict_error"
}
```

#### `GET /api/partners/product/:productId`
**Descrição**: Listar parceiros de um produto específico
**Acesso**: Público

**Query Parameters**:
- `status`: Filtrar por status (active, inactive, suspended, cancelled)
- `partnerType`: Filtrar por tipo (co_producer, affiliate)

**Response Success**:
```json
{
  "success": true,
  "partners": [
    {
      "id": "partnership-789",
      "partner_type": "affiliate",
      "commission_percentage": 15.5,
      "partner_code": "AF8B2C1D",
      "status": "active",
      "created_at": "2024-01-01T12:00:00Z",
      "partner": {
        "id": "user-456",
        "name": "João Afiliado",
        "email": "joao@afiliado.com"
      },
      "approver": null
    },
    {
      "id": "partnership-790",
      "partner_type": "co_producer",
      "commission_percentage": 30.0,
      "partner_code": "CP9D3E2F",
      "status": "active",
      "created_at": "2024-01-01T10:00:00Z",
      "partner": {
        "id": "user-789",
        "name": "Maria Co-produtora",
        "email": "maria@especialista.com"
      },
      "approver": {
        "id": "user-001",
        "name": "Admin Sistema",
        "email": "admin@sistema.com"
      }
    }
  ],
  "total": 2,
  "product": {
    "id": "product-123",
    "name": "Curso de JavaScript",
    "slug": "curso-javascript"
  }
}
```

#### `GET /api/partners/user/:userId`
**Descrição**: Listar parcerias de um usuário específico
**Acesso**: Público

**Query Parameters**:
- `status`: Filtrar por status
- `partnerType`: Filtrar por tipo

**Response Success**:
```json
{
  "success": true,
  "partnerships": [
    {
      "id": "partnership-789",
      "partner_type": "affiliate",
      "commission_percentage": 15.5,
      "partner_code": "AF8B2C1D",
      "status": "active",
      "created_at": "2024-01-01T12:00:00Z",
      "product": {
        "id": "product-123",
        "name": "Curso de JavaScript",
        "slug": "curso-javascript",
        "price": 197.00
      },
      "approver": null
    }
  ],
  "total": 1,
  "user": {
    "id": "user-456",
    "name": "João Afiliado",
    "email": "joao@afiliado.com"
  }
}
```

#### `PUT /api/partners/:id`
**Descrição**: Atualizar dados de uma parceria
**Acesso**: Autenticado

```javascript
// Payload de exemplo
{
  "commission_percentage": 20.0,
  "status": "active",
  "settings": {
    "autoApprove": false,
    "paymentMethod": "bank_transfer",
    "minPayoutAmount": 100.00
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Parceria atualizada com sucesso",
  "partnership": {
    "id": "partnership-789",
    "commission_percentage": 20.0,
    "status": "active",
    "settings": {
      "autoApprove": false,
      "paymentMethod": "bank_transfer",
      "minPayoutAmount": 100.00
    },
    "updated_at": "2024-01-01T13:00:00Z",
    "product": {
      "id": "product-123",
      "name": "Curso de JavaScript",
      "slug": "curso-javascript"
    },
    "partner": {
      "id": "user-456",
      "name": "João Afiliado",
      "email": "joao@afiliado.com"
    }
  }
}
```

#### `DELETE /api/partners/:id`
**Descrição**: Remover parceria (apenas se não houver comissões pendentes)
**Acesso**: Autenticado

**Response Success**:
```json
{
  "success": true,
  "message": "Parceria removida com sucesso"
}
```

**Response Error**:
```json
{
  "error": "Não é possível remover parceria com comissões pendentes",
  "type": "validation_error",
  "pendingCommissions": 3
}
```

### 2. Gestão de Códigos

#### `GET /api/partners/code/:partnerCode`
**Descrição**: Buscar parceria pelo código único do parceiro
**Acesso**: Público

**Response Success**:
```json
{
  "success": true,
  "partnership": {
    "id": "partnership-789",
    "partner_type": "affiliate",
    "commission_percentage": 15.5,
    "partner_code": "AF8B2C1D",
    "status": "active",
    "product": {
      "id": "product-123",
      "name": "Curso de JavaScript",
      "slug": "curso-javascript",
      "price": 197.00
    },
    "partner": {
      "id": "user-456",
      "name": "João Afiliado",
      "email": "joao@afiliado.com"
    }
  }
}
```

#### `POST /api/partners/validate-code`
**Descrição**: Validar código de parceiro para um produto específico
**Acesso**: Público

```javascript
// Payload de exemplo
{
  "partnerCode": "AF8B2C1D",
  "productId": "product-123"
}
```

**Response Success**:
```json
{
  "success": true,
  "valid": true,
  "partnership": {
    "id": "partnership-789",
    "partner_type": "affiliate",
    "commission_percentage": 15.5,
    "partner_code": "AF8B2C1D",
    "status": "active",
    "partner": {
      "id": "user-456",
      "name": "João Afiliado"
    }
  }
}
```

### 3. Gestão de Comissões

#### `GET /api/partners/:id/commissions`
**Descrição**: Listar comissões de uma parceria específica
**Acesso**: Público

**Query Parameters**:
- `status`: Filtrar por status (pending, approved, paid, cancelled)
- `startDate`: Data de início (YYYY-MM-DD)
- `endDate`: Data final (YYYY-MM-DD)

**Response Success**:
```json
{
  "success": true,
  "commissions": [
    {
      "id": "commission-123",
      "commission_amount": 29.55,
      "commission_percentage": 15.0,
      "order_amount": 197.00,
      "status": "approved",
      "created_at": "2024-01-01T12:00:00Z",
      "approved_at": "2024-01-01T14:00:00Z",
      "paid_at": null,
      "order": {
        "id": "order-456",
        "status": "completed",
        "amount": 197.00,
        "customer_info": {
          "name": "Cliente Teste",
          "email": "cliente@teste.com"
        },
        "product": {
          "id": "product-123",
          "name": "Curso de JavaScript",
          "slug": "curso-javascript"
        }
      }
    }
  ],
  "stats": {
    "total": 1,
    "totalAmount": 29.55,
    "pending": 0,
    "approved": 1,
    "paid": 0,
    "cancelled": 0
  },
  "partnership": {
    "id": "partnership-789",
    "partnerType": "affiliate",
    "commissionPercentage": 15.0
  }
}
```

#### `POST /api/partners/commissions/process`
**Descrição**: Processar comissões para um pedido específico
**Acesso**: Sistema/Webhook

```javascript
// Payload de exemplo
{
  "orderId": "order-456",
  "partnerCode": "AF8B2C1D" // opcional, se não informado busca automaticamente
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Comissões processadas com sucesso",
  "commissions": [
    {
      "id": "commission-123",
      "product_partner_id": "partnership-789",
      "order_id": "order-456",
      "commission_amount": 29.55,
      "commission_percentage": 15.0,
      "order_amount": 197.00,
      "status": "pending",
      "created_at": "2024-01-01T12:00:00Z"
    }
  ],
  "total": 1
}
```

#### `PUT /api/partners/commissions/:id/approve`
**Descrição**: Aprovar comissão específica
**Acesso**: Admin

```javascript
// Payload de exemplo
{
  "approvedBy": "user-001"
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Comissão aprovada com sucesso",
  "commission": {
    "id": "commission-123",
    "status": "approved",
    "approved_at": "2024-01-01T14:00:00Z",
    "approved_by": "user-001",
    "commission_amount": 29.55
  }
}
```

#### `PUT /api/partners/commissions/:id/pay`
**Descrição**: Marcar comissão como paga
**Acesso**: Admin

```javascript
// Payload de exemplo
{
  "paymentMethod": "pix",
  "paymentReference": "PIX123456789",
  "notes": "Pagamento realizado via PIX em 01/01/2024"
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Comissão marcada como paga",
  "commission": {
    "id": "commission-123",
    "status": "paid",
    "paid_at": "2024-01-01T16:00:00Z",
    "payment_method": "pix",
    "payment_reference": "PIX123456789",
    "payment_notes": "Pagamento realizado via PIX em 01/01/2024",
    "commission_amount": 29.55
  }
}
```

#### `PUT /api/partners/commissions/:id/cancel`
**Descrição**: Cancelar comissão
**Acesso**: Admin

```javascript
// Payload de exemplo
{
  "reason": "Pedido foi cancelado pelo cliente"
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Comissão cancelada",
  "commission": {
    "id": "commission-123",
    "status": "cancelled",
    "cancelled_at": "2024-01-01T18:00:00Z",
    "cancellation_reason": "Pedido foi cancelado pelo cliente"
  }
}
```

### 4. Estatísticas e Analytics

#### `GET /api/partners/commissions/stats/:partnerId`
**Descrição**: Obter estatísticas detalhadas de um parceiro
**Acesso**: Autenticado

**Query Parameters**:
- `startDate`: Data de início (YYYY-MM-DD)
- `endDate`: Data final (YYYY-MM-DD)

**Response Success**:
```json
{
  "success": true,
  "stats": {
    "overview": {
      "totalCommissions": 150,
      "totalAmount": 4567.85,
      "averageCommission": 30.45,
      "totalOrders": 150,
      "conversionRate": 85.2
    },
    "byStatus": {
      "pending": {
        "count": 5,
        "amount": 125.75
      },
      "approved": {
        "count": 20,
        "amount": 678.90
      },
      "paid": {
        "count": 120,
        "amount": 3650.20
      },
      "cancelled": {
        "count": 5,
        "amount": 113.00
      }
    },
    "byProduct": [
      {
        "product_id": "product-123",
        "product_name": "Curso de JavaScript",
        "count": 85,
        "amount": 2890.50,
        "commission_percentage": 15.0
      },
      {
        "product_id": "product-456",
        "product_name": "Curso de Python",
        "count": 65,
        "amount": 1677.35,
        "commission_percentage": 20.0
      }
    ],
    "monthly": [
      {
        "month": "2024-01",
        "count": 45,
        "amount": 1234.56
      },
      {
        "month": "2024-02",
        "count": 52,
        "amount": 1456.78
      }
    ],
    "topPerformance": {
      "bestMonth": {
        "month": "2024-02",
        "amount": 1456.78
      },
      "bestProduct": {
        "product_name": "Curso de JavaScript",
        "amount": 2890.50
      }
    }
  },
  "partnerId": "user-456",
  "period": {
    "startDate": "2024-01-01",
    "endDate": "2024-12-31"
  }
}
```

## Integração Frontend

### Dashboard de Parceiros
```jsx
import React, { useState, useEffect } from 'react';

const PartnerDashboard = ({ partnerId, token }) => {
  const [partnerships, setPartnerships] = useState([]);
  const [stats, setStats] = useState(null);
  const [commissions, setCommissions] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    loadPartnerData();
  }, []);
  
  const loadPartnerData = async () => {
    try {
      // Carregar parcerias
      const partnershipsResponse = await fetch(`/api/partners/user/${partnerId}`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const partnershipsData = await partnershipsResponse.json();
      setPartnerships(partnershipsData.partnerships);
      
      // Carregar estatísticas
      const statsResponse = await fetch(`/api/partners/commissions/stats/${partnerId}`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const statsData = await statsResponse.json();
      setStats(statsData.stats);
      
      // Carregar comissões recentes
      if (partnershipsData.partnerships.length > 0) {
        const partnershipId = partnershipsData.partnerships[0].id;
        const commissionsResponse = await fetch(`/api/partners/${partnershipId}/commissions`, {
          headers: { 'Authorization': `Bearer ${token}` }
        });
        const commissionsData = await commissionsResponse.json();
        setCommissions(commissionsData.commissions);
      }
      
    } catch (error) {
      console.error('Erro ao carregar dados:', error);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <div>Carregando...</div>;
  
  return (
    <div className="partner-dashboard">
      <h1>🤝 Dashboard do Parceiro</h1>
      
      {/* Estatísticas Gerais */}
      {stats && (
        <div className="stats-overview">
          <div className="stat-card">
            <h3>💰 Total Comissões</h3>
            <p className="value">R$ {stats.overview.totalAmount.toFixed(2)}</p>
            <p className="count">{stats.overview.totalCommissions} comissões</p>
          </div>
          
          <div className="stat-card">
            <h3>📊 Média por Comissão</h3>
            <p className="value">R$ {stats.overview.averageCommission.toFixed(2)}</p>
          </div>
          
          <div className="stat-card">
            <h3>🎯 Taxa de Conversão</h3>
            <p className="value">{stats.overview.conversionRate.toFixed(1)}%</p>
          </div>
        </div>
      )}
      
      {/* Parcerias Ativas */}
      <div className="partnerships-section">
        <h2>🤝 Suas Parcerias</h2>
        <div className="partnerships-grid">
          {partnerships.map(partnership => (
            <div key={partnership.id} className="partnership-card">
              <h3>{partnership.product.name}</h3>
              <div className="partnership-info">
                <span className="code">Código: {partnership.partner_code}</span>
                <span className="commission">{partnership.commission_percentage}% comissão</span>
                <span className={`status ${partnership.status}`}>
                  {partnership.status}
                </span>
              </div>
              <div className="partnership-actions">
                <button 
                  onClick={() => copyPartnerLink(partnership.partner_code)}
                  className="btn btn-primary"
                >
                  📋 Copiar Link
                </button>
                <button 
                  onClick={() => viewCommissions(partnership.id)}
                  className="btn btn-secondary"
                >
                  💰 Ver Comissões
                </button>
              </div>
            </div>
          ))}
        </div>
      </div>
      
      {/* Comissões Recentes */}
      <div className="commissions-section">
        <h2>💰 Comissões Recentes</h2>
        <div className="commissions-table">
          <table>
            <thead>
              <tr>
                <th>Data</th>
                <th>Produto</th>
                <th>Cliente</th>
                <th>Valor Pedido</th>
                <th>Comissão</th>
                <th>Status</th>
              </tr>
            </thead>
            <tbody>
              {commissions.slice(0, 10).map(commission => (
                <tr key={commission.id}>
                  <td>{new Date(commission.created_at).toLocaleDateString()}</td>
                  <td>{commission.order.product.name}</td>
                  <td>{commission.order.customer_info.name}</td>
                  <td>R$ {commission.order_amount.toFixed(2)}</td>
                  <td>R$ {commission.commission_amount.toFixed(2)}</td>
                  <td>
                    <span className={`status ${commission.status}`}>
                      {getStatusLabel(commission.status)}
                    </span>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  );
  
  const copyPartnerLink = (partnerCode) => {
    const link = `${window.location.origin}/checkout?partner=${partnerCode}`;
    navigator.clipboard.writeText(link);
    alert('Link copiado para a área de transferência!');
  };
  
  const viewCommissions = (partnershipId) => {
    // Implementar navegação para página de comissões
    window.location.href = `/partner/commissions/${partnershipId}`;
  };
  
  const getStatusLabel = (status) => {
    const labels = {
      pending: 'Pendente',
      approved: 'Aprovada',
      paid: 'Paga',
      cancelled: 'Cancelada'
    };
    return labels[status] || status;
  };
};
```

### Sistema de Checkout com Parceiros
```jsx
const CheckoutWithPartner = ({ productId }) => {
  const [partnerCode, setPartnerCode] = useState('');
  const [partnership, setPartnership] = useState(null);
  const [validatingCode, setValidatingCode] = useState(false);
  
  useEffect(() => {
    // Verificar se há código de parceiro na URL
    const urlParams = new URLSearchParams(window.location.search);
    const codeFromUrl = urlParams.get('partner');
    if (codeFromUrl) {
      setPartnerCode(codeFromUrl);
      validatePartnerCode(codeFromUrl);
    }
  }, []);
  
  const validatePartnerCode = async (code) => {
    if (!code) return;
    
    setValidatingCode(true);
    try {
      const response = await fetch('/api/partners/validate-code', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          partnerCode: code,
          productId: productId
        })
      });
      
      const data = await response.json();
      if (data.success && data.valid) {
        setPartnership(data.partnership);
      } else {
        setPartnership(null);
        alert('Código de parceiro inválido');
      }
    } catch (error) {
      console.error('Erro ao validar código:', error);
    } finally {
      setValidatingCode(false);
    }
  };
  
  const processOrderWithPartner = async (orderData) => {
    // Incluir informações do parceiro no pedido
    const orderWithPartner = {
      ...orderData,
      partnerCode: partnerCode,
      partnershipId: partnership?.id
    };
    
    // Processar pedido normalmente
    const orderResponse = await processOrder(orderWithPartner);
    
    // Se pedido foi criado com sucesso, processar comissões
    if (orderResponse.success) {
      await fetch('/api/partners/commissions/process', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          orderId: orderResponse.order.id,
          partnerCode: partnerCode
        })
      });
    }
    
    return orderResponse;
  };
  
  return (
    <div className="checkout-with-partner">
      {/* Seção do Código de Parceiro */}
      <div className="partner-section">
        <h3>🤝 Código de Parceiro</h3>
        
        <div className="partner-input-group">
          <input
            type="text"
            placeholder="Digite o código do seu parceiro"
            value={partnerCode}
            onChange={(e) => setPartnerCode(e.target.value.toUpperCase())}
            className="partner-code-input"
          />
          <button 
            onClick={() => validatePartnerCode(partnerCode)}
            disabled={validatingCode || !partnerCode}
            className="btn btn-primary"
          >
            {validatingCode ? 'Validando...' : 'Validar'}
          </button>
        </div>
        
        {partnership && (
          <div className="partner-info">
            <div className="partner-success">
              ✅ Código válido! Você está comprando através de <strong>{partnership.partner.name}</strong>
            </div>
            <div className="commission-info">
              💰 O parceiro receberá {partnership.commission_percentage}% de comissão desta venda
            </div>
          </div>
        )}
      </div>
      
      {/* Resto do checkout */}
      <div className="checkout-form">
        {/* Formulário de checkout normal */}
      </div>
    </div>
  );
};
```

### Admin - Gestão de Parcerias
```jsx
const AdminPartnerships = ({ token }) => {
  const [partnerships, setPartnerships] = useState([]);
  const [pendingCommissions, setPendingCommissions] = useState([]);
  const [newPartnership, setNewPartnership] = useState({
    productId: '',
    partnerId: '',
    partnerType: 'affiliate',
    commissionPercentage: 10
  });
  
  const createPartnership = async () => {
    try {
      const response = await fetch('/api/partners', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(newPartnership)
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Parceria criada com sucesso!');
        loadPartnerships();
        setNewPartnership({
          productId: '',
          partnerId: '',
          partnerType: 'affiliate',
          commissionPercentage: 10
        });
      }
    } catch (error) {
      alert('Erro ao criar parceria: ' + error.message);
    }
  };
  
  const approveCommission = async (commissionId) => {
    try {
      const response = await fetch(`/api/partners/commissions/${commissionId}/approve`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          approvedBy: 'current-admin-id'
        })
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Comissão aprovada!');
        loadPendingCommissions();
      }
    } catch (error) {
      alert('Erro ao aprovar comissão: ' + error.message);
    }
  };
  
  const payCommission = async (commissionId, paymentData) => {
    try {
      const response = await fetch(`/api/partners/commissions/${commissionId}/pay`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(paymentData)
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Comissão marcada como paga!');
        loadPendingCommissions();
      }
    } catch (error) {
      alert('Erro ao marcar comissão como paga: ' + error.message);
    }
  };
  
  return (
    <div className="admin-partnerships">
      <h1>🤝 Gestão de Parcerias</h1>
      
      {/* Formulário para nova parceria */}
      <div className="new-partnership-form">
        <h2>➕ Nova Parceria</h2>
        {/* Formulário de criação */}
      </div>
      
      {/* Lista de parcerias */}
      <div className="partnerships-list">
        <h2>📋 Parcerias Existentes</h2>
        {/* Lista de parcerias */}
      </div>
      
      {/* Comissões pendentes */}
      <div className="pending-commissions">
        <h2>💰 Comissões Pendentes</h2>
        {/* Lista de comissões para aprovação */}
      </div>
    </div>
  );
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Parâmetros obrigatórios ausentes ou inválidos |
| 404 | Parceria, produto ou usuário não encontrado |
| 409 | Parceria já existe entre produto e usuário |
| 500 | Erro interno do servidor |

## Considerações de Segurança

1. **Validação de Parceiros**: Verificação de existência de usuários e produtos
2. **Códigos Únicos**: Geração de códigos criptograficamente seguros
3. **Auditoria Completa**: Log de todas as operações de comissões
4. **Validação de Percentuais**: Limitação entre 0% e 100%
5. **Proteção de Exclusão**: Prevenção de remoção com comissões pendentes

## Melhores Práticas

1. **Códigos Únicos**: Use códigos de 8 caracteres para facilitar compartilhamento
2. **Aprovação Manual**: Configure aprovação manual para comissões altas
3. **Pagamentos Programados**: Estabeleça cronograma regular de pagamentos
4. **Relatórios Detalhados**: Mantenha registros completos para auditoria
5. **Comunicação Clara**: Notifique parceiros sobre mudanças de status

Este sistema oferece gestão completa de parcerias, desde a criação até o pagamento de comissões, com controles de segurança e auditoria apropriados.
