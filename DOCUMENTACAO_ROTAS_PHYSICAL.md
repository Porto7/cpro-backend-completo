# Documentação - Produtos Físicos e Frete (routes/physical.js)

## Visão Geral
Sistema completo para gerenciamento de produtos físicos, incluindo controle de estoque, cálculo de frete, gestão de envios e rastreamento de entregas.

## Características Principais
- **Produtos Físicos**: Configuração de peso, dimensões e estoque
- **Cálculo de Frete**: Integração com Correios e transportadoras
- **Controle de Estoque**: Gestão automática de inventário
- **Gestão de Envios**: Status de envio e rastreamento
- **Múltiplas Transportadoras**: Suporte a diversos métodos de entrega
- **Validação de CEP**: Verificação de endereços de entrega
- **Relatórios Avançados**: Analytics de vendas e envios

## Status de Envio

| Status | Descrição |
|--------|-----------|
| `pending` | Envio pendente de processamento |
| `processing` | Preparando para envio |
| `shipped` | Produto despachado |
| `in_transit` | Em trânsito para destino |
| `delivered` | Entregue ao destinatário |
| `failed` | Falha na entrega |

## Métodos de Envio

| Método | Descrição | Prazo Estimado |
|--------|-----------|----------------|
| `correios_pac` | PAC - Correios | 5-10 dias úteis |
| `correios_sedex` | SEDEX - Correios | 1-3 dias úteis |
| `transportadora` | Transportadora parceira | 3-7 dias úteis |
| `motoboy` | Entrega local | Mesmo dia |
| `retirada` | Retirada no local | Imediato |

## Rotas Implementadas

### 1. Configuração de Produtos Físicos

#### `POST /api/physical/products`
**Descrição**: Configurar produto como físico
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "productId": "product-123",
  "weight": 0.5, // kg
  "dimensions": {
    "height": 10, // cm
    "width": 15,  // cm
    "length": 20  // cm
  },
  "stock_quantity": 100,
  "sku": "PROD-123-PHYS",
  "category": "electronics",
  "fragile": false,
  "requires_assembly": false,
  "warranty_period": 12, // meses
  "shipping_restrictions": {
    "max_distance": null, // km, null = sem limite
    "allowed_states": ["SP", "RJ", "MG"], // null = todos os estados
    "forbidden_zipcodes": ["01000-000"]
  },
  "handling_time": 2 // dias para processar
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Produto físico configurado com sucesso",
  "data": {
    "id": "physical-456",
    "product_id": "product-123",
    "weight": 0.5,
    "dimensions": {
      "height": 10,
      "width": 15,
      "length": 20
    },
    "stock_quantity": 100,
    "sku": "PROD-123-PHYS",
    "category": "electronics",
    "fragile": false,
    "requires_assembly": false,
    "warranty_period": 12,
    "shipping_restrictions": {
      "max_distance": null,
      "allowed_states": ["SP", "RJ", "MG"],
      "forbidden_zipcodes": ["01000-000"]
    },
    "handling_time": 2,
    "created_at": "2024-01-01T12:00:00Z",
    "updated_at": "2024-01-01T12:00:00Z"
  }
}
```

**Response Error**:
```json
{
  "success": false,
  "message": "Dimensões devem incluir altura, largura e comprimento"
}
```

### 2. Gestão de Estoque

#### `PUT /api/physical/products/:productId/inventory`
**Descrição**: Atualizar estoque do produto
**Acesso**: Producer, Admin

```javascript
// Definir quantidade exata
{
  "quantity": 50,
  "operation": "set"
}

// Adicionar ao estoque
{
  "quantity": 25,
  "operation": "add"
}

