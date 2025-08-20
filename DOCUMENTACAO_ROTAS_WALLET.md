# Documentação API - Sistema de Carteira Digital (Wallet)

## Visão Geral
O sistema de carteira digital gerencia o saldo dos usuários, permitindo visualização de ganhos, solicitações de saque e histórico completo de transações financeiras.

## Endpoints Disponíveis

### 1. Consultar Saldo da Carteira
**GET** `/api/wallet/balance`

**Autenticação:** Requerida

**Descrição:** Retorna o saldo atual, total de ganhos e informações detalhadas da carteira do usuário.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "balance": {
    "available": 1250.50,
    "totalEarnings": 3500.00,
    "totalWithdrawals": 2000.00,
    "pendingWithdrawals": 249.50,
    "lastUpdated": "2024-01-15T10:30:00Z"
  },
  "summary": {
    "totalTransactions": 45,
    "lastTransaction": "2024-01-15T09:15:00Z",
    "averageTransaction": 77.78
  }
}
```

**Lógica de Cálculo:**
- Saldo disponível = (Créditos + Comissões) - (Débitos + Saques) - Saques pendentes
- Total de ganhos = Soma de todas as transações de crédito e comissão
- Saques pendentes não afetam o saldo real até serem processados

### 2. Solicitar Saque
**POST** `/api/wallet/withdraw`

**Autenticação:** Requerida

**Descrição:** Cria uma solicitação de saque com validação de saldo e dados bancários.

**Body da Requisição:**
```json
{
  "amount": 500.00,
  "method": "pix",
  "bankData": {
    "pixKey": "user@email.com",
    "pixKeyType": "email"
  }
}
```

**Parâmetros:**
- `amount` (number, obrigatório): Valor do saque (mínimo R$ 50,00)
- `method` (string, obrigatório): Método de saque (`pix` ou `bank_transfer`)
- `bankData` (object, obrigatório): Dados bancários conforme o método escolhido

**Dados Bancários PIX:**
```json
{
  "pixKey": "chave-pix",
  "pixKeyType": "email|phone|cpf|random"
}
```

**Dados Bancários Transferência:**
```json
{
  "bank": "001",
  "agency": "1234",
  "account": "12345-6",
  "accountType": "checking|savings",
  "accountHolder": "Nome do Titular"
}
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Solicitação de saque criada com sucesso",
  "withdrawal": {
    "id": 123,
    "amount": 500.00,
    "method": "pix",
    "status": "pending",
    "requestedAt": "2024-01-15T10:30:00Z",
    "estimatedProcessing": "1-3 dias úteis"
  },
  "newBalance": 750.50
}
```

**Erros Possíveis:**
- `400` - Método de saque inválido
- `400` - Saldo insuficiente
- `400` - Valor abaixo do mínimo (R$ 50,00)
- `404` - Usuário não encontrado

### 3. Histórico de Transações
**GET** `/api/wallet/transactions`

**Autenticação:** Requerida

**Descrição:** Retorna o histórico paginado de todas as transações da carteira.

**Parâmetros de Query:**
- `page` (number, opcional): Página (padrão: 1)
- `limit` (number, opcional): Itens por página (padrão: 20)
- `type` (string, opcional): Filtrar por tipo (`credit`, `debit`, `commission`, `withdrawal`)
- `status` (string, opcional): Filtrar por status (`pending`, `completed`, `failed`, `paid`)

**Exemplo de Requisição:**
```
GET /api/wallet/transactions?page=1&limit=10&type=credit&status=completed
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "transactions": [
    {
      "id": 456,
      "amount": 150.00,
      "type": "credit",
      "status": "completed",
      "description": "Venda do produto: Curso de Marketing Digital",
      "paymentMethod": "credit_card",
      "productId": 789,
      "orderId": "ORD-2024-001",
      "metadata": {
        "commission_rate": 0.1,
        "producer_id": 123
      },
      "createdAt": "2024-01-15T09:15:00Z",
      "updatedAt": "2024-01-15T09:16:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 45,
    "totalPages": 5
  }
}
```

### 4. Métodos de Saque Disponíveis
**GET** `/api/wallet/withdrawal-methods`

**Autenticação:** Requerida

**Descrição:** Lista todos os métodos de saque disponíveis com suas configurações.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "methods": [
    {
      "id": "pix",
      "name": "PIX",
      "description": "Transferência instantânea via PIX",
      "processingTime": "Instantâneo",
      "minimumAmount": 50.00,
      "maximumAmount": 10000.00,
      "fee": 0,
      "requiredFields": ["pixKey", "pixKeyType"]
    },
    {
      "id": "bank_transfer",
      "name": "Transferência Bancária",
      "description": "Transferência para conta bancária",
      "processingTime": "1-3 dias úteis",
      "minimumAmount": 50.00,
      "maximumAmount": 50000.00,
      "fee": 0,
      "requiredFields": ["bank", "agency", "account", "accountType", "accountHolder"]
    }
  ]
}
```

