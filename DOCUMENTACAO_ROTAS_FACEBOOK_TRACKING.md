# Documentação - Rastreamento Facebook (routes/facebookTracking.js)

## Visão Geral
Sistema completo de integração com Facebook Ads através da Conversion API. Permite rastreamento avançado de eventos, configuração de pixels, autenticação OAuth e gestão de conversões para otimização de campanhas publicitárias.

## Características Principais
- **Conversion API**: Integração direta com Facebook para rastreamento server-side
- **OAuth 2.0**: Autenticação segura com Facebook Business
- **Pixel Management**: Configuração e gestão de Facebook Pixels
- **Event Tracking**: Envio de eventos personalizados e padrões
- **Product-Specific**: Configurações específicas por produto
- **Test Mode**: Ambiente de testes com Event Test API
- **Real-time Logging**: Monitoramento e auditoria de eventos

## Eventos Suportados

| Evento | Descrição | Quando Usar |
|--------|-----------|-------------|
| `ViewContent` | Visualização de produto/página | Acesso a página de produto |
| `AddToCart` | Adicionar ao carrinho | Item adicionado ao checkout |
| `InitiateCheckout` | Iniciar checkout | Início do processo de compra |
| `AddPaymentInfo` | Informações de pagamento | Dados de pagamento inseridos |
| `Purchase` | Compra realizada | Transação concluída |
| `Lead` | Lead gerado | Cadastro ou interesse |
| `CompleteRegistration` | Registro completo | Conta criada |
| `Subscribe` | Assinatura | Plano ou curso adquirido |

## Rotas Implementadas

### 1. Autenticação OAuth

#### `GET /api/facebook/auth/facebook`
**Descrição**: Iniciar processo de autenticação OAuth com Facebook
**Acesso**: Usuário autenticado

**Response**: Redirecionamento para Facebook OAuth
```
Status: 302 Redirect
Location: https://www.facebook.com/v18.0/dialog/oauth?client_id=...
```

#### `GET /api/facebook/auth/facebook/callback`
**Descrição**: Callback da autenticação OAuth (interno)
**Acesso**: Sistema (chamado pelo Facebook)

**Query Parameters**:
- `code`: Código de autorização do Facebook
- `state`: ID do usuário (para segurança)

**Response Success**: Redirecionamento para dashboard
```
Status: 302 Redirect
Location: /dashboard?fb_connected=true
```

### 2. Configuração de Pixel

#### `POST /api/facebook/config/pixel`
**Descrição**: Configurar Facebook Pixel para o usuário
**Acesso**: Usuário autenticado

```javascript
// Payload com OAuth (configuração inicial)
{
  "pixel_id": "123456789012345"
}

// Payload com Access Token direto
{
  "pixel_id": "123456789012345",
  "access_token": "EAAx...",
  "test_event_code": "TEST12345"
}
```

**Response Success - OAuth Necessário**:
```json
{
  "status": "success",
  "message": "Configuração do pixel atualizada com sucesso",
  "data": {
    "pixel_id": "123456789012345",
    "test_event_code": null,
    "updated_at": "2024-01-01T12:00:00Z",
    "status": "pending_auth",
    "needs_oauth": true,
    "next_step": "Faça autenticação OAuth para ativar o rastreamento: GET /api/facebook/auth/facebook"
  }
}
```

**Response Success - Configuração Completa**:
```json
{
  "status": "success",
  "message": "Pixel configurado e ativado com sucesso",
  "data": {
    "pixel_id": "123456789012345",
    "test_event_code": "TEST12345",
    "updated_at": "2024-01-01T12:00:00Z",
    "status": "active",
    "needs_oauth": false,
    "next_step": "Sistema pronto para rastreamento de eventos"
  }
}
```

### 3. Envio de Eventos

#### `POST /api/facebook/track`
**Descrição**: Enviar evento para Facebook Conversion API
**Acesso**: Público (para tracking do frontend)

