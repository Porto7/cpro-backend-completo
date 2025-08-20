# Documentação - Sistema de Membros (routes/members.js)

## Visão Geral
Sistema completo de área de membros que permite gestão de acesso, autenticação, progresso de cursos e conteúdo exclusivo para clientes que adquiriram produtos digitais.

## Características Principais
- **Gestão de Membros**: CRUD completo de membros por vendedor
- **Autenticação Específica**: Login separado para área de membros
- **Senhas Temporárias**: Sistema automático para novos membros
- **Progresso de Cursos**: Tracking detalhado de progresso
- **Conteúdo Exclusivo**: Acesso baseado em produtos adquiridos
- **Múltiplos Produtos**: Suporte a vários produtos por membro

## Rotas Implementadas

### 1. Listagem de Membros

#### `GET /api/members/:sellerId`
**Descrição**: Lista membros de um vendedor específico
**Acesso**: Autenticado (vendedor próprio ou admin)

**Query Parameters**:
- `page`: Página (padrão: 1)
- `limit`: Itens por página (padrão: 20)
- `search`: Busca por nome ou email
- `status`: Filtro por status (active, inactive, suspended, expired)

**Response Success**:
```json
{
  "success": true,
  "members": [
    {
      "id": "member-123",
      "name": "João Silva",
      "email": "joao@exemplo.com",
      "status": "active",
      "firstAccess": true,
      "lastAccessAt": "2024-01-01T15:30:00Z",
      "purchasedProducts": [
        {
          "id": "prod-456",
          "name": "Curso de Marketing",
          "progress": 65,
          "completedLessons": ["lesson-1", "lesson-2"],
          "purchaseDate": "2024-01-01T10:00:00Z",
          "lastAccessed": "2024-01-01T15:30:00Z"
        }
      ],
      "temporaryPassword": "******",
      "passwordExpiresAt": "2024-01-02T10:00:00Z",
      "seller": {
        "id": "seller-789",
        "name": "Maria Vendedora",
        "email": "maria@exemplo.com"
      },
      "orders": [
        {
          "id": "order-101",
          "orderNumber": "ORD-2024-001",
          "total": 197.00,
          "status": "completed"
        }
      ],
      "createdAt": "2024-01-01T10:00:00Z",
      "updatedAt": "2024-01-01T15:30:00Z",
      "statusLabel": "Ativo",
      "hasAccess": true,
      "needsPasswordReset": false
    }
  ],
  "stats": {
    "total": 150,
    "active": 125,
    "inactive": 15,
    "suspended": 5,
    "firstTimeAccess": 10
  },
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

**Status de Membros**:
- `active`: Membro ativo com acesso
- `inactive`: Membro inativo (sem acesso)
- `suspended`: Membro suspenso temporariamente
- `expired`: Acesso expirado

### 2. Autenticação de Membro

#### `POST /api/members/authenticate`
**Descrição**: Autentica membro na área de membros
**Acesso**: Público

```javascript
// Payload de exemplo
{
  "email": "joao@exemplo.com",
  "password": "ABC123", // Senha temporária ou senha definitiva
  "sellerId": "seller-789"
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Autenticação realizada com sucesso",
  "member": {
    "id": "member-123",
    "name": "João Silva",
    "email": "joao@exemplo.com",
    "firstAccess": true,
    "purchasedProducts": [
      {
        "id": "prod-456",
        "name": "Curso de Marketing",
        "progress": 65,
        "completedLessons": ["lesson-1", "lesson-2"]
      }
    ],
    "seller": {
      "id": "seller-789",
      "name": "Maria Vendedora",
      "email": "maria@exemplo.com"
    },
    "hasTemporaryPassword": true
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": "7 days"
}
```

**Tipos de Senha**:
1. **Senha Temporária**: Gerada automaticamente (24h de validade)
2. **Senha Hash**: Definida pelo próprio membro

**Validações**:
- Email, senha e sellerId obrigatórios
- Membro deve estar com status `active`
- Senha temporária não pode estar expirada
- Atualiza último acesso automaticamente

### 3. Criação de Acesso

#### `POST /api/members/create-access`
**Descrição**: Cria acesso para novo membro (chamado após compra)
**Acesso**: Autenticado

```javascript
// Payload de exemplo
{
  "customerEmail": "joao@exemplo.com",
  "customerName": "João Silva",
  "orderId": "order-101",
  "sellerId": "seller-789",
  "products": [
    {
      "id": "prod-456",
      "name": "Curso de Marketing Digital",
      "type": "course",
      "purchaseDate": "2024-01-01T10:00:00Z"
    },
    {
      "id": "prod-789",
      "name": "E-book Bonus",
      "type": "ebook",
      "purchaseDate": "2024-01-01T10:00:00Z"
    }
  ]
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Acesso de membro criado com sucesso",
  "member": {
    "id": "member-123",
    "email": "joao@exemplo.com",
    "name": "João Silva",
    "temporaryPassword": "ABC123",
    "expiresAt": "2024-01-02T10:00:00Z",
    "accessUrl": "https://app.exemplo.com/members/seller-789",
    "products": 2
  }
}
```

**Funcionalidades**:
- **Membro Novo**: Cria novo registro
- **Membro Existente**: Adiciona novos produtos
- **Senha Temporária**: Gerada automaticamente (6 caracteres)
- **Email Automático**: Credenciais enviadas por email
- **Validade**: 24 horas para primeira senha

### 4. Atualização de Progresso

#### `PUT /api/members/:memberId/progress`
**Descrição**: Atualiza progresso do membro em curso/produto
**Acesso**: Autenticado

```javascript
// Payload de exemplo
{
  "productId": "prod-456",
  "progress": 75, // Percentual de 0-100
  "lessonId": "lesson-5", // Opcional
  "completed": true // Se a lição foi completada
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Progresso atualizado com sucesso",
  "progress": {
    "productId": "prod-456",
    "progress": 75,
    "completedLessons": 5,
    "lastAccessed": "2024-01-01T16:00:00Z"
  }
}
```

**Tracking Incluído**:
- Percentual de progresso (0-100%)
- Aulas completadas (array de IDs)
- Último acesso ao produto
- Último acesso geral do membro

### 5. Conteúdo do Membro

#### `GET /api/members/:memberId/content`
**Descrição**: Obtém conteúdo disponível para o membro
**Acesso**: Autenticado

**Response Success**:
```json
{
  "success": true,
  "content": [
    {
      "id": "prod-456",
      "name": "Curso de Marketing Digital",
      "description": "Aprenda marketing digital do zero",
      "type": "course",
      "thumbnail": "https://exemplo.com/thumb.jpg",
      "progress": 75,
      "lastAccessed": "2024-01-01T16:00:00Z",
      "completedLessons": ["lesson-1", "lesson-2", "lesson-3"],
      "course": {
        "id": "course-123",
        "title": "Marketing Digital Completo",
        "modules": [
          {
            "id": "module-1",
            "title": "Fundamentos",
            "lessons": [
              {
                "id": "lesson-1",
                "title": "Introdução ao Marketing",
                "duration": 1800,
                "completed": true
              }
            ]
          }
        ],
        "totalLessons": 50,
        "duration": 90000
      },
      "accessGranted": true,
      "purchaseDate": "2024-01-01T10:00:00Z"
    }
  ],
  "member": {
    "id": "member-123",
    "name": "João Silva",
    "email": "joao@exemplo.com",
    "totalProducts": 2,
    "averageProgress": 62.5
  }
}
```

**Tipos de Conteúdo**:
- `course`: Cursos com módulos e aulas
- `ebook`: E-books e materiais PDF
- `video`: Vídeos avulsos
- `download`: Downloads/arquivos
- `webinar`: Webinars gravados

## Funcionalidades Avançadas

### 1. Sistema de Senhas
```javascript
// Senha temporária (primeira vez)
{
  "temporaryPassword": "ABC123", // 6 caracteres
  "passwordExpiresAt": "2024-01-02T10:00:00Z", // 24h
  "requiresReset": true
}

// Senha definitiva (após reset)
{
  "passwordHash": "$2b$10$...", // Bcrypt hash
  "temporaryPassword": null,
  "passwordExpiresAt": null,
  "requiresReset": false
}
```

### 2. Progresso Detalhado
```javascript
// Estrutura de progresso
{
  "purchasedProducts": [
    {
      "id": "prod-456",
      "name": "Curso de Marketing",
      "type": "course",
      "progress": 75,
      "completedLessons": ["lesson-1", "lesson-2"],
      "totalLessons": 10,
      "lastAccessed": "2024-01-01T16:00:00Z",
      "purchaseDate": "2024-01-01T10:00:00Z",
      "certificateEarned": false,
      "timeSpent": 7200 // segundos
    }
  ]
}
```

### 3. Acesso Condicional
```javascript
// Verificação de acesso
function hasAccess(member, productId) {
  const product = member.purchasedProducts.find(p => p.id === productId);
  
  if (!product) return false;
  if (member.status !== 'active') return false;
  if (member.temporary_password && isExpired(member.password_expires_at)) return false;
  
  return true;
}
```

## Integração com Sistema

### 1. Webhook Pós-Compra
```javascript
// Após confirmação de pagamento
async function createMemberAccess(order) {
  const response = await fetch('/api/members/create-access', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify({
      customerEmail: order.customer_info.email,
      customerName: order.customer_info.name,
      orderId: order.id,
      sellerId: order.seller_id,
      products: order.products
    })
  });
}
```

### 2. Frontend de Membro
```jsx
import React, { useState, useEffect } from 'react';

