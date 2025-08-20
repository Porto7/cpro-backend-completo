# Documentação - Administração de Cursos (routes/courseAdmin.js)

## Visão Geral
Sistema de administração para instrutores e administradores gerenciarem cursos, matriculados, analytics detalhados e configurações avançadas. Fornece controle total sobre o ambiente educacional.

## Características Principais
- **Gestão de Matriculados**: Listar, conceder e revogar acessos
- **Analytics Detalhados**: Métricas completas de engajamento e performance
- **Controle de Acesso**: Permissões baseadas em instrutor ou admin
- **Configurações Avançadas**: Personalização completa do curso
- **Relatórios**: Insights de aprendizado e progressão

## Permissões de Acesso

| Ação | Instrutor | Admin | Usuário |
|------|-----------|-------|---------|
| Listar matriculados | ✅ (próprios cursos) | ✅ (todos) | ❌ |
| Conceder/revogar acesso | ✅ (próprios cursos) | ✅ (todos) | ❌ |
| Ver analytics | ✅ (próprios cursos) | ✅ (todos) | ❌ |
| Alterar configurações | ✅ (próprios cursos) | ✅ (todos) | ❌ |

## Rotas Implementadas

### 1. Gestão de Matriculados

#### `GET /api/admin/courses/:courseId/enrollments`
**Descrição**: Listar matriculados em um curso com filtros e estatísticas
**Acesso**: Instrutor (próprios cursos) ou Admin

**Query Parameters**:
- `page`: Página (padrão: 1)
- `limit`: Itens por página (padrão: 20)
- `status`: Filtro por status (active, expired, cancelled)
- `search`: Busca por nome ou email do estudante

**Response Success**:
```json
{
  "enrollments": [
    {
      "id": "enrollment-123",
      "enrolledAt": "2024-01-01T10:00:00Z",
      "expiresAt": "2024-12-31T23:59:59Z",
      "status": "active",
      "progressPercentage": 75,
      "timeSpentMinutes": 1440,
      "lastAccessedAt": "2024-01-15T14:30:00Z",
      "completedAt": null,
      "student": {
        "id": "user-123",
        "name": "João Silva",
        "email": "joao@exemplo.com",
        "avatar": "https://exemplo.com/avatars/joao.jpg",
        "memberSince": "2023-06-15T09:00:00Z"
      },
      "purchase": {
        "id": "order-456",
        "amount": 299.90,
        "paymentMethod": "credit_card",
        "status": "paid",
        "purchaseDate": "2024-01-01T10:00:00Z"
      },
      "progressSummary": {
        "totalLessons": 20,
        "completedLessons": 15
      }
    },
    {
      "id": "enrollment-124",
      "enrolledAt": "2024-01-02T11:00:00Z",
      "expiresAt": null,
      "status": "active",
      "progressPercentage": 100,
      "timeSpentMinutes": 2400,
      "lastAccessedAt": "2024-01-14T16:45:00Z",
      "completedAt": "2024-01-10T20:15:00Z",
      "student": {
        "id": "user-124",
        "name": "Maria Santos",
        "email": "maria@exemplo.com",
        "avatar": "https://exemplo.com/avatars/maria.jpg",
        "memberSince": "2023-03-20T14:30:00Z"
      },
      "purchase": {
        "id": "order-457",
        "amount": 299.90,
        "paymentMethod": "pix",
        "status": "paid",
        "purchaseDate": "2024-01-02T11:00:00Z"
      },
      "progressSummary": {
        "totalLessons": 20,
        "completedLessons": 20
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "totalPages": 8
  },
  "stats": {
    "totalEnrollments": 156,
    "activeEnrollments": 142,
    "expiredEnrollments": 8,
    "cancelledEnrollments": 6,
    "totalRevenue": 46784.40,
    "averageProgress": 68.5
  }
}
```

