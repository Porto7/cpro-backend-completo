# 🔐 Documentação Completa - Rotas Administrativas

## Visão Geral
O arquivo `routes/admin.js` contém rotas administrativas para gerenciamento do sistema, incluindo solicitações de exclusão de conta e aprovação de documentos KYC. Todas as rotas são protegidas e requerem privilégios de administrador.

## Middleware de Segurança
- **Autenticação**: `authenticateToken` obrigatório
- **Autorização**: `requireAdmin` - verifica se usuário tem role "admin"
- **Controle de Acesso**: Apenas administradores podem acessar estas rotas

## Funcionalidades
- **Solicitações de Exclusão**: Gerenciamento de pedidos de remoção de conta
- **Aprovação KYC**: Validação de documentos de identidade
- **Gestão de Usuários**: Controles administrativos avançados

---

## 🗑️ SOLICITAÇÕES DE EXCLUSÃO DE CONTA

### GET /api/admin/account/deletion-requests
**Descrição**: Lista todas as solicitações de exclusão de conta

**Autenticação**: Obrigatória (Admin)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "requests": [],
  "markedUsers": [
    {
      "id": "uuid-usuario-123",
      "name": "João Silva",
      "email": "joao@email.com",
      "created_at": "2024-12-01T10:00:00.000Z"
    }
  ],
  "total": 0
}
```

**Campos da Resposta**:
- `requests`: Lista de solicitações pendentes (sistema futuro)
- `markedUsers`: Usuários marcados para exclusão
- `total`: Total de solicitações

**Status Atual**: Sistema preparado para implementação futura
- TODO: Criar tabela `deletion_requests`
- TODO: Implementar workflow completo de aprovação

**Uso**: Dashboard administrativo para gerenciar exclusões

**Erros Possíveis**:
- `403`: Usuário não é administrador
- `401`: Token inválido ou expirado
- `500`: Erro interno do servidor

---

### POST /api/admin/account/deletion-requests
**Descrição**: Cria nova solicitação de exclusão de conta

**Autenticação**: Obrigatória (Admin)

**Body Parameters**:
```json
{
  "userId": "uuid-usuario-123",
  "reason": "Solicitação do usuário via suporte"
}
```

**Parâmetros Obrigatórios**:
- `userId` (string): UUID do usuário a ser excluído

**Parâmetros Opcionais**:
- `reason` (string): Motivo da solicitação

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Solicitação de exclusão criada com sucesso",
  "request": {
    "id": 1735689600000,
    "userId": "uuid-usuario-123",
    "userEmail": "joao@email.com",
    "userName": "João Silva",
    "reason": "Solicitação do usuário via suporte",
    "status": "pending",
    "createdAt": "2024-12-31T20:00:00.000Z",
    "adminId": "uuid-admin-456"
  }
}
```

**Validações**:
- Verifica se usuário existe
- Verifica se admin tem permissões
- Registra quem criou a solicitação

**Status da Solicitação**:
- `pending`: Aguardando análise
- `approved`: Aprovada para exclusão
- `rejected`: Rejeitada
- `completed`: Exclusão executada

**Uso**: Criar solicitações de exclusão via painel administrativo

**Erros Possíveis**:
- `400`: ID do usuário obrigatório
- `404`: Usuário não encontrado
- `403`: Usuário não é administrador
- `500`: Erro interno do servidor

---

### PUT /api/admin/account/deletion-requests/:id
**Descrição**: Aprova ou rejeita solicitação de exclusão

**Parâmetros**:
- `id` (string): ID da solicitação

**Autenticação**: Obrigatória (Admin)

**Body Parameters**:
```json
{
  "action": "approve",
  "reason": "Documentos validados e processo aprovado"
}
```

**Parâmetros Obrigatórios**:
- `action` (string): Ação a ser tomada
  - `approve`: Aprovar exclusão
  - `reject`: Rejeitar exclusão

**Parâmetros Opcionais**:
- `reason` (string): Justificativa da decisão

**Resposta de Sucesso - Aprovação (200)**:
```json
{
  "success": true,
  "message": "Solicitação aprovada com sucesso",
  "request": {
    "id": "123",
    "status": "approved",
    "processedAt": "2024-12-31T20:05:00.000Z",
    "processedBy": "uuid-admin-456",
    "reason": "Documentos validados e processo aprovado"
  }
}
```

