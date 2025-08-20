# Documentação - Sistema de Backup (routes/backup.js)

## Visão Geral
Sistema completo de backup e restauração que permite aos administradores criar, gerenciar e restaurar backups de diferentes partes do sistema com opções de compressão e criptografia.

## Características Principais
- **Tipos de Backup**: Full, parcial por tabelas ou módulos específicos
- **Compressão**: Suporte a compressão para reduzir tamanho
- **Criptografia**: Opção de criptografar backups sensíveis
- **Restauração**: Sistema seguro de restauração com confirmação
- **Download**: Download direto de arquivos de backup
- **Monitoramento**: Status e progresso em tempo real

## Tipos de Backup Suportados

| Tipo | Descrição | Tabelas Incluídas | Tamanho Estimado |
|------|-----------|-------------------|------------------|
| `full` | Backup completo | users, products, orders, courses, wallet_transactions, notifications | 50-150MB |
| `users` | Apenas usuários | users | 5-15MB |
| `products` | Produtos e cursos | products, courses | 10-30MB |
| `orders` | Pedidos e transações | orders, wallet_transactions | 15-45MB |
| `database` | Base completa | Todas as tabelas | 100-300MB |

## Rotas Implementadas

### 1. Criação de Backup

#### `POST /api/backup/create`
**Descrição**: Cria novo backup do sistema
**Acesso**: Admin

```javascript
// Payload de exemplo
{
  "type": "full", // full, users, products, orders, database
  "compression": true, // Opcional, padrão: true
  "encryption": false // Opcional, padrão: false
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Backup iniciado com sucesso",
  "backup": {
    "id": "backup_1753234567890",
    "type": "full",
    "status": "creating",
    "estimatedSize": 87654321,
    "estimatedTime": "2-5 minutos",
    "includedTables": [
      "users",
      "products", 
      "orders",
      "courses",
      "wallet_transactions",
      "notifications"
    ],
    "compression": true,
    "encryption": false,
    "createdAt": "2024-01-01T10:00:00Z",
    "createdBy": {
      "id": "user-123",
      "name": "Admin User",
      "email": "admin@exemplo.com"
    }
  }
}
```

**Processo de Backup**:
1. Validação de permissões (apenas admin)
2. Validação do tipo de backup
3. Criação do diretório de backups se necessário
4. Geração do nome do arquivo
5. Início do processo assíncrono
6. Retorno imediato com ID de tracking

### 2. Listagem de Backups

#### `GET /api/backup/list`
**Descrição**: Lista backups disponíveis com filtros e paginação
**Acesso**: Admin

**Query Parameters**:
- `page`: Página (padrão: 1)
- `limit`: Itens por página (padrão: 10)
- `type`: Filtro por tipo (full, users, products, orders, database)
- `status`: Filtro por status (creating, completed, failed, expired)

**Response Success**:
```json
{
  "success": true,
  "backups": [
    {
      "id": "backup_1753234567890",
      "type": "full",
      "status": "completed",
      "size": 87654321,
      "sizeFormatted": "83.6 MB",
      "compression": true,
      "encryption": false,
      "createdAt": "2024-01-01T10:00:00Z",
      "completedAt": "2024-01-01T10:03:45Z",
      "error": null,
      "metadata": {
        "tables": [
          "users",
          "products",
          "orders",
          "courses",
          "wallet_transactions"
        ],
        "records": 15420,
        "version": "1.0.0"
      },
      "typeLabel": "Completo",
      "statusLabel": "Concluído",
      "canDownload": true,
      "canRestore": true,
      "canDelete": true
    },
    {
      "id": "backup_1753134567890",
      "type": "users",
      "status": "completed",
      "size": 12345678,
      "sizeFormatted": "11.8 MB",
      "compression": true,
      "encryption": true,
      "createdAt": "2024-01-01T08:00:00Z",
      "completedAt": "2024-01-01T08:01:30Z",
      "metadata": {
        "tables": ["users"],
        "records": 1240,
        "version": "1.0.0"
      },
      "typeLabel": "Usuários",
      "statusLabel": "Concluído",
      "canDownload": true,
      "canRestore": true,
      "canDelete": true
    },
    {
      "id": "backup_1753034567890",
      "type": "products",
      "status": "failed",
      "size": 0,
      "sizeFormatted": "0 Bytes",
      "compression": true,
      "encryption": false,
      "createdAt": "2024-01-01T06:00:00Z",
      "error": "Falha na conexão com o banco de dados",
      "typeLabel": "Produtos",
      "statusLabel": "Falhou",
      "canDownload": false,
      "canRestore": false,
      "canDelete": true
    }
  ],
  "stats": {
    "total": 15,
    "completed": 12,
    "failed": 2,
    "creating": 1,
    "totalSize": 1234567890,
    "totalSizeFormatted": "1.15 GB"
  },
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 15,
    "totalPages": 2
  }
}
```