#### `POST /api/admin/courses/:courseId/enrollments`
**Descrição**: Conceder ou revogar acesso manual ao curso
**Acesso**: Instrutor (próprios cursos) ou Admin

```javascript
// Payload para conceder acesso
{
  "userId": "user-789",
  "action": "grant",
  "expirationDate": "2024-12-31T23:59:59Z", // Opcional
  "reason": "Acesso promocional concedido"
}

// Payload para revogar acesso
{
  "userId": "user-789",
  "action": "revoke",
  "reason": "Violação dos termos de uso"
}
```

**Response Success - Acesso Concedido**:
```json
{
  "success": true,
  "message": "Acesso concedido com sucesso",
  "enrollment": {
    "id": "enrollment-125",
    "status": "active",
    "enrolledAt": "2024-01-16T10:00:00Z",
    "expiresAt": "2024-12-31T23:59:59Z"
  }
}
```

**Response Success - Acesso Revogado**:
```json
{
  "success": true,
  "message": "Acesso revogado com sucesso"
}
```

**Response Error - Parâmetros Inválidos**:
```json
{
  "error": "Parâmetros inválidos. userId e action (grant/revoke) são obrigatórios",
  "type": "invalid_parameters"
}
```

**Response Error - Usuário Não Matriculado (revoke)**:
```json
{
  "error": "Usuário não está matriculado neste curso",
  "type": "not_enrolled"
}
```

### 2. Analytics Detalhados

#### `GET /api/admin/courses/:courseId/analytics`
**Descrição**: Analytics detalhado do curso com métricas de engajamento
**Acesso**: Instrutor (próprios cursos) ou Admin

**Query Parameters**:
- `period`: Período de análise (7d, 30d, 90d - padrão: 30d)

**Response Success**:
```json
{
  "course": {
    "id": "course-123",
    "title": "Desenvolvimento Web Completo",
    "instructor": "Prof. Maria Santos",
    "createdAt": "2023-12-01T00:00:00Z"
  },
  "period": "30d",
  "overview": {
    "totalEnrollments": 156,
    "activeEnrollments": 142,
    "completedEnrollments": 87,
    "completionRate": 56,
    "totalRevenue": 46784.40,
    "periodRevenue": 12450.60,
    "averageProgress": 68.5,
    "totalStudyTime": 145600
  },
  "charts": {
    "enrollmentsByDay": [
      {
        "date": "2024-01-01",
        "enrollments": 8
      },
      {
        "date": "2024-01-02",
        "enrollments": 12
      },
      {
        "date": "2024-01-03",
        "enrollments": 6
      }
    ],
    "topLessons": [
      {
        "lessonId": "lesson-456",
        "lessonTitle": "Introdução ao JavaScript",
        "moduleTitle": "Módulo 1 - Fundamentos",
        "views": 142,
        "avgProgress": 89.5,
        "completions": 128
      },
      {
        "lessonId": "lesson-457",
        "lessonTitle": "Variáveis e Tipos de Dados",
        "moduleTitle": "Módulo 1 - Fundamentos",
        "views": 138,
        "avgProgress": 92.1,
        "completions": 125
      },
      {
        "lessonId": "lesson-458",
        "lessonTitle": "Funções em JavaScript",
        "moduleTitle": "Módulo 2 - Programação",
        "views": 120,
        "avgProgress": 76.3,
        "completions": 98
      }
    ]
  },
  "insights": {
    "mostEngagingModule": "Módulo 1 - Fundamentos",
    "dropoffPoints": [
      {
        "lesson": "Programação Assíncrona",
        "dropoffRate": 15.2
      }
    ],
    "studyPatterns": "Alunos estudam mais nos fins de semana",
    "recommendations": [
      "Adicionar mais exercícios práticos no Módulo 3",
      "Revisar conteúdo sobre Programação Assíncrona",
      "Considerar quebrar aulas longas em partes menores"
    ]
  }
}
```

### 3. Configurações Avançadas