const MemberDashboard = ({ memberToken }) => {
  const [content, setContent] = useState([]);
  const [member, setMember] = useState(null);
  
  useEffect(() => {
    fetchMemberContent();
  }, []);
  
  const fetchMemberContent = async () => {
    const response = await fetch('/api/members/123/content', {
      headers: {
        'Authorization': `Bearer ${memberToken}`
      }
    });
    const data = await response.json();
    setContent(data.content);
    setMember(data.member);
  };
  
  const updateProgress = async (productId, progress, lessonId) => {
    await fetch('/api/members/123/progress', {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${memberToken}`
      },
      body: JSON.stringify({
        productId,
        progress,
        lessonId,
        completed: true
      })
    });
  };
  
  return (
    <div className="member-dashboard">
      <h1>Bem-vindo, {member?.name}!</h1>
      
      <div className="progress-overview">
        <p>Progresso Geral: {member?.averageProgress}%</p>
        <p>Produtos: {member?.totalProducts}</p>
      </div>
      
      <div className="content-grid">
        {content.map(item => (
          <div key={item.id} className="content-card">
            <img src={item.thumbnail} alt={item.name} />
            <h3>{item.name}</h3>
            <div className="progress-bar">
              <div 
                className="progress-fill" 
                style={{width: `${item.progress}%`}}
              />
            </div>
            <p>{item.progress}% Completo</p>
            <button onClick={() => openContent(item)}>
              Continuar
            </button>
          </div>
        ))}
      </div>
    </div>
  );
};
```

### 3. Login de Membro
```jsx
const MemberLogin = ({ sellerId }) => {
  const [credentials, setCredentials] = useState({
    email: '',
    password: '',
    sellerId
  });
  
  const handleLogin = async (e) => {
    e.preventDefault();
    
    const response = await fetch('/api/members/authenticate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });
    
    const data = await response.json();
    
    if (data.success) {
      localStorage.setItem('memberToken', data.token);
      window.location.href = '/member-dashboard';
    } else {
      alert(data.error);
    }
  };
  
  return (
    <form onSubmit={handleLogin}>
      <input
        type="email"
        placeholder="Seu email"
        value={credentials.email}
        onChange={(e) => setCredentials({
          ...credentials,
          email: e.target.value
        })}
      />
      <input
        type="password"
        placeholder="Sua senha"
        value={credentials.password}
        onChange={(e) => setCredentials({
          ...credentials,
          password: e.target.value
        })}
      />
      <button type="submit">Entrar na Área de Membros</button>
    </form>
  );
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Dados obrigatórios em falta |
| 401 | Credenciais inválidas ou senha expirada |
| 403 | Sem permissão para acessar membros |
| 404 | Membro ou produto não encontrado |
| 500 | Erro interno do servidor |

## Considerações de Segurança

1. **Senhas Temporárias**: Expiração automática em 24h
2. **Tokens JWT**: Expiração em 7 dias
3. **Verificação de Ownership**: Apenas vendedor ou admin
4. **Status de Acesso**: Verificação rigorosa
5. **Hash de Senhas**: Bcrypt para senhas definitivas

## Email Automático

### Template de Boas-vindas
```javascript
// services/emailService.js
export async function sendMemberAccessEmail(email, name, password, sellerId, products) {
  const template = `
    <h1>Bem-vindo à Área de Membros!</h1>
    <p>Olá ${name},</p>
    <p>Seu acesso foi criado com sucesso!</p>
    
    <div style="background: #f8f9fa; padding: 20px; margin: 20px 0;">
      <h3>Seus dados de acesso:</h3>
      <p><strong>Email:</strong> ${email}</p>
      <p><strong>Senha temporária:</strong> ${password}</p>
      <p><strong>Link de acesso:</strong> ${process.env.APP_URL}/members/${sellerId}</p>
    </div>
    
    <p><strong>Produtos adquiridos:</strong></p>
    <ul>
      ${products.map(p => `<li>${p.name}</li>`).join('')}
    </ul>
    
    <p><small>Esta senha expira em 24 horas. Faça seu primeiro acesso para definir uma senha permanente.</small></p>
  `;
  
  return sendEmail(email, 'Acesso à Área de Membros', template);
}
```

## Métricas e Analytics

### KPIs Importantes
- Taxa de primeiro acesso
- Progresso médio por produto
- Tempo médio de conclusão
- Produtos mais acessados
- Membros mais engajados

### Dashboard de Vendedor
```javascript
// Estatísticas para o vendedor
{
  "totalMembers": 150,
  "activeMembers": 125,
  "averageProgress": 68.5,
  "completionRate": 34.2,
  "mostPopularProduct": {
    "id": "prod-456",
    "name": "Curso de Marketing",
    "members": 89
  },
  "recentActivity": [
    {
      "member": "João Silva",
      "action": "completed_lesson",
      "product": "Curso de Marketing",
      "timestamp": "2024-01-01T16:00:00Z"
    }
  ]
}
```

Este sistema de membros oferece uma solução completa para gestão de acesso a conteúdo digital, com funcionalidades avançadas de progresso, autenticação e integração automática com o sistema de vendas.