**Resposta de Sucesso - Rejeição (200)**:
```json
{
  "success": true,
  "message": "Solicitação rejeitada com sucesso",
  "request": {
    "id": "123",
    "status": "rejected",
    "processedAt": "2024-12-31T20:05:00.000Z",
    "processedBy": "uuid-admin-456",
    "reason": "Não atende aos critérios de exclusão"
  }
}
```

**Fluxo de Aprovação**:
1. Admin analisa solicitação
2. Verifica documentação necessária
3. Aprova ou rejeita com justificativa
4. Sistema registra decisão e responsável

**Uso**: Processamento de solicitações no painel administrativo

**Erros Possíveis**:
- `400`: Ação inválida (deve ser approve ou reject)
- `404`: Solicitação não encontrada
- `403`: Usuário não é administrador
- `500`: Erro interno do servidor

---

## 📋 APROVAÇÃO DE DOCUMENTOS KYC

### GET /api/admin/kyc/submissions
**Descrição**: Lista todas as submissões de KYC pendentes de aprovação

**Autenticação**: Obrigatória (Admin)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "submissions": [
    {
      "id": "uuid-usuario-123",
      "name": "Maria Santos",
      "email": "maria@email.com",
      "status": "pending",
      "documents": [
        {
          "type": "rg",
          "url": "https://exemplo.com/uploads/rg_maria.jpg",
          "name": "rg"
        },
        {
          "type": "cpf",
          "url": "https://exemplo.com/uploads/cpf_maria.jpg",
          "name": "cpf"
        },
        {
          "type": "selfie",
          "url": "https://exemplo.com/uploads/selfie_maria.jpg",
          "name": "selfie"
        }
      ],
      "submittedAt": "2024-12-30T15:30:00.000Z"
    }
  ],
  "total": 1
}
```

**Campos da Submissão**:
- `id`: UUID do usuário
- `name`: Nome completo do usuário
- `email`: Email para contato
- `status`: Status do KYC (pending, approved, rejected)
- `documents`: Array de documentos enviados
- `submittedAt`: Data de submissão

**Tipos de Documentos**:
- `rg`: RG ou identidade
- `cpf`: CPF ou documento fiscal
- `selfie`: Foto do rosto com documento
- `proof_address`: Comprovante de endereço
- `bank_statement`: Extrato bancário

**Status Possíveis**:
- `pending`: Aguardando análise
- `approved`: Aprovado
- `rejected`: Rejeitado
- `incomplete`: Documentos incompletos

**Uso**: Painel administrativo para validação de identidade

**Erros Possíveis**:
- `403`: Usuário não é administrador
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### POST /api/admin/kyc/approve
**Descrição**: Aprova KYC de um usuário específico

**Autenticação**: Obrigatória (Admin)

**Body Parameters**:
```json
{
  "userId": "uuid-usuario-123"
}
```

**Parâmetros Obrigatórios**:
- `userId` (string): UUID do usuário a ser aprovado

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "KYC aprovado com sucesso"
}
```

**Efeitos da Aprovação**:
- Status KYC alterado para "approved"
- Usuário pode acessar funcionalidades restritas
- Limits de transação podem ser aumentados
- Confiabilidade da conta é estabelecida

**Funcionalidades Liberadas**:
- Vendas de alto valor
- Saques sem limite reduzido
- Acesso a recursos premium
- Certificação de vendedor verificado

**Uso**: Aprovação manual após validação de documentos

**Erros Possíveis**:
- `400`: ID do usuário obrigatório
- `404`: Usuário não encontrado
- `403`: Usuário não é administrador
- `500`: Erro interno do servidor

---

### POST /api/admin/kyc/reject
**Descrição**: Rejeita KYC de um usuário com motivo específico

**Autenticação**: Obrigatória (Admin)

**Body Parameters**:
```json
{
  "userId": "uuid-usuario-123",
  "reason": "Documentos não são legíveis. Por favor, envie fotos mais claras."
}
```

**Parâmetros Obrigatórios**:
- `userId` (string): UUID do usuário a ser rejeitado