**Status de Backup**:
- `creating`: Backup sendo criado
- `completed`: Backup concluído com sucesso
- `failed`: Falha na criação do backup
- `expired`: Backup expirado (pode ser removido automaticamente)

### 3. Restauração de Backup

#### `POST /api/backup/restore/:backupId`
**Descrição**: Restaura backup específico (OPERAÇÃO CRÍTICA)
**Acesso**: Admin

```javascript
// Payload de exemplo
{
  "confirmAction": true // OBRIGATÓRIO - confirma que entende as consequências
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Processo de restauração iniciado",
  "restore": {
    "id": "restore_1753234567890",
    "backupId": "backup_1753234567890",
    "status": "restoring",
    "estimatedTime": "5-15 minutos",
    "startedAt": "2024-01-01T11:00:00Z",
    "startedBy": {
      "id": "user-123",
      "name": "Admin User",
      "email": "admin@exemplo.com"
    },
    "backup": {
      "type": "full",
      "size": "83.6 MB",
      "createdAt": "2024-01-01T10:00:00Z"
    },
    "warnings": [
      "Todos os dados atuais serão sobrescritos",
      "O sistema pode ficar indisponível durante a restauração",
      "Faça logout de todos os usuários antes de prosseguir"
    ]
  }
}
```

**⚠️ Importantes Considerações de Segurança**:
- Requer confirmação explícita (`confirmAction: true`)
- Sobrescreve TODOS os dados atuais
- Sistema pode ficar indisponível durante restauração
- Apenas administradores podem executar
- Processo irreversível

### 4. Download de Backup

#### `GET /api/backup/download/:backupId`
**Descrição**: Gera link para download do arquivo de backup
**Acesso**: Admin

**Response Success**:
```json
{
  "success": true,
  "message": "Link de download gerado com sucesso",
  "download": {
    "backupId": "backup_1753234567890",
    "fileName": "backup_1753234567890_full.sql.gz",
    "size": "83.6 MB",
    "expiresIn": "1 hora",
    "downloadUrl": "https://api.exemplo.com/api/backup/file/backup_1753234567890",
    "createdAt": "2024-01-01T11:05:00Z"
  },
  "note": "Em ambiente de produção, este seria um link direto para download do arquivo"
}
```

**Características do Download**:
- Link temporário com expiração
- Arquivo comprimido se configurado
- Criptografado se configurado
- Log de downloads para auditoria

### 5. Remoção de Backup

#### `DELETE /api/backup/:backupId`
**Descrição**: Remove backup do sistema
**Acesso**: Admin

**Response Success**:
```json
{
  "success": true,
  "message": "Backup deletado com sucesso",
  "deletedBackup": {
    "id": "backup_1753234567890",
    "deletedAt": "2024-01-01T11:10:00Z",
    "deletedBy": {
      "id": "user-123",
      "name": "Admin User",
      "email": "admin@exemplo.com"
    }
  }
}
```

## Funcionalidades Avançadas