#### `PUT /api/admin/courses/:courseId/settings`
**Descrição**: Atualizar configurações avançadas do curso
**Acesso**: Instrutor (próprios cursos) ou Admin

```javascript
// Payload de exemplo
{
  "allowDownloads": true,
  "hasCertificate": true,
  "maxStudents": 500,
  "accessDuration": 365, // dias
  "communityEnabled": true,
  "whatsappGroupUrl": "https://chat.whatsapp.com/grupo-curso",
  "forumEnabled": true,
  "reviewsEnabled": true
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Configurações atualizadas com sucesso",
  "settings": {
    "allowDownloads": true,
    "hasCertificate": true,
    "maxStudents": 500,
    "accessDuration": 365,
    "communityEnabled": true,
    "whatsappGroupUrl": "https://chat.whatsapp.com/grupo-curso",
    "forumEnabled": true,
    "reviewsEnabled": true
  }
}
```

## Configurações Disponíveis

| Configuração | Tipo | Descrição | Padrão |
|--------------|------|-----------|---------|
| `allowDownloads` | boolean | Permitir download de materiais | false |
| `hasCertificate` | boolean | Curso oferece certificado | false |
| `maxStudents` | number | Limite máximo de estudantes | null |
| `accessDuration` | number | Duração do acesso em dias | null |
| `communityEnabled` | boolean | Habilitar comunidade | true |
| `whatsappGroupUrl` | string | URL do grupo WhatsApp | null |
| `forumEnabled` | boolean | Habilitar fórum de discussões | true |
| `reviewsEnabled` | boolean | Habilitar avaliações | true |

## Integração Frontend