```javascript
// Payload de exemplo
{
  "pixel_id": "123456789012345",
  "event_name": "Purchase",
  "event_time": 1704110400, // Unix timestamp
  "user_data": {
    "em": "usuario@exemplo.com",
    "ph": "+5511999999999",
    "fn": "João",
    "ln": "Silva"
  },
  "custom_data": {
    "value": 299.90,
    "currency": "BRL",
    "content_ids": ["product-123"],
    "content_type": "product",
    "num_items": 1
  },
  "product_id": "product-123" // Opcional: usar configuração específica do produto
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Evento enviado com sucesso",
  "event_id": "fb_event_12345",
  "events_received": 1,
  "messages": [],
  "fbtrace_id": "AXj8K..."
}
```

**Response Error - Configuração Inválida**:
```json
{
  "error": "Pixel não configurado ou token inválido",
  "pixel_id": "123456789012345"
}
```

### 4. Configuração por Produto

#### `POST /api/facebook/products/:productId/config`
**Descrição**: Configurar pixel específico para um produto
**Acesso**: Usuário autenticado

```javascript
// Payload de exemplo
{
  "pixel_id": "123456789012345",
  "access_token": "EAAx...",
  "test_event_code": "TEST12345",
  "name": "Pixel - Curso JavaScript"
}
```

**Response Success**:
```json
{
  "status": "success",
  "message": "Pixel configurado para o produto com sucesso",
  "data": {
    "product_id": "product-123",
    "pixel_id": "123456789012345",
    "test_event_code": "TEST12345",
    "name": "Pixel - Curso JavaScript",
    "status": "active",
    "created_at": "2024-01-01T12:00:00Z",
    "updated_at": "2024-01-01T12:00:00Z"
  }
}
```

#### `GET /api/facebook/products/:productId/configs`
**Descrição**: Listar configurações de pixel de um produto
**Acesso**: Usuário autenticado

**Response Success**:
```json
{
  "status": "success",
  "data": {
    "product_id": "product-123",
    "configs": [
      {
        "id": "config-456",
        "pixel_id": "123456789012345",
        "test_event_code": "TEST12345",
        "name": "Pixel - Curso JavaScript",
        "status": "active",
        "created_at": "2024-01-01T12:00:00Z",
        "updated_at": "2024-01-01T12:00:00Z"
      },
      {
        "id": "config-457",
        "pixel_id": "987654321098765",
        "test_event_code": "TEST67890",
        "name": "Pixel - Campanha Black Friday",
        "status": "active",
        "created_at": "2024-01-01T10:00:00Z",
        "updated_at": "2024-01-01T10:00:00Z"
      }
    ],
    "total": 2
  }
}
```

#### `DELETE /api/facebook/products/:productId/configs/:pixelId`
**Descrição**: Remover configuração de pixel de um produto
**Acesso**: Usuário autenticado

**Response Success**:
```json
{
  "status": "success",
  "message": "Configuração de pixel removida do produto com sucesso",
  "data": {
    "product_id": "product-123",
    "pixel_id": "123456789012345"
  }
}
```

### 5. Testes de Eventos

#### `POST /api/facebook/test-event`
**Descrição**: Enviar evento de teste para validação
**Acesso**: Usuário autenticado

```javascript
// Payload de exemplo
{
  "pixelId": "123456789012345",
  "eventName": "Purchase",
  "userData": {
    "em": "teste@exemplo.com",
    "ph": "+5511999999999",
    "fn": "João",
    "ln": "Teste"
  },
  "customData": {
    "value": 99.90,
    "currency": "BRL",
    "content_ids": ["test-product"],
    "content_type": "product"
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Evento de teste enviado com sucesso",
  "facebook_response": {
    "events_received": 1,
    "messages": [],
    "fbtrace_id": "AXj8K..."
  },
  "processed_data": {
    "event_name": "Purchase",
    "user_data": {
      "em": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3",
      "ph": "8c15fd6dbf85f26b1b85b74d7a27b5f3...criptografado...",
      "fn": "4ea5c5075966f43b4b4b6cca76e5fda...criptografado...",
      "ln": "b4e4a5a4d7c7e4d4f7e2b6c8a9d5e3f...criptografado..."
    },
    "custom_data": {
      "value": 99.90,
      "currency": "BRL",
      "content_ids": ["test-product"],
      "content_type": "product"
    }
  }
}
```