## Tipos de Transação

### 1. Credit (Crédito)
- Entrada de dinheiro na carteira
- Origem: Vendas de produtos próprios
- Status: `completed` quando o pagamento é confirmado

### 2. Commission (Comissão)
- Ganhos por vendas como afiliado
- Calculado com base na taxa de comissão do produto
- Creditado automaticamente após confirmação de pagamento

### 3. Debit (Débito)
- Saída de dinheiro da carteira
- Pode ser usado para estornos ou ajustes

### 4. Withdrawal (Saque)
- Solicitação de transferência para conta bancária
- Status `pending` até processamento manual
- Deduzido do saldo disponível imediatamente

## Status de Transação

- **pending**: Aguardando processamento
- **completed**: Transação finalizada com sucesso
- **failed**: Transação falhou
- **paid**: Pagamento confirmado (para vendas)

## Validações e Regras de Negócio

### Saldo Disponível
```javascript
// Cálculo do saldo disponível
let availableBalance = 0;
transactions.forEach(transaction => {
  if (transaction.status === 'completed' || transaction.status === 'paid') {
    if (transaction.type === 'credit' || transaction.type === 'commission') {
      availableBalance += parseFloat(transaction.amount);
    } else if (transaction.type === 'debit' || transaction.type === 'withdrawal') {
      availableBalance -= parseFloat(transaction.amount);
    }
  }
});

// Subtrair saques pendentes
const pendingWithdrawals = transactions
  .filter(t => t.type === 'withdrawal' && t.status === 'pending')
  .reduce((sum, t) => sum + parseFloat(t.amount), 0);

availableBalance -= pendingWithdrawals;
```

### Validações de Saque
1. **Valor Mínimo**: R$ 50,00
2. **Saldo Suficiente**: Verificação antes da criação
3. **Método Válido**: Apenas `pix` e `bank_transfer`
4. **Dados Bancários**: Validação conforme método escolhido

### Segurança
- Autenticação obrigatória em todas as rotas
- Validação de propriedade (usuário só acessa própria carteira)
- Logs detalhados de todas as operações
- Verificação de saldo em tempo real

## Integração com Outros Sistemas

### Modelos Relacionados
- **WalletTransaction**: Armazena todas as transações
- **User**: Dados do proprietário da carteira
- **Product**: Produtos relacionados às transações
- **Order**: Pedidos que geraram transações

### Eventos Automáticos
- Criação de transação de crédito na confirmação de venda
- Cálculo automático de comissões para afiliados
- Atualização em tempo real do saldo disponível

## Exemplos de Uso

### Consultar Saldo
```javascript
const response = await fetch('/api/wallet/balance', {
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
});
const data = await response.json();
console.log(`Saldo disponível: R$ ${data.balance.available}`);
```

### Solicitar Saque PIX
```javascript
const withdrawalData = {
  amount: 200.00,
  method: 'pix',
  bankData: {
    pixKey: 'usuario@email.com',
    pixKeyType: 'email'
  }
};

const response = await fetch('/api/wallet/withdraw', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(withdrawalData)
});
```

### Consultar Histórico
```javascript
const response = await fetch('/api/wallet/transactions?page=1&limit=20&type=credit', {
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
});
const data = await response.json();
console.log(`Total de transações: ${data.pagination.total}`);
```

## Observações Técnicas

1. **Performance**: Consultas otimizadas com paginação
2. **Precisão Decimal**: Uso de `parseFloat()` para cálculos monetários
3. **Auditoria**: Logs detalhados de todas as operações
4. **Escalabilidade**: Sistema preparado para alto volume de transações
5. **Manutenibilidade**: Código modular com validações centralizadas