### Painel de Matriculados
```jsx
import React, { useState, useEffect } from 'react';

const EnrollmentsManager = ({ courseId, token }) => {
  const [enrollments, setEnrollments] = useState([]);
  const [stats, setStats] = useState(null);
  const [filters, setFilters] = useState({
    page: 1,
    limit: 20,
    status: '',
    search: ''
  });
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    fetchEnrollments();
  }, [filters]);
  
  const fetchEnrollments = async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams(filters);
      const response = await fetch(`/api/admin/courses/${courseId}/enrollments?${params}`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setEnrollments(data.enrollments);
      setStats(data.stats);
    } catch (error) {
      console.error('Erro ao buscar matriculados:', error);
    } finally {
      setLoading(false);
    }
  };
  
  const manageAccess = async (userId, action, reason = '') => {
    try {
      const response = await fetch(`/api/admin/courses/${courseId}/enrollments`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({ userId, action, reason })
      });
      
      const data = await response.json();
      if (data.success) {
        alert(`Acesso ${action === 'grant' ? 'concedido' : 'revogado'} com sucesso!`);
        fetchEnrollments();
      }
    } catch (error) {
      alert('Erro ao gerenciar acesso: ' + error.message);
    }
  };
  
  const getStatusBadge = (status) => {
    const badges = {
      'active': { text: 'Ativo', color: '#28a745' },
      'expired': { text: 'Expirado', color: '#6c757d' },
      'cancelled': { text: 'Cancelado', color: '#dc3545' }
    };
    
    const badge = badges[status] || badges['active'];
    
    return (
      <span 
        className="status-badge"
        style={{ 
          backgroundColor: badge.color,
          color: 'white',
          padding: '4px 8px',
          borderRadius: '4px',
          fontSize: '12px'
        }}
      >
        {badge.text}
      </span>
    );
  };
  
  return (
    <div className="enrollments-manager">
      <h2>👨‍🎓 Gerenciar Matriculados</h2>
      
      {/* Estatísticas */}
      {stats && (
        <div className="stats-overview">
          <div className="stat-card">
            <h3>{stats.totalEnrollments}</h3>
            <p>Total de Matrículas</p>
          </div>
          <div className="stat-card">
            <h3>{stats.activeEnrollments}</h3>
            <p>Matrículas Ativas</p>
          </div>
          <div className="stat-card">
            <h3>R$ {stats.totalRevenue.toLocaleString()}</h3>
            <p>Receita Total</p>
          </div>
          <div className="stat-card">
            <h3>{stats.averageProgress.toFixed(1)}%</h3>
            <p>Progresso Médio</p>
          </div>
        </div>
      )}
      
      {/* Filtros */}
      <div className="filters">
        <input
          type="text"
          placeholder="Buscar por nome ou email..."
          value={filters.search}
          onChange={(e) => setFilters({...filters, search: e.target.value})}
        />
        
        <select
          value={filters.status}
          onChange={(e) => setFilters({...filters, status: e.target.value})}
        >
          <option value="">Todos os status</option>
          <option value="active">Ativo</option>
          <option value="expired">Expirado</option>
          <option value="cancelled">Cancelado</option>
        </select>
      </div>
      
      {/* Lista de Matriculados */}
      <div className="enrollments-list">
        {loading ? (
          <div>Carregando...</div>
        ) : (
          enrollments.map(enrollment => (
            <div key={enrollment.id} className="enrollment-card">
              <div className="student-info">
                <img 
                  src={enrollment.student.avatar || '/default-avatar.png'} 
                  alt={enrollment.student.name}
                  className="avatar"
                />
                <div>
                  <h4>{enrollment.student.name}</h4>
                  <p>{enrollment.student.email}</p>
                  <p>Membro desde: {new Date(enrollment.student.memberSince).toLocaleDateString()}</p>
                </div>
              </div>
              
              <div className="enrollment-details">
                <p><strong>Status:</strong> {getStatusBadge(enrollment.status)}</p>
                <p><strong>Progresso:</strong> {enrollment.progressPercentage}%</p>
                <p><strong>Tempo de Estudo:</strong> {Math.round(enrollment.timeSpentMinutes / 60)}h</p>
                <p><strong>Matriculado em:</strong> {new Date(enrollment.enrolledAt).toLocaleDateString()}</p>
                {enrollment.expiresAt && (
                  <p><strong>Expira em:</strong> {new Date(enrollment.expiresAt).toLocaleDateString()}</p>
                )}
              </div>
              
              {enrollment.purchase && (
                <div className="purchase-info">
                  <p><strong>Compra:</strong> R$ {enrollment.purchase.amount}</p>
                  <p><strong>Método:</strong> {enrollment.purchase.paymentMethod}</p>
                  <p><strong>Data:</strong> {new Date(enrollment.purchase.purchaseDate).toLocaleDateString()}</p>
                </div>
              )}
              
              <div className="progress-info">
                <div className="progress-bar">
                  <div 
                    className="progress-fill" 
                    style={{ width: `${enrollment.progressPercentage}%` }}
                  />
                </div>
                <p>{enrollment.progressSummary.completedLessons} de {enrollment.progressSummary.totalLessons} aulas</p>
              </div>
              
              <div className="actions">
                {enrollment.status === 'active' ? (
                  <button 
                    onClick={() => {
                      const reason = prompt('Motivo da revogação:');
                      if (reason) manageAccess(enrollment.student.id, 'revoke', reason);
                    }}
                    className="btn btn-danger"
                  >
                    🚫 Revogar Acesso
                  </button>
                ) : (
                  <button 
                    onClick={() => manageAccess(enrollment.student.id, 'grant')}
                    className="btn btn-success"
                  >
                    ✅ Conceder Acesso
                  </button>
                )}
              </div>
            </div>
          ))
        )}
      </div>
    </div>
  );
};
```

