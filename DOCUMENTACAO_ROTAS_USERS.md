# 👥 Documentação Completa - Rotas de Usuários

## Visão Geral
O arquivo `routes/users.js` contém todas as rotas para gerenciamento CRUD de usuários, incluindo atualização de perfil, configurações de segurança, carteira e estatísticas.

## Middleware Utilizado
- **Autenticação**: Algumas rotas podem usar `authenticateToken` (verificar implementação específica)
- **Validação**: Validação de dados de entrada customizada

---

## 👥 CRUD DE USUÁRIOS

### GET /api/users
**Descrição**: Lista todos os usuários do sistema

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "users": [
    {
      "id": "uuid-user-1",
      "name": "João Silva",
      "email": "joao@email.com",
      "cpf": "123.456.789-00",
      "birthDate": "1990-01-01",
      "whatsapp": "(11) 99999-9999",
      "role": "user",
      "emailVerified": true,
      "profileCompleted": true,
      "walletBalance": 150.50,
      "twoFactorEnabled": false,
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2025-01-01T00:00:00Z"
    },
    {
      "id": "uuid-user-2",
      "name": "Maria Santos",
      "email": "maria@email.com",
      "cpf": "987.654.321-00",
      "birthDate": "1985-05-15",
      "whatsapp": "(11) 88888-8888",
      "role": "admin",
      "emailVerified": true,
      "profileCompleted": true,
      "walletBalance": 0.00,
      "twoFactorEnabled": true,
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2025-01-01T00:00:00Z"
    }
  ],
  "total": 2
}
```

**Observações**:
- A senha é automaticamente excluída da resposta
- Ordenação por data de criação (mais recentes primeiro)
- Retorna todos os usuários sem paginação

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

### GET /api/users/:id
**Descrição**: Busca um usuário específico por ID

**Parâmetros**:
- `id` (string): UUID do usuário

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "user": {
    "id": "uuid-user-1",
    "name": "João Silva",
    "email": "joao@email.com",
    "cpf": "123.456.789-00",
    "birthDate": "1990-01-01",
    "whatsapp": "(11) 99999-9999",
    "role": "user",
    "emailVerified": true,
    "profileCompleted": true,
    "walletBalance": 150.50,
    "twoFactorEnabled": false,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

**Erros Possíveis**:
- `404`: Usuário não encontrado
- `500`: Erro interno do servidor

---

### PUT /api/users/:id
**Descrição**: Atualiza dados de um usuário específico

**Parâmetros**:
- `id` (string): UUID do usuário

**Dados Esperados**:
```json
{
  "name": "João Silva Santos",
  "email": "joao.santos@email.com",
  "cpf": "123.456.789-00",
  "birthDate": "1990-01-01",
  "whatsapp": "(11) 99999-9999",
  "role": "admin",
  "emailVerified": true,
  "profileCompleted": true
}
```

**Campos Permitidos para Atualização**:
- `name`: Nome completo do usuário
- `email`: Email (validado e verificado unicidade)
- `cpf`: CPF do usuário
- `birthDate`: Data de nascimento (formato YYYY-MM-DD)
- `whatsapp`: Número do WhatsApp
- `role`: Papel do usuário (user, admin, etc.)
- `emailVerified`: Status de verificação do email
- `profileCompleted`: Status de completude do perfil

**Validações**:
- Email deve ter formato válido
- Email deve ser único (exceto para o próprio usuário)
- Campos são convertidos automaticamente de camelCase para snake_case

**Conversões Automáticas**:
- `profileCompleted` → `profile_completed`
- `birthDate` → `birth_date`
- `emailVerified` → `email_verified`

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Usuário atualizado com sucesso",
  "user": {
    "id": "uuid-user-1",
    "name": "João Silva Santos",
    "email": "joao.santos@email.com",
    "cpf": "123.456.789-00",
    "birthDate": "1990-01-01",
    "whatsapp": "(11) 99999-9999",
    "role": "admin",
    "emailVerified": true,
    "profileCompleted": true,
    "walletBalance": 150.50,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

**Erros Possíveis**:
- `400`: Formato de email inválido
- `404`: Usuário não encontrado
- `409`: Email já está em uso por outro usuário
- `500`: Erro interno do servidor

---

## 🔒 CONFIGURAÇÕES DE SEGURANÇA

### PUT /api/users/:id/security
**Descrição**: Atualiza configurações de segurança do usuário (senha e 2FA)

**Parâmetros**:
- `id` (string): UUID do usuário

**Dados Esperados**:
```json
{
  "currentPassword": "senha_atual",
  "newPassword": "nova_senha_123",
  "twoFactorEnabled": true
}
```

**Campos Opcionais**:
- `currentPassword`: Senha atual (obrigatória se alterando senha)
- `newPassword`: Nova senha (mínimo 6 caracteres)
- `twoFactorEnabled`: Habilitar/desabilitar 2FA

**Validações**:
- Se `newPassword` fornecida, `currentPassword` é obrigatória
- Senha atual deve estar correta
- Nova senha deve ter pelo menos 6 caracteres
- Hash da senha com bcrypt (salt rounds 12)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Configurações de segurança atualizadas com sucesso"
}
```

**Erros Possíveis**:
- `400`: Senha atual obrigatória, nova senha muito curta
- `401`: Senha atual incorreta
- `404`: Usuário não encontrado
- `500`: Erro interno do servidor

---

## 💰 GESTÃO DE CARTEIRA