#### `POST /api/facebook/products/:productId/test-event`
**Descrição**: Enviar evento de teste para produto específico
**Acesso**: Usuário autenticado

### 6. Status e Monitoramento

#### `GET /api/facebook/status`
**Descrição**: Verificar status da configuração do Facebook
**Acesso**: Usuário autenticado

**Response Success - Configurado**:
```json
{
  "status": "success",
  "data": {
    "connected": true,
    "pixel_id": "123456789012345",
    "test_event_code": "TEST12345",
    "connected_at": "2024-01-01T12:00:00Z",
    "status": "active",
    "message": "Facebook totalmente configurado e pronto para uso",
    "next_step": null
  }
}
```

**Response Success - Não Configurado**:
```json
{
  "status": "success",
  "data": {
    "connected": false,
    "message": "Facebook não conectado. Configure o pixel primeiro.",
    "next_step": "Configure o pixel usando POST /api/facebook/config/pixel"
  }
}
```

#### `GET /api/facebook/logs`
**Descrição**: Obter logs de eventos enviados
**Acesso**: Usuário autenticado

**Query Parameters**:
- `limit`: Número máximo de registros (padrão: 50)
- `offset`: Offset para paginação (padrão: 0)

**Response Success**:
```json
{
  "success": true,
  "logs": [
    {
      "id": "log-123",
      "event_name": "Purchase",
      "pixel_id": "123456789012345",
      "event_time": "2024-01-01T12:00:00Z",
      "user_data": {
        "em_hashed": true,
        "ph_hashed": true
      },
      "custom_data": {
        "value": 299.90,
        "currency": "BRL"
      },
      "status": "success",
      "facebook_response": {
        "events_received": 1,
        "fbtrace_id": "AXj8K..."
      },
      "created_at": "2024-01-01T12:00:05Z"
    }
  ],
  "total": 1
}
```

### 7. Validação de Token

#### `POST /api/facebook/validate-token`
**Descrição**: Validar Access Token do Facebook
**Acesso**: Usuário autenticado

```javascript
// Payload de exemplo
{
  "access_token": "EAAx..."
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Token válido",
  "token_info": {
    "id": "123456789",
    "name": "João Silva"
  }
}
```

**Response Error**:
```json
{
  "success": false,
  "error": "Token inválido ou expirado",
  "details": {
    "error": {
      "message": "Invalid OAuth access token.",
      "type": "OAuthException",
      "code": 190
    }
  }
}
```

## Integração Frontend

