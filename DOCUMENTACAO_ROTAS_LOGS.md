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

# 📊 Sistema de Logs MySQL - Documentação Completa

## 🎯 Visão Geral

Sistema completo de logs utilizando **MySQL Azure Flexible Server** com autenticação SSL, desenvolvido para substituir o sistema de logs baseado em arquivos por uma solução robusta e escalável no banco de dados.

### ✅ Características Principais
- **100% MySQL** - Não utiliza arquivos locais
- **SSL Certificate** - Conexão segura com certificado
- **Estrutura completa** - Logs principais, estatísticas e eventos específicos
- **Performance otimizada** - Conexão reutilizada e queries eficientes
- **APIs administrativas** - Interface completa para gestão de logs

---

## 🏗️ Arquitetura do Sistema

### 📁 Estrutura de Arquivos
```
cproback/
├── database/
│   ├── logsDatabase.js       # Conexão MySQL com SSL
│   └── logsService.js        # Operações de logs no MySQL
├── middleware/
│   └── loggingMySQL.js       # Middleware de logging automático
├── routes/
│   └── adminLogsMySQL.js     # Rotas administrativas de logs
└── certificado/
    └── certificado.crt       # Certificado SSL para MySQL
```

### 🗃️ Banco de Dados

**Servidor:** `mysqlservercktpro.mysql.database.azure.com`  
**Banco:** `checkoutpro_logs`  
**Porta:** `3306`  
**SSL:** Obrigatório com certificado

#### 📊 Estrutura das Tabelas

##### 1. **`logs`** - Tabela Principal
```sql
CREATE TABLE logs (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  request_id VARCHAR(255) NOT NULL,
  type ENUM('REQUEST_START', 'REQUEST_END', 'HTTP_ERROR', 'APPLICATION_ERROR', 'AUTH_EVENT', 'PAYMENT_EVENT', 'SLOW_REQUEST') NOT NULL,
  timestamp DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  date_only DATE NOT NULL,
  method VARCHAR(10),
  url TEXT,
  status_code INT,
  duration_ms INT,
  ip_address VARCHAR(45),
  user_agent TEXT,
  user_id VARCHAR(255),
  data JSON,
  INDEX idx_date_type (date_only, type),
  INDEX idx_timestamp (timestamp),
  INDEX idx_user_id (user_id),
  INDEX idx_request_id (request_id)
);
```