### PUT /api/users/:id/wallet
**Descrição**: Atualiza saldo da carteira do usuário

**Parâmetros**:
- `id` (string): UUID do usuário

**Dados Esperados**:
```json
{
  "balance": 250.75
}
```

**Validações**:
- `balance` deve ser um número
- `balance` deve ser positivo (≥ 0)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Saldo atualizado com sucesso",
  "balance": 250.75
}
```

**Erros Possíveis**:
- `400`: Saldo deve ser um número positivo
- `404`: Usuário não encontrado
- `500`: Erro interno do servidor

---

## 📊 ESTATÍSTICAS DO USUÁRIO

### GET /api/users/:id/stats
**Descrição**: Obtém estatísticas e resumo do usuário

**Parâmetros**:
- `id` (string): UUID do usuário

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "stats": {
    "totalProducts": 0,
    "totalOrders": 0,
    "totalRevenue": 0,
    "totalCustomers": 0,
    "walletBalance": 150.50,
    "twoFactorEnabled": false,
    "profileCompleted": true,
    "userRole": "user",
    "memberSince": "2024-01-01T00:00:00Z",
    "lastUpdated": "2025-01-01T00:00:00Z"
  }
}
```

**Campos de Estatísticas**:
- `totalProducts`: Total de produtos criados (TODO: implementar contagem real)
- `totalOrders`: Total de pedidos recebidos (TODO: implementar contagem real)
- `totalRevenue`: Receita total gerada (TODO: implementar cálculo real)
- `totalCustomers`: Clientes únicos (TODO: implementar contagem real)
- `walletBalance`: Saldo atual da carteira
- `twoFactorEnabled`: Status do 2FA
- `profileCompleted`: Status de completude do perfil
- `userRole`: Papel do usuário
- `memberSince`: Data de criação da conta
- `lastUpdated`: Última atualização do perfil

**Observações**:
- Algumas estatísticas estão marcadas como TODO e retornam 0
- Implementação futura incluirá dados reais de produtos, pedidos e receita

**Erros Possíveis**:
- `404`: Usuário não encontrado
- `500`: Erro interno do servidor

---

## 🗑️ EXCLUSÃO DE USUÁRIO

### DELETE /api/users/:id
**Descrição**: Remove um usuário do sistema permanentemente

**Parâmetros**:
- `id` (string): UUID do usuário

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Usuário deletado com sucesso"
}
```

**Observações**:
- Operação irreversível
- Remove permanentemente o usuário do banco de dados
- Email do usuário é registrado nos logs antes da exclusão

**Erros Possíveis**:
- `404`: Usuário não encontrado
- `500`: Erro interno do servidor

---

## 🔍 Detalhes Técnicos

### Conversão de Campos
O sistema faz conversão automática de camelCase para snake_case para compatibilidade com o banco de dados:

| Frontend (camelCase) | Backend (snake_case) |
|---------------------|---------------------|
| `profileCompleted` | `profile_completed` |
| `birthDate` | `birth_date` |
| `emailVerified` | `email_verified` |
| `walletBalance` | `wallet_balance` |
| `twoFactorEnabled` | `two_factor_enabled` |

### Segurança
- **Exclusão de Senha**: Todas as consultas excluem automaticamente o campo `password`
- **Hash de Senha**: Senhas são hasheadas com bcrypt (salt rounds 12)
- **Validação de Email**: Verificação de formato e unicidade
- **Logs Detalhados**: Todas as operações são registradas nos logs

### Validações Implementadas
1. **Email único**: Verificado em atualizações (exceto próprio usuário)
2. **Formato de email**: Regex de validação padrão
3. **Senha forte**: Mínimo 6 caracteres
4. **Saldo positivo**: Carteira aceita apenas valores ≥ 0
5. **Campos permitidos**: Filtro de campos editáveis

### Campos de Resposta Padrão
Todos os usuários retornados incluem:
- `id`, `name`, `email`, `cpf`, `birthDate`, `whatsapp`
- `role`, `emailVerified`, `profileCompleted`
- `walletBalance`, `twoFactorEnabled`
- `created_at`, `updated_at`
- **Excluído**: `password`

### Status Codes
- `200`: Operação bem-sucedida
- `400`: Dados inválidos
- `401`: Autenticação falhou
- `404`: Recurso não encontrado
- `409`: Conflito (email duplicado)
- `500`: Erro interno do servidor

### Logs
Todas as operações geram logs detalhados:
- `📋`: Listagem de usuários
- `👤`: Busca por ID
- `✏️`: Atualização de dados
- `🔒`: Configurações de segurança
- `💰`: Operações de carteira
- `📊`: Consulta de estatísticas
- `🗑️`: Exclusão de usuário

---

## 🚀 Melhorias Futuras (TODO)

### Estatísticas Reais
- Implementar contagem real de produtos por usuário
- Calcular receita total baseada em pedidos
- Contar clientes únicos por vendedor
- Adicionar métricas de conversão

### Funcionalidades Adicionais
- Paginação para listagem de usuários
- Filtros de busca (nome, email, role)
- Histórico de alterações
- Soft delete (exclusão lógica)
- Upload de avatar
- Validação de CPF

### Segurança Avançada
- Rate limiting para operações sensíveis
- Auditoria de alterações
- Políticas de senha mais rígidas
- Verificação de 2FA para operações críticas

---

**Próxima documentação**: `routes/products.js` - Gestão de produtos e catálogo