### 1. Configuração de Compressão
```javascript
{
  "compression": {
    "enabled": true,
    "algorithm": "gzip", // gzip, bzip2, xz
    "level": 6, // 1-9 (velocidade vs tamanho)
    "estimated_reduction": "60-80%"
  }
}
```

### 2. Configuração de Criptografia
```javascript
{
  "encryption": {
    "enabled": true,
    "algorithm": "AES-256-GCM",
    "key_source": "environment", // environment, vault, manual
    "salt": "auto_generated"
  }
}
```

### 3. Backup Incremental (Futuro)
```javascript
{
  "type": "incremental",
  "base_backup_id": "backup_1753234567890",
  "changes_since": "2024-01-01T10:00:00Z",
  "estimated_size": "5-15MB"
}
```

### 4. Agendamento Automático (Futuro)
```javascript
{
  "schedule": {
    "enabled": true,
    "frequency": "daily", // daily, weekly, monthly
    "time": "02:00",
    "timezone": "America/Sao_Paulo",
    "retention": {
      "daily": 7,
      "weekly": 4,
      "monthly": 12
    }
  }
}
```

## Sistema de Monitoramento

### 1. Status em Tempo Real
```javascript
// WebSocket ou polling para acompanhar progresso
{
  "backupId": "backup_1753234567890",
  "status": "creating",
  "progress": {
    "percentage": 65,
    "current_table": "orders",
    "processed_records": 8500,
    "total_records": 15420,
    "elapsed_time": "00:02:30",
    "estimated_remaining": "00:01:15"
  }
}
```

### 2. Logs Detalhados
```javascript
{
  "logs": [
    {
      "timestamp": "2024-01-01T10:00:00Z",
      "level": "info",
      "message": "Backup iniciado",
      "details": { "type": "full", "tables": 6 }
    },
    {
      "timestamp": "2024-01-01T10:00:15Z",
      "level": "info", 
      "message": "Processando tabela users",
      "details": { "records": 1240 }
    },
    {
      "timestamp": "2024-01-01T10:01:30Z",
      "level": "warning",
      "message": "Tabela grande detectada: orders",
      "details": { "records": 8500, "estimated_time": "2min" }
    }
  ]
}
```

## Integração Frontend