**Parâmetros Opcionais**:
- `reason` (string): Motivo específico da rejeição

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "KYC rejeitado com sucesso"
}
```

**Efeitos da Rejeição**:
- Status KYC alterado para "rejected"
- Motivo da rejeição salvo para feedback
- Usuário pode reenviar documentos corrigidos
- Funcionalidades restritas permanecem limitadas

**Motivos Comuns de Rejeição**:
- Documentos ilegíveis ou com má qualidade
- Informações inconsistentes entre documentos
- Documentos expirados ou inválidos
- Selfie não corresponde ao documento
- Falta de documentos obrigatórios

**Campos Atualizados no Usuário**:
```javascript
{
  kyc_status: 'rejected',
  kyc_rejection_reason: 'Motivo específico da rejeição'
}
```

**Uso**: Rejeição com feedback para melhoria

**Erros Possíveis**:
- `400`: ID do usuário obrigatório
- `404`: Usuário não encontrado
- `403`: Usuário não é administrador
- `500`: Erro interno do servidor

---

## 🔒 Middleware de Segurança

### requireAdmin
**Função**: Verifica se usuário logado tem privilégios de administrador

```javascript
const requireAdmin = (req, res, next) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({
      success: false,
      error: 'Acesso negado. Apenas administradores podem acessar este recurso.'
    });
  }
  next();
};
```

**Verificações**:
- `req.user.role` deve ser exatamente "admin"
- Rejeita qualquer outro role (seller, aluno, etc.)
- Retorna erro 403 para acesso negado

**Uso**: Aplicado a todas as rotas administrativas

**Roles Existentes no Sistema**:
- `admin`: Administrador total
- `seller`: Vendedor/instrutor
- `aluno`: Estudante/comprador
- `moderator`: Moderador (futuro)

---

## 📊 Fluxos de Trabalho

### Fluxo de Exclusão de Conta
```
1. Admin cria solicitação → POST /deletion-requests
2. Análise de documentação e validação
3. Decisão → PUT /deletion-requests/:id
4. Se aprovado → Executar exclusão (processo separado)
5. Se rejeitado → Notificar usuário
```

### Fluxo de Aprovação KYC
```
1. Usuário envia documentos → (via route separada)
2. Admin lista submissões → GET /kyc/submissions
3. Admin analisa documentos manualmente
4. Decisão → POST /kyc/approve OU POST /kyc/reject
5. Usuário é notificado do resultado
```

---

## 🚧 Funcionalidades Futuras (TODOs)

### Sistema de Exclusão Completo
```sql
-- Tabela futura para solicitações
CREATE TABLE deletion_requests (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  admin_id UUID REFERENCES users(id),
  reason TEXT,
  status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW(),
  processed_at TIMESTAMP,
  processed_by UUID REFERENCES users(id)
);
```

### Melhorias no KYC
- Upload direto de documentos via admin
- Sistema de comentários para feedback
- Histórico de submissões anteriores
- Integração com APIs de validação automática
- Dashboard de estatísticas de aprovação

### Logs de Auditoria
- Registro de todas as ações administrativas
- Histórico de aprovações/rejeições
- Rastreamento de responsabilidade
- Relatórios de atividade administrativa

---

## 🔍 Casos de Uso

### Para Administradores
- **Gestão de Compliance**: Aprovar documentos KYC conforme regulamentações
- **Controle de Qualidade**: Garantir que apenas usuários verificados vendam
- **Suporte ao Cliente**: Processar solicitações de exclusão de conta
- **Auditoria**: Manter rastro de todas as decisões administrativas

### Para o Sistema
- **Automação**: Liberar funcionalidades baseadas em status KYC
- **Segurança**: Prevenir fraudes através de verificação manual
- **Compliance**: Atender requisitos legais de identificação
- **Qualidade**: Manter padrão alto de usuários na plataforma

---

## 📈 Métricas Administrativas

### KYC Analytics
```javascript
const metrics = {
  total_submissions: 150,
  pending_review: 25,
  approved_this_month: 45,
  rejected_this_month: 15,
  approval_rate: 75.0,
  avg_review_time_hours: 24
};
```

### Exclusão de Contas
```javascript
const deletion_metrics = {
  total_requests: 10,
  pending_requests: 3,
  approved_this_month: 5,
  rejected_this_month: 2,
  avg_processing_time_days: 7
};
```

---

## 🚀 Integrações Futuras

### APIs de Validação
- **Serasa/SPC**: Validação automática de CPF
- **Receita Federal**: Verificação de dados fiscais
- **TSE**: Validação de documentos eleitorais
- **Facial Recognition**: Comparação automática de selfies

### Notificações
- **Email**: Notificar decisões de KYC
- **SMS**: Alertas para aprovações importantes
- **Push Notifications**: Updates via app mobile
- **Webhook**: Integrações com sistemas externos

---

**Próxima documentação**: `routes/notifications.js` - Sistema de notificações