### Dashboard de Analytics
```jsx
const CourseAnalytics = ({ courseId, token }) => {
  const [analytics, setAnalytics] = useState(null);
  const [period, setPeriod] = useState('30d');
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchAnalytics();
  }, [period]);
  
  const fetchAnalytics = async () => {
    setLoading(true);
    try {
      const response = await fetch(`/api/admin/courses/${courseId}/analytics?period=${period}`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setAnalytics(data);
    } catch (error) {
      console.error('Erro ao buscar analytics:', error);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <div>Carregando analytics...</div>;
  if (!analytics) return <div>Erro ao carregar dados</div>;
  
  return (
    <div className="course-analytics">
      <h2>📊 Analytics do Curso</h2>
      
      <div className="period-selector">
        <button 
          className={period === '7d' ? 'active' : ''}
          onClick={() => setPeriod('7d')}
        >
          7 dias
        </button>
        <button 
          className={period === '30d' ? 'active' : ''}
          onClick={() => setPeriod('30d')}
        >
          30 dias
        </button>
        <button 
          className={period === '90d' ? 'active' : ''}
          onClick={() => setPeriod('90d')}
        >
          90 dias
        </button>
      </div>
      
      <div className="overview-metrics">
        <div className="metric-card">
          <h3>{analytics.overview.totalEnrollments}</h3>
          <p>Total de Matrículas</p>
        </div>
        <div className="metric-card">
          <h3>{analytics.overview.completionRate}%</h3>
          <p>Taxa de Conclusão</p>
        </div>
        <div className="metric-card">
          <h3>R$ {analytics.overview.totalRevenue.toLocaleString()}</h3>
          <p>Receita Total</p>
        </div>
        <div className="metric-card">
          <h3>{(analytics.overview.totalStudyTime / 60).toFixed(0)}h</h3>
          <p>Tempo Total de Estudo</p>
        </div>
      </div>
      
      <div className="charts-section">
        <div className="chart-container">
          <h4>Matrículas por Dia</h4>
          <div className="line-chart">
            {analytics.charts.enrollmentsByDay.map((item, index) => (
              <div key={index} className="chart-point">
                <span className="date">{new Date(item.date).toLocaleDateString()}</span>
                <span className="value">{item.enrollments}</span>
              </div>
            ))}
          </div>
        </div>
        
        <div className="chart-container">
          <h4>Aulas Mais Assistidas</h4>
          <div className="bar-chart">
            {analytics.charts.topLessons.slice(0, 5).map((lesson, index) => (
              <div key={index} className="bar-item">
                <span className="lesson-title">{lesson.lessonTitle}</span>
                <div className="bar">
                  <div 
                    className="bar-fill" 
                    style={{ width: `${(lesson.views / analytics.charts.topLessons[0].views) * 100}%` }}
                  />
                </div>
                <span className="value">{lesson.views} visualizações</span>
              </div>
            ))}
          </div>
        </div>
      </div>
      
      <div className="insights-section">
        <h4>💡 Insights e Recomendações</h4>
        <div className="insights-list">
          <div className="insight-card">
            <h5>Módulo Mais Engajante</h5>
            <p>{analytics.insights.mostEngagingModule}</p>
          </div>
          <div className="insight-card">
            <h5>Padrões de Estudo</h5>
            <p>{analytics.insights.studyPatterns}</p>
          </div>
          {analytics.insights.recommendations && (
            <div className="insight-card">
              <h5>Recomendações</h5>
              <ul>
                {analytics.insights.recommendations.map((rec, index) => (
                  <li key={index}>{rec}</li>
                ))}
              </ul>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};
```

