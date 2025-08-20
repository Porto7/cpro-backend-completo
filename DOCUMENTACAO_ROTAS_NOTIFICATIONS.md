# 🔔 Documentação Completa - Rotas de Notificações

## Visão Geral
O arquivo `routes/notifications.js` contém o sistema completo de notificações em tempo real para usuários. Oferece funcionalidades para listar, marcar como lida, criar e gerenciar notificações com diferentes tipos, prioridades e filtros avançados.

## Funcionalidades Principais
- **Listagem Paginada**: Notificações com filtros e ordenação
- **Controle de Leitura**: Marcar individual ou em massa
- **Sistema de Tipos**: Categorização por tipo de notificação
- **Prioridades**: Sistema de urgência para notificações
- **Estatísticas**: Analytics detalhados de notificações
- **Criação Administrativa**: Criação manual para admins

## Tipos de Notificação
- `sale`: Nova venda realizada
- `system`: Notificações do sistema
- `warning`: Avisos importantes
- `info`: Informações gerais
- `success`: Confirmações de sucesso
- `error`: Notificações de erro
- `payment`: Relacionadas a pagamentos
- `course`: Relacionadas a cursos
- `product`: Relacionadas a produtos

## Níveis de Prioridade
- `low`: Baixa prioridade
- `normal`: Prioridade normal (padrão)
- `high`: Alta prioridade
- `urgent`: Urgente

---

## 📋 LISTAGEM DE NOTIFICAÇÕES

### GET /api/notifications
**Descrição**: Lista notificações do usuário logado com filtros e paginação

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Query Parameters**:
- `page` (opcional): Página da paginação (padrão: 1)
- `limit` (opcional): Itens por página (padrão: 20)
- `type` (opcional): Filtrar por tipo de notificação
- `read` (opcional): Filtrar por status de leitura (true/false)
- `priority` (opcional): Filtrar por prioridade

**Exemplo de Requisição**:
```
GET /api/notifications?page=1&limit=10&type=sale&read=false&priority=high
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "notifications": [
    {
      "id": "uuid-notif-123",
      "title": "Nova Venda Realizada! 🎉",
      "message": "Você vendeu o produto 'Curso de JavaScript' por R$ 197,50",
      "type": "sale",
      "priority": "high",
      "read": false,
      "data": {
        "productId": "uuid-produto-456",
        "orderId": "uuid-pedido-789",
        "amount": 197.50,
        "buyerName": "João Silva"
      },
      "createdAt": "2024-12-31T20:30:00.000Z",
      "updatedAt": "2024-12-31T20:30:00.000Z",
      "typeLabel": "Nova Venda",
      "priorityLabel": "Alta",
      "icon": "shopping-cart",
      "color": "orange"
    },
    {
      "id": "uuid-notif-124",
      "title": "Sistema de Backup Concluído",
      "message": "Backup dos seus dados foi realizado com sucesso",
      "type": "system",
      "priority": "normal",
      "read": true,
      "data": {
        "backupSize": "2.5MB",
        "timestamp": "2024-12-31T18:00:00.000Z"
      },
      "createdAt": "2024-12-31T18:05:00.000Z",
      "updatedAt": "2024-12-31T19:00:00.000Z",
      "typeLabel": "Sistema",
      "priorityLabel": "Normal",
      "icon": "settings",
      "color": "blue"
    }
  ],
  "unreadCount": 5,
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 25,
    "totalPages": 3,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
```

**Campos da Notificação**:
- `id`: UUID único da notificação
- `title`: Título da notificação
- `message`: Mensagem detalhada
- `type`: Tipo da notificação
- `priority`: Nível de prioridade
- `read`: Status de leitura (boolean)
- `data`: Dados adicionais em JSON
- `typeLabel`: Label traduzida do tipo
- `priorityLabel`: Label traduzida da prioridade
- `icon`: Ícone sugerido para frontend
- `color`: Cor baseada na prioridade

**Ordenação**: Por prioridade (DESC) e data de criação (DESC)

**Uso**: Dashboard principal e centro de notificações

**Erros Possíveis**:
- `401`: Token inválido ou expirado
- `500`: Erro interno do servidor

---

## ✅ MARCAR COMO LIDA

### PUT /api/notifications/:id/read
**Descrição**: Marca notificação específica como lida