### React Component Example
```jsx
import React, { useState, useEffect } from 'react';

const BackupManager = ({ token }) => {
  const [backups, setBackups] = useState([]);
  const [stats, setStats] = useState(null);
  const [creating, setCreating] = useState(false);
  
  useEffect(() => {
    fetchBackups();
  }, []);
  
  const fetchBackups = async () => {
    const response = await fetch('/api/backup/list', {
      headers: { 'Authorization': `Bearer ${token}` }
    });
    const data = await response.json();
    setBackups(data.backups);
    setStats(data.stats);
  };
  
  const createBackup = async (type) => {
    setCreating(true);
    try {
      const response = await fetch('/api/backup/create', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          type,
          compression: true,
          encryption: type === 'full' // Criptografar apenas backups completos
        })
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Backup iniciado com sucesso!');
        fetchBackups();
      }
    } catch (error) {
      alert('Erro ao criar backup: ' + error.message);
    } finally {
      setCreating(false);
    }
  };
  
  const restoreBackup = async (backupId) => {
    const confirmed = window.confirm(
      '⚠️ ATENÇÃO: Esta ação irá sobrescrever TODOS os dados atuais do sistema. ' +
      'Esta operação é IRREVERSÍVEL. Tem certeza que deseja continuar?'
    );
    
    if (!confirmed) return;
    
    const doubleConfirmed = window.prompt(
      'Digite "CONFIRMAR RESTAURAÇÃO" para prosseguir:'
    );
    
    if (doubleConfirmed !== 'CONFIRMAR RESTAURAÇÃO') {
      alert('Restauração cancelada.');
      return;
    }
    
    try {
      const response = await fetch(`/api/backup/restore/${backupId}`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({ confirmAction: true })
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Restauração iniciada! O sistema pode ficar indisponível por alguns minutos.');
      }
    } catch (error) {
      alert('Erro ao restaurar backup: ' + error.message);
    }
  };
  
  const downloadBackup = async (backupId) => {
    try {
      const response = await fetch(`/api/backup/download/${backupId}`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      
      if (data.success) {
        window.open(data.download.downloadUrl, '_blank');
      }
    } catch (error) {
      alert('Erro ao baixar backup: ' + error.message);
    }
  };
  
  return (
    <div className="backup-manager">
      <h2>Gerenciamento de Backups</h2>
      
      {/* Estatísticas */}
      <div className="backup-stats">
        <div className="stat">
          <h3>{stats?.total}</h3>
          <p>Total de Backups</p>
        </div>
        <div className="stat">
          <h3>{stats?.completed}</h3>
          <p>Concluídos</p>
        </div>
        <div className="stat">
          <h3>{stats?.totalSizeFormatted}</h3>
          <p>Espaço Total</p>
        </div>
      </div>
      
      {/* Criação de Backups */}
      <div className="backup-creation">
        <h3>Criar Novo Backup</h3>
        <div className="backup-types">
          {['full', 'users', 'products', 'orders'].map(type => (
            <button
              key={type}
              onClick={() => createBackup(type)}
              disabled={creating}
              className={`backup-type-btn ${type}`}
            >
              {creating ? 'Criando...' : `Backup ${type}`}
            </button>
          ))}
        </div>
      </div>
      
      {/* Lista de Backups */}
      <div className="backups-list">
        <h3>Backups Existentes</h3>
        <div className="backups-table">
          {backups.map(backup => (
            <div key={backup.id} className="backup-row">
              <div className="backup-info">
                <h4>{backup.typeLabel}</h4>
                <p>ID: {backup.id}</p>
                <p>Tamanho: {backup.sizeFormatted}</p>
                <p>Criado: {new Date(backup.createdAt).toLocaleString()}</p>
              </div>
              
              <div className={`backup-status ${backup.status}`}>
                {backup.statusLabel}
              </div>
              
              <div className="backup-actions">
                {backup.canDownload && (
                  <button onClick={() => downloadBackup(backup.id)}>
                    📥 Download
                  </button>
                )}
                {backup.canRestore && (
                  <button 
                    onClick={() => restoreBackup(backup.id)}
                    className="danger"
                  >
                    🔄 Restaurar
                  </button>
                )}
                {backup.canDelete && (
                  <button 
                    onClick={() => deleteBackup(backup.id)}
                    className="danger"
                  >
                    🗑️ Deletar
                  </button>
                )}
              </div>
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
| 400 | Tipo de backup inválido ou confirmação ausente |
| 401 | Token de autenticação inválido |
| 403 | Acesso negado (apenas administradores) |
| 404 | Backup não encontrado |
| 500 | Erro interno no processo de backup/restauração |

## Considerações de Segurança

1. **Acesso Restrito**: Apenas administradores
2. **Confirmação Dupla**: Restaurações requerem confirmação explícita
3. **Criptografia**: Opção para backups sensíveis
4. **Audit Log**: Todas as operações são logadas
5. **Links Temporários**: Downloads com expiração

## Monitoramento e Alertas

### Métricas Importantes
- Tempo de criação de backup
- Taxa de sucesso/falha
- Tamanho médio dos backups
- Frequência de uso
- Espaço em disco utilizado

### Alertas Recomendados
- Falha na criação de backup
- Espaço em disco baixo
- Backup não executado em X dias
- Tentativa de restauração crítica

## Melhores Práticas

1. **Agendamento Regular**: Backups automáticos diários
2. **Rotação de Backups**: Remover backups antigos automaticamente
3. **Teste de Restauração**: Testar restaurações periodicamente
4. **Monitoramento**: Acompanhar métricas de backup
5. **Documentação**: Manter procedimentos de recuperação documentados

Este sistema de backup oferece uma solução robusta e segura para proteção de dados, com funcionalidades avançadas de compressão, criptografia e monitoramento.