// Subtrair do estoque
{
  "quantity": 10,
  "operation": "subtract"
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Estoque atualizado com sucesso",
  "data": {
    "id": "physical-456",
    "product_id": "product-123",
    "stock_quantity": 50,
    "previous_quantity": 25,
    "operation_performed": "set",
    "updated_at": "2024-01-01T14:00:00Z",
    "stock_status": "available", // available, low_stock, out_of_stock
    "low_stock_threshold": 10
  }
}
```

#### `GET /api/physical/inventory`
**Descrição**: Relatório de estoque de todos os produtos
**Acesso**: Producer, Admin

**Response Success**:
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "physical-456",
        "product_id": "product-123",
        "product_name": "Smartphone XYZ",
        "sku": "PROD-123-PHYS",
        "stock_quantity": 50,
        "stock_status": "available",
        "low_stock_threshold": 10,
        "reserved_quantity": 5, // Produtos em pedidos pendentes
        "available_quantity": 45,
        "last_sale": "2024-01-01T10:00:00Z",
        "total_sold": 150,
        "category": "electronics"
      },
      {
        "id": "physical-457",
        "product_id": "product-124",
        "product_name": "Livro ABC",
        "sku": "BOOK-124",
        "stock_quantity": 3,
        "stock_status": "low_stock",
        "low_stock_threshold": 5,
        "reserved_quantity": 0,
        "available_quantity": 3,
        "last_sale": "2024-01-01T08:00:00Z",
        "total_sold": 47,
        "category": "books"
      }
    ],
    "summary": {
      "total_products": 2,
      "total_stock_value": 15670.50,
      "low_stock_count": 1,
      "out_of_stock_count": 0,
      "total_reserved": 5
    }
  }
}
```

### 3. Cálculo de Frete

#### `POST /api/physical/calculate-shipping`
**Descrição**: Calcular opções de frete para um produto
**Acesso**: Público

```javascript
// Payload de exemplo
{
  "productId": "product-123",
  "destinationZipcode": "01310-100",
  "quantity": 2
}
```

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "method": "correios_pac",
      "name": "PAC - Correios",
      "price": 15.50,
      "estimated_days": 7,
      "estimated_date": "2024-01-08",
      "carrier": "Correios",
      "service_code": "04510",
      "description": "Entrega econômica dos Correios"
    },
    {
      "method": "correios_sedex",
      "name": "SEDEX - Correios",
      "price": 25.80,
      "estimated_days": 2,
      "estimated_date": "2024-01-03",
      "carrier": "Correios",
      "service_code": "04014",
      "description": "Entrega expressa dos Correios"
    },
    {
      "method": "transportadora",
      "name": "Transportadora",
      "price": 18.90,
      "estimated_days": 5,
      "estimated_date": "2024-01-06",
      "carrier": "Transportadora XYZ",
      "description": "Entrega via transportadora parceira"
    }
  ]
}
```

**Response Error**:
```json
{
  "success": false,
  "error": "CEP de destino não atendido pelos Correios"
}
```

#### `POST /api/physical/validate-zipcode`
**Descrição**: Validar CEP e obter informações de endereço
**Acesso**: Público

```javascript
// Payload de exemplo
{
  "zipcode": "01310-100"
}
```

**Response Success**:
```json
{
  "success": true,
  "data": {
    "zipcode": "01310-100",
    "street": "Avenida Paulista",
    "neighborhood": "Bela Vista",
    "city": "São Paulo",
    "state": "SP",
    "valid": true,
    "deliverable": true,
    "restrictions": []
  }
}
```

**Response Error**:
```json
{
  "success": false,
  "error": "CEP não encontrado",
  "data": {
    "zipcode": "00000-000",
    "valid": false
  }
}
```

### 4. Gestão de Envios

#### `POST /api/physical/orders/:orderId/shipping`
**Descrição**: Criar informações de envio para um pedido
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "shipping_address": {
    "street": "Rua das Flores, 123",
    "neighborhood": "Centro",
    "city": "São Paulo",
    "state": "SP",
    "zipcode": "01310-100",
    "number": "123",
    "complement": "Apto 45"
  },
  "shipping_method": "correios_sedex",
  "shipping_cost": 25.80,
  "estimated_delivery": "2024-01-03",
  "carrier": "Correios",
  "service_code": "04014",
  "handling_time": 2,
  "notes": "Entregar no portão"
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Informações de envio criadas com sucesso",
  "data": {
    "id": "shipping-789",
    "order_id": "order-456",
    "status": "pending",
    "shipping_address": {
      "street": "Rua das Flores, 123",
      "neighborhood": "Centro",
      "city": "São Paulo",
      "state": "SP",
      "zipcode": "01310-100",
      "number": "123",
      "complement": "Apto 45"
    },
    "shipping_method": "correios_sedex",
    "shipping_cost": 25.80,
    "estimated_delivery": "2024-01-03",
    "carrier": "Correios",
    "service_code": "04014",
    "tracking_code": null,
    "tracking_url": null,
    "shipped_at": null,
    "delivered_at": null,
    "created_at": "2024-01-01T12:00:00Z"
  }
}
```

