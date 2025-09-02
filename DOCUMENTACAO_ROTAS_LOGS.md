# Documentação - Sistema de Logs (routes/logs.js)

## Visão Geral
Sistema completo de logs para monitoramento, auditoria e diagnóstico do sistema. Permite consultar, filtrar, categorizar e gerenciar logs com diferentes níveis de criticidade.

## Características Principais
- **Níveis de Log**: Debug, Info, Warning, Error, Critical
- **Categorização**: Logs organizados por módulos e funcionalidades
- **Filtragem Avançada**: Por data, usuário, categoria e nível
- **Limpeza Automática**: Remoção de logs antigos
- **Logs Manuais**: Criação de logs personalizados
- **Estatísticas**: Dashboard com métricas de logs

## Níveis de Log

| Nível | Cor | Descrição | Quando Usar |
|-------|-----|-----------|-------------|
| `debug` | #6c757d (Cinza) | Informações detalhadas para debugging | Desenvolvimento e troubleshooting |
| `info` | #17a2b8 (Azul) | Informações gerais sobre operações | Operações normais do sistema |
| `warn` | #ffc107 (Amarelo) | Situações que requerem atenção | Problemas potenciais |
| `error` | #dc3545 (Vermelho) | Erros que não impedem o funcionamento | Falhas recuperáveis |
| `critical` | #6f42c1 (Roxo) | Erros críticos do sistema | Falhas graves que podem afetar o sistema |

## Categorias de Log

| Categoria | Descrição | Exemplos |
|-----------|-----------|----------|
| `USER_MANAGEMENT` | Gestão de usuários | Login, logout, criação de conta |
| `PRODUCT_MANAGEMENT` | Gestão de produtos | Criação, edição, exclusão de produtos |
| `ORDER_PROCESSING` | Processamento de pedidos | Criação, pagamento, entrega |
| `PAYMENT_PROCESSING` | Processamento de pagamentos | PIX, cartão, webhooks |
| `SUBSCRIPTION_MANAGEMENT` | Gestão de assinaturas | Renovação, cancelamento |
| `PIXEL_TRACKING` | Rastreamento de pixels | Facebook, Google, eventos |
| `FUNNEL_MANAGEMENT` | Gestão de funis | Upsell, downsell, recuperação |
| `SHIPPING_MANAGEMENT` | Gestão de envios | Produtos físicos, rastreamento |
| `PANEL_MANAGEMENT` | Gestão do painel | Permissões, configurações |
| `SYSTEM_MAINTENANCE` | Manutenção do sistema | Backups, limpeza, otimização |
| `SECURITY` | Segurança | Tentativas de login, acessos suspeitos |
| `API_ACCESS` | Acesso à API | Chamadas externas, webhooks |
| `ERROR_TRACKING` | Rastreamento de erros | Exceptions, falhas de sistema |

## Rotas Implementadas

### 1. Consulta de Logs

#### `GET /api/logs`
**Descrição**: Buscar logs do sistema com filtros avançados
**Acesso**: Admin