### Configuração Inicial
```jsx
import React, { useState, useEffect } from 'react';

const FacebookPixelConfig = ({ token }) => {
  const [config, setConfig] = useState({
    pixel_id: '',
    access_token: '',
    test_event_code: ''
  });
  const [status, setStatus] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    checkStatus();
  }, []);
  
  const checkStatus = async () => {
    try {
      const response = await fetch('/api/facebook/status', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setStatus(data.data);
    } catch (error) {
      console.error('Erro ao verificar status:', error);
    }
  };
  
  const configurePixel = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/facebook/config/pixel', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(config)
      });
      
      const data = await response.json();
      if (data.status === 'success') {
        if (data.data.needs_oauth) {
          // Redirecionar para OAuth
          window.location.href = '/api/facebook/auth/facebook';
        } else {
          alert('Pixel configurado com sucesso!');
          checkStatus();
        }
      }
    } catch (error) {
      alert('Erro ao configurar pixel: ' + error.message);
    } finally {
      setLoading(false);
    }
  };
  
  const testEvent = async () => {
    try {
      const response = await fetch('/api/facebook/test-event', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          pixelId: config.pixel_id,
          eventName: 'Purchase',
          userData: {
            em: 'teste@exemplo.com',
            fn: 'João',
            ln: 'Teste'
          },
          customData: {
            value: 99.90,
            currency: 'BRL'
          }
        })
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Evento de teste enviado com sucesso!');
      }
    } catch (error) {
      alert('Erro no evento de teste: ' + error.message);
    }
  };
  
  return (
    <div className="facebook-pixel-config">
      <h2>🎯 Configuração Facebook Pixel</h2>
      
      {status && (
        <div className={`status-card ${status.connected ? 'connected' : 'disconnected'}`}>
          <h3>Status: {status.connected ? '✅ Conectado' : '❌ Não Conectado'}</h3>
          {status.pixel_id && <p>Pixel ID: {status.pixel_id}</p>}
          {status.message && <p>{status.message}</p>}
          {status.next_step && <p><strong>Próximo passo:</strong> {status.next_step}</p>}
        </div>
      )}
      
      <div className="config-form">
        <h4>Configuração</h4>
        
        <label>
          Pixel ID:
          <input
            type="text"
            value={config.pixel_id}
            onChange={(e) => setConfig({...config, pixel_id: e.target.value})}
            placeholder="123456789012345"
          />
        </label>
        
        <label>
          Access Token (opcional):
          <input
            type="text"
            value={config.access_token}
            onChange={(e) => setConfig({...config, access_token: e.target.value})}
            placeholder="EAAx..."
          />
          <small>Se não fornecido, será necessário fazer OAuth</small>
        </label>
        
        <label>
          Test Event Code (opcional):
          <input
            type="text"
            value={config.test_event_code}
            onChange={(e) => setConfig({...config, test_event_code: e.target.value})}
            placeholder="TEST12345"
          />
          <small>Para testar eventos no Event Manager</small>
        </label>
        
        <div className="actions">
          <button 
            onClick={configurePixel}
            disabled={loading || !config.pixel_id}
            className="btn btn-primary"
          >
            {loading ? 'Configurando...' : '⚙️ Configurar Pixel'}
          </button>
          
          {status?.connected && (
            <button 
              onClick={testEvent}
              className="btn btn-secondary"
            >
              🧪 Enviar Evento Teste
            </button>
          )}
        </div>
      </div>
    </div>
  );
};
```

### Rastreamento de Eventos
```jsx
// Hook para rastreamento de eventos
const useFacebookTracking = () => {
  const trackEvent = async (eventName, eventData = {}) => {
    try {
      const response = await fetch('/api/facebook/track', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          pixel_id: window.FB_PIXEL_ID, // Definido globalmente
          event_name: eventName,
          event_time: Math.floor(Date.now() / 1000),
          user_data: {
            em: eventData.email,
            ph: eventData.phone,
            fn: eventData.firstName,
            ln: eventData.lastName
          },
          custom_data: eventData.customData || {}
        })
      });
      
      const result = await response.json();
      console.log('Facebook event tracked:', result);
      return result;
    } catch (error) {
      console.error('Error tracking Facebook event:', error);
    }
  };
  
  return { trackEvent };
};

// Componente de checkout com tracking
const CheckoutPage = ({ product, user }) => {
  const { trackEvent } = useFacebookTracking();
  
  useEffect(() => {
    // Track InitiateCheckout quando a página carrega
    trackEvent('InitiateCheckout', {
      email: user.email,
      firstName: user.firstName,
      lastName: user.lastName,
      customData: {
        value: product.price,
        currency: 'BRL',
        content_ids: [product.id],
        content_type: 'product'
      }
    });
  }, []);
  
  const handlePurchase = async () => {
    // Processar compra...
    
    // Track Purchase após compra bem-sucedida
    await trackEvent('Purchase', {
      email: user.email,
      firstName: user.firstName,
      lastName: user.lastName,
      customData: {
        value: product.price,
        currency: 'BRL',
        content_ids: [product.id],
        content_type: 'product',
        num_items: 1
      }
    });
  };
  
  return (
    <div className="checkout-page">
      {/* Checkout UI */}
    </div>
  );
};
```