**Parâmetros**:
- `id` (string): UUID da notificação

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Notificação marcada como lida",
  "unreadCount": 4
}
```

**Efeitos da Ação**:
- Campo `read` alterado para `true`
- Campo `read_at` definido com timestamp atual
- Contador de não lidas atualizado

**Segurança**: Usuário só pode marcar suas próprias notificações

**Uso**: Quando usuário clica/visualiza uma notificação

**Erros Possíveis**:
- `404`: Notificação não encontrada ou não pertence ao usuário
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### PUT /api/notifications/mark-all-read
**Descrição**: Marca todas as notificações não lidas como lidas

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "5 notificações marcadas como lidas",
  "markedCount": 5,
  "unreadCount": 0
}
```

**Operação**: Atualiza em massa todas as notificações não lidas do usuário

**Campos Atualizados**:
- `read`: true
- `read_at`: timestamp atual

**Uso**: Botão "Marcar todas como lidas"

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## ➕ CRIAÇÃO DE NOTIFICAÇÕES

### POST /api/notifications/create
**Descrição**: Cria nova notificação (admins ou auto-notificação)

**Autenticação**: Obrigatória

**Body Parameters**:
```json
{
  "userId": "uuid-usuario-123",
  "title": "Produto Aprovado",
  "message": "Seu produto 'Curso de React' foi aprovado e está disponível para venda",
  "type": "success",
  "priority": "high",
  "data": {
    "productId": "uuid-produto-456",
    "productName": "Curso de React",
    "approvedBy": "admin"
  }
}
```

**Parâmetros Obrigatórios**:
- `title` (string): Título da notificação
- `message` (string): Mensagem detalhada

**Parâmetros Opcionais**:
- `userId` (string): ID do usuário destinatário (se omitido, usa usuário logado)
- `type` (string): Tipo de notificação (padrão: "info")
- `priority` (string): Prioridade (padrão: "normal")
- `data` (object): Dados adicionais

**Permissões**:
- **Admin**: Pode criar para qualquer usuário
- **Usuário**: Pode criar apenas para si mesmo

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Notificação criada com sucesso",
  "notification": {
    "id": "uuid-notif-novo",
    "title": "Produto Aprovado",
    "message": "Seu produto 'Curso de React' foi aprovado e está disponível para venda",
    "type": "success",
    "priority": "high",
    "createdAt": "2024-12-31T21:00:00.000Z"
  }
}
```

**Uso**: 
- Sistemas automáticos (webhooks, processos internos)
- Criação manual por administradores
- Auto-notificações de usuários

**Erros Possíveis**:
- `400`: Título e mensagem obrigatórios
- `403`: Sem permissão para criar para outro usuário
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 🗑️ EXCLUSÃO DE NOTIFICAÇÕES

### DELETE /api/notifications/:id
**Descrição**: Deleta notificação específica

**Parâmetros**:
- `id` (string): UUID da notificação

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Notificação deletada com sucesso"
}
```

**Segurança**: Usuário só pode deletar suas próprias notificações

**Uso**: Limpeza manual de notificações antigas

**Erros Possíveis**:
- `404`: Notificação não encontrada ou não pertence ao usuário
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 📊 ESTATÍSTICAS DE NOTIFICAÇÕES

### GET /api/notifications/stats
**Descrição**: Retorna estatísticas detalhadas das notificações do usuário

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "stats": {
    "total": 25,
    "unread": 5,
    "read": 20,
    "readPercentage": 80.0,
    "byType": {
      "sale": 8,
      "system": 6,
      "warning": 3,
      "info": 4,
      "success": 2,
      "error": 1,
      "payment": 1
    },
    "byPriority": {
      "low": 5,
      "normal": 15,
      "high": 4,
      "urgent": 1
    }
  }
}
```

**Métricas Calculadas**:
- **Total**: Número total de notificações
- **Unread**: Notificações não lidas
- **Read**: Notificações lidas
- **ReadPercentage**: Percentual de leitura
- **ByType**: Distribuição por tipo
- **ByPriority**: Distribuição por prioridade

**Uso**: 
- Dashboard de analytics
- Gráficos de engajamento
- Relatórios de uso

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 🎨 Sistema de Apresentação

### Ícones por Tipo
```javascript
const icons = {
  sale: 'shopping-cart',
  system: 'settings',
  warning: 'alert-triangle',
  info: 'info',
  success: 'check-circle',
  error: 'x-circle',
  payment: 'credit-card',
  course: 'book',
  product: 'package'
};
```

### Cores por Prioridade
```javascript
const colors = {
  low: 'gray',
  normal: 'blue',
  high: 'orange',
  urgent: 'red'
};
```

### Labels Traduzidas
```javascript
const typeLabels = {
  sale: 'Nova Venda',
  system: 'Sistema',
  warning: 'Aviso',
  info: 'Informação',
  success: 'Sucesso',
  error: 'Erro',
  payment: 'Pagamento',
  course: 'Curso',
  product: 'Produto'
};