### Configurações do Curso
```jsx
const CourseSettings = ({ courseId, token }) => {
  const [settings, setSettings] = useState({
    allowDownloads: false,
    hasCertificate: false,
    maxStudents: null,
    accessDuration: null,
    communityEnabled: true,
    whatsappGroupUrl: '',
    forumEnabled: true,
    reviewsEnabled: true
  });
  const [loading, setLoading] = useState(false);
  
  const updateSettings = async () => {
    setLoading(true);
    try {
      const response = await fetch(`/api/admin/courses/${courseId}/settings`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(settings)
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Configurações salvas com sucesso!');
      }
    } catch (error) {
      alert('Erro ao salvar configurações: ' + error.message);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="course-settings">
      <h2>⚙️ Configurações do Curso</h2>
      
      <div className="settings-form">
        <div className="setting-group">
          <h4>Acesso e Conteúdo</h4>
          
          <label>
            <input
              type="checkbox"
              checked={settings.allowDownloads}
              onChange={(e) => setSettings({...settings, allowDownloads: e.target.checked})}
            />
            Permitir download de materiais
          </label>
          
          <label>
            <input
              type="checkbox"
              checked={settings.hasCertificate}
              onChange={(e) => setSettings({...settings, hasCertificate: e.target.checked})}
            />
            Curso oferece certificado
          </label>
          
          <label>
            Limite máximo de estudantes:
            <input
              type="number"
              value={settings.maxStudents || ''}
              onChange={(e) => setSettings({...settings, maxStudents: e.target.value ? parseInt(e.target.value) : null})}
              placeholder="Sem limite"
            />
          </label>
          
          <label>
            Duração do acesso (dias):
            <input
              type="number"
              value={settings.accessDuration || ''}
              onChange={(e) => setSettings({...settings, accessDuration: e.target.value ? parseInt(e.target.value) : null})}
              placeholder="Acesso vitalício"
            />
          </label>
        </div>
        
        <div className="setting-group">
          <h4>Comunidade</h4>
          
          <label>
            <input
              type="checkbox"
              checked={settings.communityEnabled}
              onChange={(e) => setSettings({...settings, communityEnabled: e.target.checked})}
            />
            Habilitar recursos de comunidade
          </label>
          
          <label>
            URL do grupo WhatsApp:
            <input
              type="url"
              value={settings.whatsappGroupUrl}
              onChange={(e) => setSettings({...settings, whatsappGroupUrl: e.target.value})}
              placeholder="https://chat.whatsapp.com/..."
            />
          </label>
          
          <label>
            <input
              type="checkbox"
              checked={settings.forumEnabled}
              onChange={(e) => setSettings({...settings, forumEnabled: e.target.checked})}
            />
            Habilitar fórum de discussões
          </label>
          
          <label>
            <input
              type="checkbox"
              checked={settings.reviewsEnabled}
              onChange={(e) => setSettings({...settings, reviewsEnabled: e.target.checked})}
            />
            Habilitar avaliações do curso
          </label>
        </div>
      </div>
      
      <button 
        onClick={updateSettings}
        disabled={loading}
        className="btn btn-primary"
      >
        {loading ? 'Salvando...' : '💾 Salvar Configurações'}
      </button>
    </div>
  );
};
```

## Códigos de Erro

| Código | Tipo | Descrição |
|--------|------|-----------|
| 400 | invalid_parameters | Parâmetros obrigatórios ausentes ou inválidos |
| 403 | permission_denied | Usuário sem permissão para acessar/modificar |
| 404 | not_found | Curso não encontrado |
| 404 | user_not_found | Usuário não encontrado |
| 404 | not_enrolled | Usuário não está matriculado no curso |
| 500 | server_error | Erro interno do servidor |

## Considerações de Segurança

1. **Verificação de Propriedade**: Instrutores só acessam próprios cursos
2. **Permissões de Admin**: Admins têm acesso total
3. **Auditoria**: Todas as ações são logadas com responsável
4. **Validação**: Dados validados antes de processamento
5. **Rate Limiting**: Proteção contra uso excessivo

## Melhores Práticas

1. **Analytics**: Atualizar métricas periodicamente
2. **Notificações**: Informar estudantes sobre mudanças de acesso
3. **Backup**: Manter histórico de configurações
4. **Performance**: Cache de estatísticas para cursos grandes
5. **UX**: Interface intuitiva para instrutores

Este sistema de administração oferece controle total aos instrutores e administradores, fornecendo insights valiosos para otimização dos cursos e gestão eficiente dos estudantes.