### Configuração por Produto
```jsx
const ProductPixelConfig = ({ productId, token }) => {
  const [configs, setConfigs] = useState([]);
  const [newConfig, setNewConfig] = useState({
    pixel_id: '',
    access_token: '',
    test_event_code: '',
    name: ''
  });
  
  useEffect(() => {
    fetchConfigs();
  }, []);
  
  const fetchConfigs = async () => {
    try {
      const response = await fetch(`/api/facebook/products/${productId}/configs`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setConfigs(data.data.configs);
    } catch (error) {
      console.error('Erro ao buscar configurações:', error);
    }
  };
  
  const addConfig = async () => {
    try {
      const response = await fetch(`/api/facebook/products/${productId}/config`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(newConfig)
      });
      
      const data = await response.json();
      if (data.status === 'success') {
        alert('Configuração adicionada com sucesso!');
        fetchConfigs();
        setNewConfig({ pixel_id: '', access_token: '', test_event_code: '', name: '' });
      }
    } catch (error) {
      alert('Erro ao adicionar configuração: ' + error.message);
    }
  };
  
  const removeConfig = async (pixelId) => {
    if (!confirm('Tem certeza que deseja remover esta configuração?')) return;
    
    try {
      const response = await fetch(`/api/facebook/products/${productId}/configs/${pixelId}`, {
        method: 'DELETE',
        headers: { 'Authorization': `Bearer ${token}` }
      });
      
      const data = await response.json();
      if (data.status === 'success') {
        alert('Configuração removida com sucesso!');
        fetchConfigs();
      }
    } catch (error) {
      alert('Erro ao remover configuração: ' + error.message);
    }
  };
  
  return (
    <div className="product-pixel-config">
      <h3>📊 Pixels do Produto</h3>
      
      {/* Lista de configurações existentes */}
      <div className="existing-configs">
        {configs.map(config => (
          <div key={config.id} className="config-card">
            <h4>{config.name || config.pixel_id}</h4>
            <p>Pixel ID: {config.pixel_id}</p>
            <p>Status: {config.status}</p>
            <p>Criado: {new Date(config.created_at).toLocaleDateString()}</p>
            <button 
              onClick={() => removeConfig(config.pixel_id)}
              className="btn btn-danger"
            >
              🗑️ Remover
            </button>
          </div>
        ))}
      </div>
      
      {/* Formulário para nova configuração */}
      <div className="new-config-form">
        <h4>➕ Adicionar Novo Pixel</h4>
        
        <input
          type="text"
          placeholder="Nome da configuração"
          value={newConfig.name}
          onChange={(e) => setNewConfig({...newConfig, name: e.target.value})}
        />
        
        <input
          type="text"
          placeholder="Pixel ID"
          value={newConfig.pixel_id}
          onChange={(e) => setNewConfig({...newConfig, pixel_id: e.target.value})}
        />
        
        <input
          type="text"
          placeholder="Access Token"
          value={newConfig.access_token}
          onChange={(e) => setNewConfig({...newConfig, access_token: e.target.value})}
        />
        
        <input
          type="text"
          placeholder="Test Event Code"
          value={newConfig.test_event_code}
          onChange={(e) => setNewConfig({...newConfig, test_event_code: e.target.value})}
        />
        
        <button 
          onClick={addConfig}
          disabled={!newConfig.pixel_id || !newConfig.access_token}
          className="btn btn-primary"
        >
          ➕ Adicionar Configuração
        </button>
      </div>
    </div>
  );
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Parâmetros obrigatórios ausentes |
| 401 | Token de autenticação inválido |
| 404 | Configuração não encontrada |
| 500 | Erro interno ou falha na API do Facebook |

## Considerações de Segurança

1. **Hash de Dados**: Emails e telefones são hasheados (SHA-256)
2. **OAuth Seguro**: Fluxo OAuth 2.0 padrão do Facebook
3. **Validação de Token**: Tokens são validados antes do uso
4. **Rate Limiting**: Proteção contra uso excessivo
5. **Logs Seguros**: Dados sensíveis não são logados

## Melhores Práticas

1. **Test Mode**: Use test_event_code em desenvolvimento
2. **Deduplicação**: Use event_id único para evitar duplicatas
3. **Timing**: Envie eventos imediatamente após ação
4. **Dados Qualificados**: Forneça máximo de user_data possível
5. **Monitoramento**: Acompanhe logs para detectar problemas

Este sistema oferece integração completa com Facebook Ads, permitindo rastreamento preciso de conversões e otimização avançada de campanhas publicitárias.