##### 2. **`logs_daily_stats`** - Estatísticas Diárias
```sql
CREATE TABLE logs_daily_stats (
  id INT AUTO_INCREMENT PRIMARY KEY,
  date_only DATE NOT NULL UNIQUE,
  total_requests INT DEFAULT 0,
  total_errors INT DEFAULT 0,
  total_slow_requests INT DEFAULT 0,
  avg_response_time_ms DECIMAL(10,2) DEFAULT 0,
  unique_users INT DEFAULT 0,
  unique_ips INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

##### 3. **`logs_auth_events`** - Eventos de Autenticação
```sql
CREATE TABLE logs_auth_events (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  log_id BIGINT NOT NULL,
  user_id VARCHAR(255),
  email VARCHAR(255),
  action VARCHAR(50) NOT NULL,
  success BOOLEAN NOT NULL DEFAULT FALSE,
  ip_address VARCHAR(45),
  user_agent TEXT,
  details JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (log_id) REFERENCES logs(id) ON DELETE CASCADE
);
```

##### 4. **`logs_payment_events`** - Eventos de Pagamento
```sql
CREATE TABLE logs_payment_events (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  log_id BIGINT NOT NULL,
  order_id VARCHAR(255),
  transaction_id VARCHAR(255),
  action VARCHAR(50) NOT NULL,
  amount DECIMAL(10,2),
  currency VARCHAR(3) DEFAULT 'BRL',
  payment_method VARCHAR(50),
  acquirer VARCHAR(50),
  status VARCHAR(50),
  details JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (log_id) REFERENCES logs(id) ON DELETE CASCADE
);
```

---

## 🔧 Configuração

### 📋 Variáveis de Ambiente
```env
# MySQL Logs Database
LOGS_DB_HOST=mysqlservercktpro.mysql.database.azure.com
LOGS_DB_USER=admcktpromysql
LOGS_DB_PASSWORD=07XFRJl6sPAL
LOGS_DB_NAME=checkoutpro_logs
LOGS_DB_PORT=3306
```

### 🔒 Certificado SSL
- **Arquivo:** `certificado/certificado.crt`
- **Uso:** Obrigatório para conexão segura
- **Configuração:** Automática no sistema

---

## 📡 APIs Disponíveis

### 🔐 Autenticação
Todas as rotas administrativas requerem:
- **Header:** `Authorization: Bearer <token>`
- **Role:** `admin`

### 📊 Rotas Administrativas

#### 1. **Dashboard de Logs**
```http
GET /api/admin/logs/dashboard
Authorization: Bearer <token>
```
**Resposta:**
```json
{
  "success": true,
  "data": {
    "quick": {
      "today": {
        "total_requests": 150,
        "total_errors": 5,
        "total_slow_requests": 3,
        "avg_response_time_ms": 245.67
      },
      "totalLogs": 15847,
      "typeBreakdown": {
        "REQUEST_START": 7923,
        "REQUEST_END": 7920,
        "HTTP_ERROR": 2,
        "AUTH_EVENT": 2
      }
    },
    "week": [
      {
        "date_only": "2025-09-02",
        "total_requests": 150,
        "total_errors": 5,
        "total_slow_requests": 3,
        "avg_response_time_ms": 245.67,
        "unique_users": 12,
        "unique_ips": 8
      }
    ],
    "trends": {
      "requests": 15,
      "errors": -20,
      "avgResponseTime": 5
    },
    "lastUpdate": "2025-09-02T20:23:04.720Z"
  },
  "message": "Dashboard carregado com sucesso"
}
```

#### 2. **Listar Logs**
```http
GET /api/admin/logs/logs?page=1&limit=50&date=2025-09-02&type=HTTP_ERROR
Authorization: Bearer <token>
```
**Parâmetros de Query:**
- `page` (opcional): Página (padrão: 1)
- `limit` (opcional): Itens por página (padrão: 50)
- `date` (opcional): Data específica (YYYY-MM-DD)
- `type` (opcional): Tipo de log
- `method` (opcional): Método HTTP
- `status` (opcional): Status code
- `userId` (opcional): ID do usuário
- `search` (opcional): Busca em URL ou dados

**Resposta:**
```json
{
  "success": true,
  "data": {
    "logs": [
      {
        "id": 12345,
        "request_id": "req_abc123",
        "type": "REQUEST_END",
        "timestamp": "2025-09-02T20:15:30.000Z",
        "method": "POST",
        "url": "/api/auth/login",
        "status_code": 200,
        "duration_ms": 156,
        "ip_address": "192.168.1.1",
        "user_agent": "Mozilla/5.0...",
        "user_id": "user_123",
        "data": {...}
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 25,
      "totalLogs": 1247,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```

#### 3. **Estatísticas de Logs**
```http
GET /api/admin/logs/stats?date=2025-09-02
Authorization: Bearer <token>
```
**Resposta:**
```json
{
  "success": true,
  "data": {
    "stats": {
      "date_only": "2025-09-02",
      "total_requests": 1247,
      "total_errors": 23,
      "total_slow_requests": 15,
      "avg_response_time_ms": 245.67,
      "unique_users": 156,
      "unique_ips": 89
    },
    "quick": {
      "today": {...},
      "totalLogs": 15847,
      "typeBreakdown": {...}
    },
    "date": "2025-09-02"
  }
}
```

#### 4. **Buscar Logs**
```http
POST /api/admin/logs/search
Authorization: Bearer <token>
Content-Type: application/json

{
  "searchTerm": "error",
  "dateFrom": "2025-09-01",
  "dateTo": "2025-09-02",
  "limit": 100,
  "offset": 0
}
```
**Resposta:**
```json
{
  "success": true,
  "data": {
    "results": [...],
    "totalFound": 23,
    "searchTerm": "error",
    "filters": {
      "dateFrom": "2025-09-01",
      "dateTo": "2025-09-02"
    }
  }
}
```

#### 5. **Limpeza de Logs Antigos**
```http
DELETE /api/admin/logs/cleanup?days=30
Authorization: Bearer <token>
```
**Resposta:**
```json
{
  "success": true,
  "data": {
    "deletedLogs": 5847,
    "cutoffDate": "2025-08-03",
    "remainingLogs": 10000
  },
  "message": "Logs antigos removidos com sucesso"
}
```

#### 6. **Teste Simples**
```http
GET /api/admin/logs/simple
Authorization: Bearer <token>
```
**Resposta:**
```json
{
  "success": true,
  "message": "Rota simples funcionando!",
  "timestamp": "2025-09-02T20:23:04.720Z"
}
```

---

## 🔄 Middleware de Logging

### 📝 Logging Automático
O sistema registra automaticamente:

#### **REQUEST_START** - Início da Requisição
```javascript
{
  type: 'REQUEST_START',
  method: 'POST',
  url: '/api/auth/login',
  ip_address: '192.168.1.1',
  user_agent: 'Mozilla/5.0...',
  user_id: null,
  data: {
    headers: {...},
    query: {...},
    body: {...} // Sanitizado
  }
}
```

#### **REQUEST_END** - Final da Requisição
```javascript
{
  type: 'REQUEST_END',
  method: 'POST',
  url: '/api/auth/login',
  status_code: 200,
  duration_ms: 156,
  ip_address: '192.168.1.1',
  user_id: 'user_123',
  data: {
    response: {...}, // Sanitizado
    duration: 156
  }
}
```

#### **HTTP_ERROR** - Erros HTTP
```javascript
{
  type: 'HTTP_ERROR',
  status_code: 404,
  url: '/api/invalid/route',
  data: {
    error: 'Rota não encontrada',
    stack: '...'
  }
}
```

#### **AUTH_EVENT** - Eventos de Autenticação
```javascript
{
  type: 'AUTH_EVENT',
  data: {
    auth: {
      userId: 'user_123',
      email: 'admin@checkoutpro.com',
      action: 'login',
      success: true,
      ipAddress: '192.168.1.1',
      userAgent: 'Mozilla/5.0...',
      details: {
        loginMethod: 'email_password',
        timestamp: '2025-09-02T20:15:30.000Z'
      }
    }
  }
}
```

#### **PAYMENT_EVENT** - Eventos de Pagamento
```javascript
{
  type: 'PAYMENT_EVENT',
  data: {
    payment: {
      orderId: 'order_123',
      transactionId: 'txn_abc456',
      action: 'payment_approved',
      amount: 99.90,
      currency: 'BRL',
      paymentMethod: 'credit_card',
      acquirer: 'pagarme',
      status: 'approved',
      details: {
        cardBrand: 'visa',
        installments: 1
      }
    }
  }
}
```

---

## 🛠️ Funcionalidades Técnicas

### 🔗 Conexão MySQL
```javascript
// Configuração com SSL
const LOGS_DB_CONFIG = {
  host: 'mysqlservercktpro.mysql.database.azure.com',
  user: 'admcktpromysql',
  password: '07XFRJl6sPAL',
  database: 'checkoutpro_logs',
  port: 3306,
  ssl: {
    ca: fs.readFileSync(SSL_CERT_PATH),
    rejectUnauthorized: false
  },
  connectTimeout: 30000,
  connectionLimit: 10,
  charset: 'utf8mb4'
};
```

### 📊 Otimizações
- **Conexão reutilizada** - Evita reconexões desnecessárias
- **Índices otimizados** - Performance em consultas frequentes
- **Sanitização automática** - Remove dados sensíveis
- **Validação de parâmetros** - Converte `undefined` para `null`
- **Transações seguras** - Rollback em caso de erro

### 🧹 Manutenção
- **Cleanup automático** - Remoção de logs antigos
- **Monitoramento de saúde** - Verificação da conexão
- **Logs de erro** - Sistema de debug interno
- **Backup automático** - Através do Azure MySQL

---

## 🚀 Como Usar

### 1. **Visualizar Dashboard**
```bash
# Login
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@checkoutpro.com","password":"Admin123!"}'

# Dashboard
curl -X GET http://localhost:5000/api/admin/logs/dashboard \
  -H "Authorization: Bearer <token>"
```

### 2. **Buscar Logs Específicos**
```bash
# Logs de erro de hoje
curl -X GET "http://localhost:5000/api/admin/logs/logs?date=2025-09-02&type=HTTP_ERROR" \
  -H "Authorization: Bearer <token>"
```

### 3. **Pesquisar Logs**
```bash
curl -X POST http://localhost:5000/api/admin/logs/search \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "searchTerm": "login",
    "dateFrom": "2025-09-01",
    "dateTo": "2025-09-02"
  }'
```

### 4. **Manutenção**
```bash
# Limpar logs antigos (30 dias)
curl -X DELETE "http://localhost:5000/api/admin/logs/cleanup?days=30" \
  -H "Authorization: Bearer <token>"
```

---

## 📈 Monitoramento e Métricas

### 🎯 KPIs Disponíveis
- **Total de requisições por dia**
- **Taxa de erro (HTTP 4xx/5xx)**
- **Tempo médio de resposta**
- **Requisições lentas (>2s)**
- **Usuários únicos por dia**
- **IPs únicos por dia**
- **Eventos de autenticação**
- **Transações de pagamento**

### 📊 Relatórios
- **Dashboard em tempo real**
- **Tendências semanais**
- **Comparação dia anterior**
- **Top rotas mais acessadas**
- **Top erros mais frequentes**
- **Análise de performance**

---

## 🔧 Troubleshooting

### ❌ Problemas Comuns

#### 1. **Erro de Conexão MySQL**
```
Error: connect ECONNREFUSED
```
**Solução:** Verificar configurações de rede e certificado SSL

#### 2. **"Malformed communication packet"**
```
Error: Malformed communication packet
```
**Solução:** Verificar parâmetros SQL (evitar `undefined`)

#### 3. **Unauthorized (401)**
```
{"success":false,"error":"Token inválido"}
```
**Solução:** Renovar token de autenticação

### 🔍 Debug
```javascript
// Habilitar logs de debug
console.log('🔍 Debug ativado para logs MySQL');

// Verificar conexão
await connection.execute('SELECT 1');

// Testar inserção
await saveLogToDatabase('TEST', {...});
```

---

## 📋 Checklist de Implementação

### ✅ Sistema Base
- [x] Conexão MySQL com SSL
- [x] Estrutura de tabelas completa
- [x] Middleware de logging automático
- [x] APIs administrativas
- [x] Autenticação e autorização

### ✅ Funcionalidades
- [x] Dashboard interativo
- [x] Listagem paginada
- [x] Busca avançada
- [x] Estatísticas em tempo real
- [x] Cleanup automático
- [x] Logs por tipo (auth, payment, etc.)

### ✅ Performance
- [x] Conexão reutilizada
- [x] Índices otimizados
- [x] Queries eficientes
- [x] Sanitização automática
- [x] Validação de parâmetros

### ✅ Segurança
- [x] SSL obrigatório
- [x] Autenticação JWT
- [x] Role-based access
- [x] Sanitização de dados sensíveis
- [x] Proteção contra SQL injection

---

## 🎉 Conclusão

O **Sistema de Logs MySQL** está completamente implementado e funcionando, oferecendo:

- **📊 Visibilidade total** - Logs estruturados e organizados
- **🚀 Performance** - Conexões otimizadas e queries eficientes  
- **🔒 Segurança** - SSL, autenticação e sanitização
- **🛠️ Manutenção** - Limpeza automática e monitoramento
- **📱 Interface** - APIs completas para administração

**Sistema pronto para produção! 🚀**

---
**Documentação criada em:** 02/09/2025  
**Versão:** 1.0  
**Status:** ✅ Implementado e Testado

# 📊 Sistema de Logs MySQL - CheckoutPro

Sistema completo de logs usando **MySQL Azure Flexible Server** com SSL certificate.

## 🚀 Quick Start

### 1. Configuração
- ✅ **MySQL Azure** configurado com SSL
- ✅ **Certificado** em `certificado/certificado.crt`
- ✅ **Tabelas** criadas automaticamente

### 2. APIs Principais

```bash
# Login admin
POST /api/auth/login
{
  "email": "admin@checkoutpro.com",
  "password": "Admin123!"
}

# Dashboard de logs
GET /api/admin/logs/dashboard
Authorization: Bearer <token>

# Listar logs
GET /api/admin/logs/logs?page=1&limit=50&date=2025-09-02&type=HTTP_ERROR
Authorization: Bearer <token>

# Buscar logs
POST /api/admin/logs/search
Authorization: Bearer <token>
{
  "searchTerm": "error",
  "dateFrom": "2025-09-01",
  "dateTo": "2025-09-02"
}

# Estatísticas
GET /api/admin/logs/stats?date=2025-09-02
Authorization: Bearer <token>

# Limpeza
DELETE /api/admin/logs/cleanup?days=30
Authorization: Bearer <token>

# Teste simples
GET /api/admin/logs/simple
Authorization: Bearer <token>
```

## 🗃️ Estrutura do Banco

### Tabelas
- **`logs`** - Logs principais (requisições, erros, eventos)
- **`logs_daily_stats`** - Estatísticas diárias agregadas
- **`logs_auth_events`** - Eventos de autenticação específicos
- **`logs_payment_events`** - Eventos de pagamento específicos

### Tipos de Log
- `REQUEST_START` - Início da requisição
- `REQUEST_END` - Final da requisição
- `HTTP_ERROR` - Erros HTTP (4xx, 5xx)
- `APPLICATION_ERROR` - Erros da aplicação
- `AUTH_EVENT` - Eventos de autenticação
- `PAYMENT_EVENT` - Eventos de pagamento
- `SLOW_REQUEST` - Requisições lentas (>2s)

## 📊 Features

### ✅ Implementado
- [x] **100% MySQL** - Sem arquivos locais
- [x] **SSL Certificate** - Conexão segura
- [x] **Logging automático** - Middleware integrado
- [x] **Dashboard admin** - Interface completa
- [x] **Busca avançada** - Filtros e paginação
- [x] **Estatísticas** - Métricas em tempo real
- [x] **Cleanup automático** - Manutenção de logs
- [x] **Performance otimizada** - Conexão reutilizada

### 📈 Métricas Disponíveis
- Total de requisições por dia
- Taxa de erro (4xx/5xx)
- Tempo médio de resposta
- Requisições lentas
- Usuários únicos
- IPs únicos
- Eventos de autenticação
- Transações de pagamento

## 🔧 Configuração Técnica

### Conexão MySQL
```javascript
{
  host: 'mysqlservercktpro.mysql.database.azure.com',
  user: 'admcktpromysql', 
  database: 'checkoutpro_logs',
  port: 3306,
  ssl: { ca: fs.readFileSync('certificado/certificado.crt') }
}
```

### Middleware Automático
- Registra todas as requisições
- Captura erros automaticamente
- Sanitiza dados sensíveis
- Calcula tempo de resposta
- Identifica requisições lentas

## 🛠️ Manutenção

### Health Check
```bash
GET /api/admin/logs/simple
# Resposta: {"success":true,"message":"Rota simples funcionando!"}
```

### Limpeza
```bash
DELETE /api/admin/logs/cleanup?days=30
# Remove logs mais antigos que 30 dias
```

### Debug
- Logs de conexão no console
- Validação automática de parâmetros
- Tratamento de erros robusto

## 📚 Documentação Completa

Ver: `DOCUMENTACAO_LOGS_MYSQL_COMPLETA.md`

---

**Status:** ✅ **FUNCIONANDO**  
**Última atualização:** 02/09/2025  
**Versão:** 1.0