const priorityLabels = {
  low: 'Baixa',
  normal: 'Normal',
  high: 'Alta',
  urgent: 'Urgente'
};
```

---

## 🔄 Automações e Integrações

### Criação Automática de Notificações

**Nova Venda**:
```javascript
await Notification.create({
  user_id: sellerId,
  title: "Nova Venda Realizada! 🎉",
  message: `Você vendeu "${productName}" por R$ ${amount}`,
  type: "sale",
  priority: "high",
  data: { productId, orderId, amount, buyerName }
});
```

**Produto Aprovado**:
```javascript
await Notification.create({
  user_id: sellerId,
  title: "Produto Aprovado",
  message: `Seu produto "${productName}" foi aprovado`,
  type: "success",
  priority: "normal",
  data: { productId, approvedBy }
});
```

**Pagamento Processado**:
```javascript
await Notification.create({
  user_id: userId,
  title: "Pagamento Confirmado",
  message: `Pagamento de R$ ${amount} foi processado`,
  type: "payment",
  priority: "high",
  data: { orderId, paymentMethod, transactionId }
});
```

### Integração com WebSockets
```javascript
// Notificação em tempo real via Socket.IO
io.to(`user_${userId}`).emit('new_notification', {
  id: notification.id,
  title: notification.title,
  message: notification.message,
  type: notification.type,
  priority: notification.priority
});
```

---

## 📱 Implementação Frontend

### Polling de Notificações
```javascript
// Verificar novas notificações a cada 30 segundos
setInterval(async () => {
  const response = await fetch('/api/notifications?limit=5');
  const data = await response.json();
  updateNotificationBadge(data.unreadCount);
}, 30000);
```

### Marcação Automática
```javascript
// Marcar como lida quando notificação é clicada
const markAsRead = async (notificationId) => {
  await fetch(`/api/notifications/${notificationId}/read`, {
    method: 'PUT',
    headers: { 'Authorization': `Bearer ${token}` }
  });
};
```

### Filtros Dinâmicos
```javascript
// Filtrar notificações por tipo
const filterByType = async (type) => {
  const response = await fetch(`/api/notifications?type=${type}`);
  const data = await response.json();
  renderNotifications(data.notifications);
};
```

---

## 🚀 Funcionalidades Futuras

### Push Notifications
- Integração com Firebase Cloud Messaging
- Notificações push para mobile
- Configurações de preferência de notificação

### Templates de Notificação
- Sistema de templates para tipos específicos
- Personalização de mensagens
- Multi-idioma

### Agrupamento Inteligente
- Agrupar notificações similares
- Resumos automáticos
- Digest de notificações

### Analytics Avançados
- Taxa de abertura de notificações
- Tempo médio de leitura
- Eficácia por tipo de notificação

---

## 📈 Casos de Uso

### Para Vendedores
- **Vendas**: Notificação imediata de novas vendas
- **Produtos**: Updates sobre aprovação/rejeição de produtos
- **Sistema**: Avisos sobre manutenção ou atualizações

### Para Compradores
- **Pedidos**: Confirmação de compra e status de entrega
- **Cursos**: Novos conteúdos disponíveis
- **Promoções**: Ofertas personalizadas

### Para Administradores
- **Sistema**: Alerts sobre performance e erros
- **Usuários**: Notificações sobre ações que precisam de atenção
- **Relatórios**: Resumos automáticos de atividade

---

## 🔧 Configurações Avançadas

### Preferências de Notificação
```javascript
const preferences = {
  email: true,
  push: false,
  inApp: true,
  types: {
    sale: true,
    system: false,
    marketing: true
  },
  schedule: {
    startTime: '08:00',
    endTime: '22:00',
    daysOfWeek: [1, 2, 3, 4, 5] // Segunda a Sexta
  }
};
```

### Rate Limiting
- Máximo 10 notificações por usuário por hora
- Agrupamento automático de notificações similares
- Throttling para evitar spam

### Retenção de Dados
- Notificações lidas: 90 dias
- Notificações não lidas: indefinido
- Limpeza automática de notificações antigas

---

**Próxima documentação**: `routes/settings.js` - Configurações do sistema