#### `PUT /api/physical/orders/:orderId/shipping/status`
**Descrição**: Atualizar status de envio
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "status": "shipped",
  "trackingCode": "BR123456789BR",
  "trackingUrl": "https://www.correios.com.br/track/BR123456789BR",
  "carrier": "Correios"
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Status de envio atualizado com sucesso",
  "data": {
    "id": "shipping-789",
    "order_id": "order-456",
    "status": "shipped",
    "tracking_code": "BR123456789BR",
    "tracking_url": "https://www.correios.com.br/track/BR123456789BR",
    "carrier": "Correios",
    "shipped_at": "2024-01-01T15:00:00Z",
    "estimated_delivery": "2024-01-03",
    "status_history": [
      {
        "status": "pending",
        "timestamp": "2024-01-01T12:00:00Z",
        "notes": "Envio criado"
      },
      {
        "status": "processing",
        "timestamp": "2024-01-01T13:00:00Z",
        "notes": "Preparando para envio"
      },
      {
        "status": "shipped",
        "timestamp": "2024-01-01T15:00:00Z",
        "notes": "Produto despachado"
      }
    ]
  }
}
```

#### `POST /api/physical/orders/:orderId/tracking`
**Descrição**: Adicionar código de rastreamento
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "trackingCode": "BR123456789BR",
  "carrier": "Correios"
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Código de rastreamento adicionado com sucesso",
  "data": {
    "id": "shipping-789",
    "tracking_code": "BR123456789BR",
    "tracking_url": "https://www.correios.com.br/track/BR123456789BR",
    "carrier": "Correios",
    "status": "shipped",
    "updated_at": "2024-01-01T15:30:00Z"
  }
}
```

### 5. Relatórios e Consultas

#### `GET /api/physical/shippings`
**Descrição**: Listar envios por status
**Acesso**: Producer, Admin