**Query Parameters**:
- `level`: Nível do log (debug, info, warn, error, critical)
- `category`: Categoria do log
- `userId`: ID do usuário associado
- `startDate`: Data inicial (ISO 8601)
- `endDate`: Data final (ISO 8601)
- `limit`: Número máximo de logs (padrão: 100)

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "log-12345",
      "timestamp": "2024-01-01T10:30:00Z",
      "level": "info",
      "message": "Usuário realizou login com sucesso",
      "category": "USER_MANAGEMENT",
      "userId": "user-123",
      "metadata": {
        "email": "usuario@exemplo.com",
        "ip": "192.168.1.100",
        "userAgent": "Mozilla/5.0...",
        "sessionId": "sess_abc123"
      },
      "context": {
        "request": {
          "method": "POST",
          "url": "/api/auth/login",
          "headers": {
            "content-type": "application/json"
          }
        },
        "response": {
          "status": 200,
          "duration": 245
        }
      }
    },
    {
      "id": "log-12346",
      "timestamp": "2024-01-01T10:25:00Z",
      "level": "error",
      "message": "Falha no processamento de pagamento",
      "category": "PAYMENT_PROCESSING",
      "userId": "user-456",
      "metadata": {
        "orderId": "order-789",
        "paymentMethod": "credit_card",
        "amount": 99.90,
        "error": "Card declined",
        "acquirer": "stripe"
      },
      "context": {
        "stack": "Error: Card declined\n    at payment.process...",
        "request": {
          "paymentData": {
            "cardLastFour": "1234",
            "cardBrand": "visa"
          }
        }
      }
    },
    {
      "id": "log-12347",
      "timestamp": "2024-01-01T10:20:00Z",
      "level": "warn",
      "message": "Tentativa de acesso a produto inexistente",
      "category": "PRODUCT_MANAGEMENT",
      "userId": "user-789",
      "metadata": {
        "productId": "product-999",
        "action": "view_product",
        "referrer": "https://google.com"
      },
      "context": {
        "request": {
          "url": "/api/products/product-999",
          "method": "GET"
        },
        "response": {
          "status": 404
        }
      }
    }
  ],
  "total": 3,
  "filters": {
    "level": null,
    "category": null,
    "userId": null,
    "startDate": "2024-01-01T00:00:00Z",
    "endDate": "2024-01-01T23:59:59Z",
    "limit": 100
  }
}
```

### 2. Criação de Log Manual

#### `POST /api/logs`
**Descrição**: Criar log manual (útil para debugging e auditoria)
**Acesso**: Admin

```javascript
// Payload de exemplo
{
  "level": "info",
  "message": "Manutenção programada iniciada",
  "category": "SYSTEM_MAINTENANCE",
  "metadata": {
    "maintenanceType": "database_optimization",
    "estimatedDuration": "30 minutes",
    "affectedServices": ["api", "dashboard"]
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Log criado com sucesso",
  "log": {
    "id": "log-manual-123",
    "timestamp": "2024-01-01T11:00:00Z",
    "level": "info",
    "message": "Manutenção programada iniciada",
    "category": "SYSTEM_MAINTENANCE",
    "userId": "admin-123",
    "metadata": {
      "maintenanceType": "database_optimization",
      "estimatedDuration": "30 minutes",
      "affectedServices": ["api", "dashboard"],
      "manualEntry": true
    }
  }
}
```

### 3. Categorias de Log

#### `GET /api/logs/categories`
**Descrição**: Listar todas as categorias de log disponíveis
**Acesso**: Admin

**Response Success**:
```json
{
  "success": true,
  "categories": [
    {
      "value": "USER_MANAGEMENT",
      "label": "Gestão de Usuários",
      "description": "Logs relacionados ao gerenciamento de usuários",
      "examples": [
        "Login e logout",
        "Criação de contas",
        "Alteração de perfil",
        "Recuperação de senha"
      ]
    },
    {
      "value": "PRODUCT_MANAGEMENT",
      "label": "Gestão de Produtos",
      "description": "Logs relacionados ao gerenciamento de produtos",
      "examples": [
        "Criação de produtos",
        "Alteração de preços",
        "Publicação/despublicação",
        "Gestão de estoque"
      ]
    },
    {
      "value": "ORDER_PROCESSING",
      "label": "Processamento de Pedidos",
      "description": "Logs relacionados ao processamento de pedidos",
      "examples": [
        "Criação de pedidos",
        "Processamento de pagamentos",
        "Alteração de status",
        "Cancelamentos"
      ]
    },
    {
      "value": "PAYMENT_PROCESSING",
      "label": "Processamento de Pagamentos",
      "description": "Logs relacionados aos pagamentos",
      "examples": [
        "Transações PIX",
        "Pagamentos com cartão",
        "Webhooks de pagamento",
        "Estornos"
      ]
    },
    {
      "value": "SECURITY",
      "label": "Segurança",
      "description": "Logs relacionados à segurança",
      "examples": [
        "Tentativas de login falharam",
        "Acessos suspeitos",
        "Alterações de permissões",
        "Bloqueios de conta"
      ]
    }
  ]
}
```

### 4. Níveis de Log

#### `GET /api/logs/levels`
**Descrição**: Listar todos os níveis de log disponíveis
**Acesso**: Admin

**Response Success**:
```json
{
  "success": true,
  "levels": [
    {
      "value": "debug",
      "label": "Debug",
      "color": "#6c757d",
      "description": "Informações detalhadas para debugging",
      "icon": "🔍",
      "priority": 1
    },
    {
      "value": "info",
      "label": "Info",
      "color": "#17a2b8",
      "description": "Informações gerais sobre operações",
      "icon": "ℹ️",
      "priority": 2
    },
    {
      "value": "warn",
      "label": "Warning",
      "color": "#ffc107",
      "description": "Situações que requerem atenção",
      "icon": "⚠️",
      "priority": 3
    },
    {
      "value": "error",
      "label": "Error",
      "color": "#dc3545",
      "description": "Erros que não impedem o funcionamento",
      "icon": "❌",
      "priority": 4
    },
    {
      "value": "critical",
      "label": "Critical",
      "color": "#6f42c1",
      "description": "Erros críticos do sistema",
      "icon": "🚨",
      "priority": 5
    }
  ]
}
```

### 5. Limpeza de Logs

#### `DELETE /api/logs/cleanup`
**Descrição**: Remover logs antigos para otimizar performance
**Acesso**: Admin

**Query Parameters**:
- `days`: Dias a manter (mínimo: 7, padrão: 30)

**Response Success**:
```json
{
  "success": true,
  "message": "1,234 logs antigos removidos",
  "deletedCount": 1234,
  "cleanup": {
    "daysToKeep": 30,
    "cutoffDate": "2023-12-01T00:00:00Z",
    "totalLogsBeforeCleanup": 5678,
    "totalLogsAfterCleanup": 4444,
    "spaceFreed": "15.2 MB",
    "executedAt": "2024-01-01T12:00:00Z",
    "executedBy": {
      "id": "admin-123",
      "name": "Admin User",
      "email": "admin@exemplo.com"
    }
  }
}
```

### 6. Estatísticas de Logs

#### `GET /api/logs/stats`
**Descrição**: Estatísticas e métricas dos logs
**Acesso**: Admin

**Response Success**:
```json
{
  "success": true,
  "data": {
    "totalLogs": 45678,
    "logsByLevel": {
      "debug": 15234,
      "info": 25123,
      "warn": 3456,
      "error": 1654,
      "critical": 211
    },
    "logsByCategory": {
      "USER_MANAGEMENT": 12345,
      "ORDER_PROCESSING": 8765,
      "PAYMENT_PROCESSING": 6543,
      "PRODUCT_MANAGEMENT": 4321,
      "SECURITY": 987,
      "SYSTEM_MAINTENANCE": 543,
      "API_ACCESS": 2134,
      "ERROR_TRACKING": 1654,
      "PIXEL_TRACKING": 3456,
      "FUNNEL_MANAGEMENT": 2345,
      "SUBSCRIPTION_MANAGEMENT": 1876,
      "SHIPPING_MANAGEMENT": 876,
      "PANEL_MANAGEMENT": 433
    },
    "trendsLast7Days": {
      "2024-01-07": { "total": 1234, "errors": 45, "warnings": 123 },
      "2024-01-06": { "total": 1156, "errors": 38, "warnings": 110 },
      "2024-01-05": { "total": 1298, "errors": 52, "warnings": 134 },
      "2024-01-04": { "total": 1087, "errors": 33, "warnings": 98 },
      "2024-01-03": { "total": 1445, "errors": 61, "warnings": 156 },
      "2024-01-02": { "total": 1321, "errors": 47, "warnings": 128 },
      "2024-01-01": { "total": 1398, "errors": 55, "warnings": 142 }
    },
    "topErrors": [
      {
        "message": "Falha na conexão com o banco de dados",
        "count": 23,
        "lastOccurrence": "2024-01-01T11:45:00Z",
        "category": "SYSTEM_MAINTENANCE"
      },
      {
        "message": "Cartão de crédito recusado",
        "count": 18,
        "lastOccurrence": "2024-01-01T11:30:00Z",
        "category": "PAYMENT_PROCESSING"
      }
    ],
    "recentActivity": [
      {
        "id": "log-latest-1",
        "timestamp": "2024-01-01T12:00:00Z",
        "level": "info",
        "message": "Backup automático concluído",
        "category": "SYSTEM_MAINTENANCE"
      },
      {
        "id": "log-latest-2",
        "timestamp": "2024-01-01T11:58:00Z",
        "level": "info",
        "message": "Pedido processado com sucesso",
        "category": "ORDER_PROCESSING"
      }
    ],
    "systemHealth": {
      "errorRate": 3.6,
      "warningRate": 7.5,
      "avgResponseTime": 245,
      "status": "healthy"
    }
  }
}
```

## Integração Frontend

### React Component Example
```jsx
import React, { useState, useEffect } from 'react';

const LogsManager = ({ token }) => {
  const [logs, setLogs] = useState([]);
  const [filters, setFilters] = useState({
    level: '',
    category: '',
    startDate: '',
    endDate: '',
    limit: 100
  });
  const [categories, setCategories] = useState([]);
  const [levels, setLevels] = useState([]);
  const [stats, setStats] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    fetchCategories();
    fetchLevels();
    fetchStats();
    fetchLogs();
  }, []);
  
  const fetchLogs = async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams();
      Object.entries(filters).forEach(([key, value]) => {
        if (value) params.append(key, value);
      });
      
      const response = await fetch(`/api/logs?${params}`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setLogs(data.data || []);
    } catch (error) {
      console.error('Erro ao buscar logs:', error);
    } finally {
      setLoading(false);
    }
  };
  
  const fetchCategories = async () => {
    try {
      const response = await fetch('/api/logs/categories', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setCategories(data.categories || []);
    } catch (error) {
      console.error('Erro ao buscar categorias:', error);
    }
  };
  
  const fetchLevels = async () => {
    try {
      const response = await fetch('/api/logs/levels', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setLevels(data.levels || []);
    } catch (error) {
      console.error('Erro ao buscar níveis:', error);
    }
  };
  
  const fetchStats = async () => {
    try {
      const response = await fetch('/api/logs/stats', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setStats(data.data);
    } catch (error) {
      console.error('Erro ao buscar estatísticas:', error);
    }
  };
  
  const createManualLog = async (logData) => {
    try {
      const response = await fetch('/api/logs', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(logData)
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Log criado com sucesso!');
        fetchLogs();
      }
    } catch (error) {
      alert('Erro ao criar log: ' + error.message);
    }
  };
  
  const cleanupLogs = async (days = 30) => {
    const confirmed = window.confirm(
      `Tem certeza que deseja remover logs com mais de ${days} dias? Esta ação é irreversível.`
    );
    
    if (!confirmed) return;
    
    try {
      const response = await fetch(`/api/logs/cleanup?days=${days}`, {
        method: 'DELETE',
        headers: { 'Authorization': `Bearer ${token}` }
      });
      
      const data = await response.json();
      if (data.success) {
        alert(`${data.deletedCount} logs removidos com sucesso!`);
        fetchStats();
        fetchLogs();
      }
    } catch (error) {
      alert('Erro na limpeza: ' + error.message);
    }
  };
  
  const getLevelBadge = (level) => {
    const levelConfig = levels.find(l => l.value === level);
    if (!levelConfig) return <span>{level}</span>;
    
    return (
      <span 
        className="level-badge"
        style={{ 
          backgroundColor: levelConfig.color,
          color: 'white',
          padding: '2px 8px',
          borderRadius: '4px',
          fontSize: '12px'
        }}
      >
        {levelConfig.icon} {levelConfig.label}
      </span>
    );
  };
  
  return (
    <div className="logs-manager">
      <h2>Sistema de Logs</h2>
      
      {/* Estatísticas */}
      {stats && (
        <div className="logs-stats">
          <h3>Estatísticas</h3>
          <div className="stats-grid">
            <div className="stat-card">
              <h4>{stats.totalLogs.toLocaleString()}</h4>
              <p>Total de Logs</p>
            </div>
            <div className="stat-card">
              <h4>{stats.systemHealth.errorRate}%</h4>
              <p>Taxa de Erro</p>
            </div>
            <div className="stat-card">
              <h4>{stats.systemHealth.avgResponseTime}ms</h4>
              <p>Tempo Médio</p>
            </div>
            <div className="stat-card">
              <h4>{stats.logsByLevel.error + stats.logsByLevel.critical}</h4>
              <p>Logs Críticos</p>
            </div>
          </div>
          
          <div className="logs-by-level">
            <h4>Logs por Nível</h4>
            {Object.entries(stats.logsByLevel).map(([level, count]) => (
              <div key={level} className="level-stat">
                {getLevelBadge(level)}
                <span>{count.toLocaleString()}</span>
              </div>
            ))}
          </div>
        </div>
      )}
      
      {/* Filtros */}
      <div className="logs-filters">
        <h3>Filtros</h3>
        <div className="filters-grid">
          <select 
            value={filters.level}
            onChange={(e) => setFilters({...filters, level: e.target.value})}
          >
            <option value="">Todos os níveis</option>
            {levels.map(level => (
              <option key={level.value} value={level.value}>
                {level.label}
              </option>
            ))}
          </select>
          
          <select 
            value={filters.category}
            onChange={(e) => setFilters({...filters, category: e.target.value})}
          >
            <option value="">Todas as categorias</option>
            {categories.map(category => (
              <option key={category.value} value={category.value}>
                {category.label}
              </option>
            ))}
          </select>
          
          <input
            type="datetime-local"
            value={filters.startDate}
            onChange={(e) => setFilters({...filters, startDate: e.target.value})}
            placeholder="Data inicial"
          />
          
          <input
            type="datetime-local"
            value={filters.endDate}
            onChange={(e) => setFilters({...filters, endDate: e.target.value})}
            placeholder="Data final"
          />
          
          <input
            type="number"
            value={filters.limit}
            onChange={(e) => setFilters({...filters, limit: e.target.value})}
            placeholder="Limite"
            min="1"
            max="1000"
          />
          
          <button onClick={fetchLogs} disabled={loading}>
            {loading ? 'Carregando...' : 'Buscar'}
          </button>
        </div>
      </div>
      
      {/* Ações */}
      <div className="logs-actions">
        <button onClick={() => cleanupLogs(30)}>
          🗑️ Limpar Logs (30 dias)
        </button>
        <button onClick={() => cleanupLogs(7)}>
          🗑️ Limpar Logs (7 dias)
        </button>
      </div>
      
      {/* Lista de Logs */}
      <div className="logs-list">
        <h3>Logs ({logs.length})</h3>
        <div className="logs-table">
          {logs.map(log => (
            <div key={log.id} className="log-entry">
              <div className="log-header">
                <span className="log-timestamp">
                  {new Date(log.timestamp).toLocaleString()}
                </span>
                {getLevelBadge(log.level)}
                <span className="log-category">{log.category}</span>
              </div>
              
              <div className="log-message">{log.message}</div>
              
              {log.metadata && (
                <details className="log-metadata">
                  <summary>Metadados</summary>
                  <pre>{JSON.stringify(log.metadata, null, 2)}</pre>
                </details>
              )}
              
              {log.context && (
                <details className="log-context">
                  <summary>Contexto</summary>
                  <pre>{JSON.stringify(log.context, null, 2)}</pre>
                </details>
              )}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Parâmetros inválidos ou campos obrigatórios ausentes |
| 401 | Token de autenticação inválido |
| 403 | Acesso negado (apenas administradores podem acessar logs) |
| 500 | Erro interno do servidor |

## Considerações de Segurança

1. **Acesso Restrito**: Apenas administradores com permissão específica
2. **Sanitização**: Dados sensíveis removidos ou mascarados
3. **Auditoria**: Todas as operações de log são logadas
4. **Retenção**: Logs antigos são removidos automaticamente
5. **Criptografia**: Logs sensíveis podem ser criptografados

## Melhores Práticas

1. **Filtragem Inteligente**: Use filtros específicos para reduzir volume
2. **Limpeza Regular**: Configure limpeza automática
3. **Monitoramento Ativo**: Configure alertas para logs críticos
4. **Contexto Adequado**: Inclua metadados relevantes
5. **Performance**: Limite o número de logs retornados

Este sistema de logs oferece uma solução completa para monitoramento, auditoria e diagnóstico, com funcionalidades avançadas de filtragem, categorização e análise estatística.

# 📊 Sistema de Logs - CheckoutPro Backend

## 🎯 Visão Geral

O sistema de logs do CheckoutPro Backend é uma solução completa para monitoramento, auditoria e debugging que captura automaticamente todas as atividades do sistema e fornece uma interface administrativa para análise.

## 🏗️ Arquitetura

### 📁 Estrutura de Arquivos
```
cproback/
├── middleware/
│   └── logging.js          # Middleware principal e serviços de log
├── logs/                   # Diretório de logs (criado automaticamente)
│   ├── requests-2024-09-02.log
│   ├── responses-2024-09-02.log
│   ├── errors-2024-09-02.log
│   ├── performance-2024-09-02.log
│   ├── auth-2024-09-02.log
│   ├── payments-2024-09-02.log
│   ├── analytics-2024-09-02.log
│   ├── events-2024-09-02.log
│   └── general-2024-09-02.log
└── routes/
    └── admin.js            # Rotas administrativas para logs
```

### 🔧 Componentes

1. **Middleware de Logging** (`middleware/logging.js`)
   - Captura automática de todas as requisições
   - Interceptação de respostas
   - Detecção de erros e performance
   - Sanitização de dados sensíveis

2. **Rotas Administrativas** (`routes/admin.js`)
   - Interface para visualização de logs
   - Busca e filtros avançados
   - Estatísticas e relatórios
   - Limpeza automática

## 📝 Tipos de Logs

### 1. 🌐 Logs de Requisições (`requests-YYYY-MM-DD.log`)
Captura todas as requisições HTTP recebidas:
```json
{
  "timestamp": "2024-09-02T15:30:45.123Z",
  "type": "REQUEST_START",
  "requestId": "abc123def456",
  "method": "POST",
  "url": "/api/auth/login",
  "originalUrl": "/api/auth/login",
  "path": "/api/auth/login",
  "query": {},
  "headers": {
    "user-agent": "Mozilla/5.0...",
    "content-type": "application/json",
    "authorization": "Bearer ***"
  },
  "ip": "192.168.1.100",
  "body": {
    "email": "user@example.com",
    "password": "***"
  },
  "startTime": "2024-09-02T15:30:45.123Z"
}
```

### 2. 📤 Logs de Respostas (`responses-YYYY-MM-DD.log`)
Captura todas as respostas enviadas:
```json
{
  "timestamp": "2024-09-02T15:30:45.156Z",
  "type": "REQUEST_END",
  "request": { /* dados da requisição */ },
  "response": {
    "requestId": "abc123def456",
    "statusCode": 200,
    "statusMessage": "OK",
    "duration": "33ms",
    "responseSize": 1024,
    "responseBody": {
      "success": true,
      "token": "***"
    },
    "endTime": "2024-09-02T15:30:45.156Z"
  }
}
```

### 3. ❌ Logs de Erros (`errors-YYYY-MM-DD.log`)
Captura erros HTTP (4xx, 5xx) e exceções:
```json
{
  "timestamp": "2024-09-02T15:31:00.789Z",
  "type": "HTTP_ERROR",
  "requestId": "def456ghi789",
  "method": "GET",
  "url": "/api/products/999999",
  "statusCode": 404,
  "duration": "15ms",
  "error": {
    "success": false,
    "error": "Produto não encontrado"
  },
  "userAgent": "Mozilla/5.0...",
  "ip": "192.168.1.100"
}
```

### 4. ⚡ Logs de Performance (`performance-YYYY-MM-DD.log`)
Captura requisições lentas (>2 segundos):
```json
{
  "timestamp": "2024-09-02T15:32:10.456Z",
  "type": "SLOW_REQUEST",
  "requestId": "ghi789jkl012",
  "method": "POST",
  "url": "/api/orders",
  "duration": "3247ms",
  "statusCode": 200
}
```

### 5. 🔐 Logs de Autenticação (`auth-YYYY-MM-DD.log`)
Eventos de login, logout, registro:
```json
{
  "timestamp": "2024-09-02T15:33:20.789Z",
  "type": "AUTH_EVENT",
  "action": "LOGIN_SUCCESS",
  "userId": 123,
  "email": "user@example.com",
  "ip": "192.168.1.100",
  "userAgent": "Mozilla/5.0..."
}
```

### 6. 💳 Logs de Pagamentos (`payments-YYYY-MM-DD.log`)
Eventos de transações e pagamentos:
```json
{
  "timestamp": "2024-09-02T15:34:30.012Z",
  "type": "PAYMENT_EVENT",
  "action": "PAYMENT_APPROVED",
  "orderId": "ORD-2024-001",
  "amount": 99.90,
  "method": "PIX",
  "acquirer": "pagarme"
}
```

### 7. 📈 Logs de Analytics (`analytics-YYYY-MM-DD.log`)
Eventos de rastreamento e analytics:
```json
{
  "timestamp": "2024-09-02T15:35:40.345Z",
  "type": "ANALYTICS_EVENT",
  "event": "course_enrollment",
  "courseId": 456,
  "studentId": 789,
  "source": "checkout_page"
}
```

### 8. 🎯 Logs de Eventos (`events-YYYY-MM-DD.log`)
Eventos personalizados do sistema:
```json
{
  "timestamp": "2024-09-02T15:36:50.678Z",
  "type": "USER_REGISTRATION",
  "userId": 999,
  "userType": "seller",
  "registrationSource": "website"
}
```

### 9. 📋 Log Geral (`general-YYYY-MM-DD.log`)
Todos os logs combinados para análise geral.

## 🚀 Implementação

### 1. Middleware Automático
O middleware é aplicado automaticamente a todas as rotas:

```javascript
// server.js
import { requestLogger } from './middleware/logging.js';

// Aplicar logging a todas as requisições
app.use(requestLogger);
```

### 2. Logging Personalizado
Para eventos específicos, use as funções do serviço:

```javascript
import { logEvent, logError, logAuth, logPayment, logAnalytics } from '../middleware/logging.js';

// Log de evento personalizado
logEvent('USER_ACTION', {
  userId: 123,
  action: 'profile_update',
  changes: ['email', 'phone']
});

// Log de erro
logError(new Error('Falha na conexão'), {
  context: 'database_connection',
  retryAttempt: 3
});

// Log de autenticação
logAuth('LOGIN_ATTEMPT', 123, {
  email: 'user@example.com',
  success: false,
  reason: 'invalid_password'
});

// Log de pagamento
logPayment('PAYMENT_CREATED', 'ORD-001', {
  amount: 99.90,
  method: 'credit_card',
  status: 'pending'
});

// Log de analytics
logAnalytics('page_view', {
  page: '/checkout/produto-123',
  userId: 456,
  sessionId: 'sess_789'
});
```

## 📊 API Administrativa

### 🔐 Autenticação
Todas as rotas de logs requerem autenticação de administrador:
```
Authorization: Bearer <admin_token>
```

### 📋 Listar Arquivos de Logs
```http
GET /api/admin/logs
```

**Resposta:**
```json
{
  "success": true,
  "message": "Lista de arquivos de logs recuperada com sucesso",
  "data": {
    "totalFiles": 15,
    "files": [
      {
        "filename": "requests-2024-09-02.log",
        "type": "requests",
        "date": "2024-09-02",
        "size": 2048576,
        "created": "2024-09-02T00:00:00.000Z",
        "modified": "2024-09-02T23:59:59.999Z"
      }
    ]
  }
}
```

### 📊 Estatísticas dos Logs
```http
GET /api/admin/logs/stats?date=2024-09-02
```

**Resposta:**
```json
{
  "success": true,
  "data": {
    "date": "2024-09-02",
    "totalFiles": 8,
    "totalSize": 15728640,
    "logTypes": {
      "requests": { "files": 1, "size": 8388608 },
      "errors": { "files": 1, "size": 1048576 },
      "performance": { "files": 1, "size": 524288 }
    },
    "requestCount": 1542,
    "errorCount": 23,
    "avgResponseTime": 0
  }
}
```

### 📖 Ler Arquivo de Log
```http
GET /api/admin/logs/requests-2024-09-02.log?type=REQUEST_START&limit=100
```

**Parâmetros de Query:**
- `type` - Filtrar por tipo de log
- `startTime` - Data/hora inicial (ISO)
- `endTime` - Data/hora final (ISO)
- `limit` - Número máximo de registros (padrão: 1000)

### 🚨 Erros do Dia Atual
```http
GET /api/admin/logs/today/errors
```

### 📋 Requisições do Dia Atual
```http
GET /api/admin/logs/today/requests?limit=100
```

### ⚡ Dados de Performance
```http
GET /api/admin/logs/performance?date=2024-09-02
```

### 🔍 Busca Personalizada
```http
POST /api/admin/logs/search
Content-Type: application/json

{
  "date": "2024-09-02",
  "logType": "requests",
  "searchTerm": "login",
  "startTime": "2024-09-02T10:00:00.000Z",
  "endTime": "2024-09-02T18:00:00.000Z",
  "limit": 500
}
```

### 🧹 Limpeza de Logs Antigos
```http
DELETE /api/admin/logs/cleanup
Content-Type: application/json

{
  "daysToKeep": 30
}
```

## 🔒 Segurança e Privacidade

### 🛡️ Sanitização Automática
O sistema automaticamente remove ou oculta dados sensíveis:

- **Senhas**: Substituídas por `***`
- **Tokens**: Substituídos por `***`
- **Números de cartão**: Substituídos por `***`
- **CPF/CNPJ**: Substituídos por `***`
- **Chaves API**: Substituídas por `***`

### 📏 Limitação de Tamanho
- Respostas grandes (>5KB) são truncadas no log
- Body das requisições é limitado
- Headers são filtrados (apenas os essenciais)

### 🔐 Controle de Acesso
- Apenas administradores podem acessar logs
- Autenticação obrigatória com token JWT
- Logs de acesso aos próprios logs

## 🔄 Rotação e Manutenção

### 📅 Rotação Diária
- Novos arquivos criados automaticamente a cada dia
- Formato: `tipo-YYYY-MM-DD.log`
- Sem interrupção do serviço

### 🧹 Limpeza Automática
- Logs antigos removidos automaticamente
- Configurável (padrão: 30 dias)
- Limpeza manual via API administrativa

### 💾 Backup
Recomendações para backup:
```bash
# Backup diário dos logs
tar -czf logs-backup-$(date +%Y%m%d).tar.gz logs/

# Sincronização com storage externo
rsync -av logs/ backup-server:/backup/checkoutpro/logs/
```

## 📈 Monitoramento e Alertas

### 🚨 Alertas Recomendados
1. **Taxa de Erro Alta**: >5% de requisições com erro
2. **Performance Degradada**: >10% de requisições lentas
3. **Volume Anormal**: Picos de tráfego inesperados
4. **Falhas de Autenticação**: Múltiplas tentativas falhadas
5. **Espaço em Disco**: Logs ocupando >80% do espaço

### 📊 Métricas Importantes
- Requisições por segundo (RPS)
- Tempo médio de resposta
- Taxa de sucesso/erro
- Distribuição de status codes
- Endpoints mais utilizados
- Usuários mais ativos

## 🧪 Teste do Sistema

Execute o script de teste para verificar todas as funcionalidades:

```bash
# No diretório cproback
node test-logs-system.js
```

O script testa:
- ✅ Login administrativo
- ✅ Geração de logs de exemplo
- ✅ Listagem de arquivos
- ✅ Leitura de logs
- ✅ Estatísticas
- ✅ Busca personalizada
- ✅ Filtros por tipo e data
- ✅ Logs de erro
- ✅ Performance

## 🛠️ Troubleshooting

### ❌ Problemas Comuns

1. **Logs não sendo criados**
   - Verificar permissões do diretório `logs/`
   - Verificar espaço em disco disponível
   - Confirmar que o middleware está carregado

2. **Erro de acesso às rotas administrativas**
   - Verificar token de autenticação
   - Confirmar role de administrador
   - Verificar se o usuário existe

3. **Performance degradada**
   - Logs consomem poucos recursos
   - Verificar rotação automática
   - Limpar logs antigos se necessário

### 🔧 Debug
Para debug, ative logs detalhados:
```javascript
// No início do server.js
process.env.DEBUG_LOGGING = 'true';
```

## 📋 Checklist de Implementação

- [x] ✅ Middleware de logging implementado
- [x] ✅ Rotas administrativas criadas
- [x] ✅ Sanitização de dados sensíveis
- [x] ✅ Rotação diária automática
- [x] ✅ Sistema de busca e filtros
- [x] ✅ Estatísticas e relatórios
- [x] ✅ Limpeza automática de logs antigos
- [x] ✅ Documentação completa
- [x] ✅ Script de teste
- [x] ✅ Integração com servidor principal

## 🎯 Próximos Passos

1. **Dashboard Web**: Interface gráfica para visualização
2. **Alertas Automáticos**: Notificações por email/Slack
3. **Integração Analytics**: Envio para ferramentas externas
4. **Machine Learning**: Detecção de anomalias
5. **Relatórios Automáticos**: Relatórios diários/semanais

---

**🔧 Sistema implementado por:** CheckoutPro Backend Team  
**📅 Data:** Setembro 2024  
**🔄 Versão:** 1.0.0