**Query Parameters**:
- `status`: Filtrar por status (pending, processing, shipped, in_transit, delivered, failed)

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "shipping-789",
      "order_id": "order-456",
      "status": "shipped",
      "tracking_code": "BR123456789BR",
      "shipping_method": "correios_sedex",
      "shipping_cost": 25.80,
      "estimated_delivery": "2024-01-03",
      "shipped_at": "2024-01-01T15:00:00Z",
      "order": {
        "id": "order-456",
        "customer_name": "João Silva",
        "customer_email": "joao@email.com",
        "total_amount": 225.80,
        "product": {
          "id": "product-123",
          "name": "Smartphone XYZ"
        }
      }
    }
  ]
}
```

#### `GET /api/physical/stats`
**Descrição**: Estatísticas de envios e vendas físicas
**Acesso**: Producer, Admin

**Query Parameters**:
- `startDate`: Data de início (YYYY-MM-DD)
- `endDate`: Data final (YYYY-MM-DD)

**Response Success**:
```json
{
  "success": true,
  "data": {
    "overview": {
      "total_orders": 156,
      "total_revenue": 31250.75,
      "total_shipping_cost": 3890.50,
      "average_shipping_cost": 24.94,
      "orders_shipped": 142,
      "orders_delivered": 138,
      "shipping_success_rate": 97.2
    },
    "by_status": {
      "pending": 5,
      "processing": 9,
      "shipped": 12,
      "in_transit": 8,
      "delivered": 138,
      "failed": 2
    },
    "by_method": {
      "correios_pac": {
        "count": 89,
        "total_cost": 1334.50,
        "average_cost": 15.00,
        "average_delivery_days": 6.8
      },
      "correios_sedex": {
        "count": 45,
        "total_cost": 1170.00,
        "average_cost": 26.00,
        "average_delivery_days": 2.1
      },
      "transportadora": {
        "count": 22,
        "total_cost": 415.60,
        "average_cost": 18.89,
        "average_delivery_days": 4.5
      }
    },
    "by_location": [
      {
        "state": "SP",
        "count": 78,
        "percentage": 50.0
      },
      {
        "state": "RJ",
        "count": 34,
        "percentage": 21.8
      },
      {
        "state": "MG",
        "count": 25,
        "percentage": 16.0
      }
    ],
    "monthly_trend": [
      {
        "month": "2024-01",
        "orders": 45,
        "revenue": 9750.25,
        "shipping_cost": 1125.50
      },
      {
        "month": "2024-02",
        "orders": 52,
        "revenue": 11230.80,
        "shipping_cost": 1298.70
      }
    ]
  }
}
```

#### `GET /api/physical/shipping-methods`
**Descrição**: Listar métodos de envio disponíveis
**Acesso**: Producer, Admin

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "correios_pac",
      "name": "PAC - Correios",
      "description": "Entrega econômica dos Correios",
      "estimatedDays": "5-10 dias úteis",
      "carrier": "Correios",
      "service_code": "04510"
    },
    {
      "id": "correios_sedex",
      "name": "SEDEX - Correios",
      "description": "Entrega expressa dos Correios",
      "estimatedDays": "1-3 dias úteis",
      "carrier": "Correios",
      "service_code": "04014"
    },
    {
      "id": "transportadora",
      "name": "Transportadora",
      "description": "Entrega via transportadora parceira",
      "estimatedDays": "3-7 dias úteis",
      "carrier": "Transportadora Parceira"
    },
    {
      "id": "motoboy",
      "name": "Motoboy",
      "description": "Entrega local via motoboy",
      "estimatedDays": "Mesmo dia",
      "carrier": "Motoboy Local"
    },
    {
      "id": "retirada",
      "name": "Retirada no Local",
      "description": "Cliente retira no endereço do vendedor",
      "estimatedDays": "Imediato",
      "carrier": null
    }
  ]
}
```

## Integração Frontend

### Configuração de Produto Físico
```jsx
import React, { useState } from 'react';

const PhysicalProductConfig = ({ productId, token }) => {
  const [config, setConfig] = useState({
    weight: '',
    dimensions: { height: '', width: '', length: '' },
    stock_quantity: '',
    sku: '',
    fragile: false,
    handling_time: 2
  });
  
  const savePhysicalConfig = async () => {
    try {
      const response = await fetch('/api/physical/products', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          productId,
          ...config
        })
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Produto físico configurado com sucesso!');
      }
    } catch (error) {
      alert('Erro ao configurar produto: ' + error.message);
    }
  };
  
  return (
    <div className="physical-product-config">
      <h2>📦 Configuração Produto Físico</h2>
      
      <div className="config-form">
        <div className="form-group">
          <label>Peso (kg):</label>
          <input
            type="number"
            step="0.01"
            value={config.weight}
            onChange={(e) => setConfig({...config, weight: parseFloat(e.target.value)})}
          />
        </div>
        
        <div className="dimensions-group">
          <h4>Dimensões (cm):</h4>
          <div className="dimensions-inputs">
            <input
              type="number"
              placeholder="Altura"
              value={config.dimensions.height}
              onChange={(e) => setConfig({
                ...config,
                dimensions: {...config.dimensions, height: parseInt(e.target.value)}
              })}
            />
            <input
              type="number"
              placeholder="Largura"
              value={config.dimensions.width}
              onChange={(e) => setConfig({
                ...config,
                dimensions: {...config.dimensions, width: parseInt(e.target.value)}
              })}
            />
            <input
              type="number"
              placeholder="Comprimento"
              value={config.dimensions.length}
              onChange={(e) => setConfig({
                ...config,
                dimensions: {...config.dimensions, length: parseInt(e.target.value)}
              })}
            />
          </div>
        </div>
        
        <div className="form-group">
          <label>Estoque Inicial:</label>
          <input
            type="number"
            value={config.stock_quantity}
            onChange={(e) => setConfig({...config, stock_quantity: parseInt(e.target.value)})}
          />
        </div>
        
        <div className="form-group">
          <label>SKU:</label>
          <input
            type="text"
            value={config.sku}
            onChange={(e) => setConfig({...config, sku: e.target.value})}
          />
        </div>
        
        <div className="form-group">
          <label>
            <input
              type="checkbox"
              checked={config.fragile}
              onChange={(e) => setConfig({...config, fragile: e.target.checked})}
            />
            Produto frágil
          </label>
        </div>
        
        <div className="form-group">
          <label>Tempo de processamento (dias):</label>
          <input
            type="number"
            value={config.handling_time}
            onChange={(e) => setConfig({...config, handling_time: parseInt(e.target.value)})}
          />
        </div>
        
        <button onClick={savePhysicalConfig} className="btn btn-primary">
          💾 Salvar Configuração
        </button>
      </div>
    </div>
  );
};
```

### Calculadora de Frete
```jsx
const ShippingCalculator = ({ productId }) => {
  const [zipcode, setZipcode] = useState('');
  const [quantity, setQuantity] = useState(1);
  const [shippingOptions, setShippingOptions] = useState([]);
  const [loading, setLoading] = useState(false);
  
  const calculateShipping = async () => {
    if (!zipcode) return;
    
    setLoading(true);
    try {
      const response = await fetch('/api/physical/calculate-shipping', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          productId,
          destinationZipcode: zipcode,
          quantity
        })
      });
      
      const data = await response.json();
      if (data.success) {
        setShippingOptions(data.data);
      } else {
        alert('Erro ao calcular frete: ' + data.error);
      }
    } catch (error) {
      alert('Erro ao calcular frete: ' + error.message);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="shipping-calculator">
      <h3>📦 Calcular Frete</h3>
      
      <div className="calculator-inputs">
        <input
          type="text"
          placeholder="Digite seu CEP"
          value={zipcode}
          onChange={(e) => setZipcode(e.target.value)}
          maxLength="9"
        />
        <input
          type="number"
          min="1"
          value={quantity}
          onChange={(e) => setQuantity(parseInt(e.target.value))}
        />
        <button onClick={calculateShipping} disabled={loading}>
          {loading ? 'Calculando...' : '🔍 Calcular'}
        </button>
      </div>
      
      {shippingOptions.length > 0 && (
        <div className="shipping-options">
          <h4>Opções de Entrega:</h4>
          {shippingOptions.map((option, index) => (
            <div key={index} className="shipping-option">
              <div className="option-info">
                <strong>{option.name}</strong>
                <span className="price">R$ {option.price.toFixed(2)}</span>
              </div>
              <div className="option-details">
                <span className="delivery-time">
                  Entrega em {option.estimated_days} dias úteis
                </span>
                <span className="delivery-date">
                  Até {new Date(option.estimated_date).toLocaleDateString()}
                </span>
              </div>
              <p className="description">{option.description}</p>
            </div>
          ))}
        </div>
      )}
    </div>
  );
};
```

### Gestão de Estoque
```jsx
const InventoryManagement = ({ token }) => {
  const [inventory, setInventory] = useState([]);
  const [selectedProduct, setSelectedProduct] = useState(null);
  const [stockUpdate, setStockUpdate] = useState({
    quantity: '',
    operation: 'set'
  });
  
  useEffect(() => {
    loadInventory();
  }, []);
  
  const loadInventory = async () => {
    try {
      const response = await fetch('/api/physical/inventory', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setInventory(data.data.products);
    } catch (error) {
      console.error('Erro ao carregar estoque:', error);
    }
  };
  
  const updateStock = async (productId) => {
    try {
      const response = await fetch(`/api/physical/products/${productId}/inventory`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(stockUpdate)
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Estoque atualizado com sucesso!');
        loadInventory();
        setSelectedProduct(null);
        setStockUpdate({ quantity: '', operation: 'set' });
      }
    } catch (error) {
      alert('Erro ao atualizar estoque: ' + error.message);
    }
  };
  
  return (
    <div className="inventory-management">
      <h2>📊 Gestão de Estoque</h2>
      
      <div className="inventory-summary">
        <div className="summary-card">
          <h3>Total de Produtos</h3>
          <span className="value">{inventory.length}</span>
        </div>
        <div className="summary-card">
          <h3>Estoque Baixo</h3>
          <span className="value warning">
            {inventory.filter(p => p.stock_status === 'low_stock').length}
          </span>
        </div>
        <div className="summary-card">
          <h3>Sem Estoque</h3>
          <span className="value danger">
            {inventory.filter(p => p.stock_status === 'out_of_stock').length}
          </span>
        </div>
      </div>
      
      <div className="inventory-table">
        <table>
          <thead>
            <tr>
              <th>Produto</th>
              <th>SKU</th>
              <th>Estoque</th>
              <th>Reservado</th>
              <th>Disponível</th>
              <th>Status</th>
              <th>Ações</th>
            </tr>
          </thead>
          <tbody>
            {inventory.map(product => (
              <tr key={product.id}>
                <td>{product.product_name}</td>
                <td>{product.sku}</td>
                <td>{product.stock_quantity}</td>
                <td>{product.reserved_quantity}</td>
                <td>{product.available_quantity}</td>
                <td>
                  <span className={`status ${product.stock_status}`}>
                    {getStatusLabel(product.stock_status)}
                  </span>
                </td>
                <td>
                  <button 
                    onClick={() => setSelectedProduct(product)}
                    className="btn btn-small"
                  >
                    ✏️ Editar
                  </button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
      
      {selectedProduct && (
        <div className="stock-update-modal">
          <h3>Atualizar Estoque - {selectedProduct.product_name}</h3>
          
          <div className="update-form">
            <select
              value={stockUpdate.operation}
              onChange={(e) => setStockUpdate({...stockUpdate, operation: e.target.value})}
            >
              <option value="set">Definir quantidade</option>
              <option value="add">Adicionar ao estoque</option>
              <option value="subtract">Subtrair do estoque</option>
            </select>
            
            <input
              type="number"
              placeholder="Quantidade"
              value={stockUpdate.quantity}
              onChange={(e) => setStockUpdate({...stockUpdate, quantity: e.target.value})}
            />
            
            <div className="modal-actions">
              <button 
                onClick={() => updateStock(selectedProduct.product_id)}
                className="btn btn-primary"
              >
                💾 Salvar
              </button>
              <button 
                onClick={() => setSelectedProduct(null)}
                className="btn btn-secondary"
              >
                ❌ Cancelar
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
  
  const getStatusLabel = (status) => {
    const labels = {
      available: 'Disponível',
      low_stock: 'Estoque Baixo',
      out_of_stock: 'Sem Estoque'
    };
    return labels[status] || status;
  };
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Parâmetros obrigatórios ausentes ou inválidos |
| 404 | Produto físico ou envio não encontrado |
| 403 | Acesso negado (apenas Producer/Admin) |
| 500 | Erro interno ou falha na API externa |

## Considerações de Segurança

1. **Validação de Dados**: Verificação rigorosa de peso, dimensões e estoque
2. **Controle de Acesso**: Apenas produtores e admins podem gerenciar
3. **Validação de CEP**: Verificação real através de APIs externas
4. **Auditoria**: Log de todas as operações de estoque e envio
5. **Prevenção de Estoque Negativo**: Validações para evitar vendas sem estoque

## Melhores Práticas

1. **Gestão de Estoque**: Mantenha sempre atualizado e monitore níveis baixos
2. **Informações Precisas**: Peso e dimensões corretos garantem cálculo preciso
3. **Múltiplas Opções**: Ofereça diferentes métodos de envio
4. **Rastreamento**: Sempre forneça códigos de rastreamento
5. **Comunicação**: Mantenha clientes informados sobre status do envio

Este sistema oferece gestão completa de produtos físicos, desde configuração até entrega, com integração robusta para cálculo de frete e controle de estoque.
