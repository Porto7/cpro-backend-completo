# Documentação API - Sistema de Pagamento PIX

## Visão Geral
Sistema completo de pagamento PIX com múltiplas adquirentes, seleção automática de processadora e fallback automático para garantir maior taxa de aprovação.

## Endpoints Disponíveis

### 1. Criar Pagamento PIX
**POST** `/api/pix-payment/create`

**Autenticação:** Opcional

**Descrição:** Cria um pagamento PIX com seleção automática de adquirente e fallback em caso de falha.

**Body da Requisição:**
```json
{
  "productId": "uuid-produto",
  "customer": {
    "name": "João Silva",
    "email": "joao@email.com",
    "phone": "11999999999",
    "document": {
      "number": "12345678901",
      "type": "cpf"
    }
  },
  "preferredAcquirer": "cartwavehub",
  "externalCode": "REF-2024-001",
  "discount": 50.00,
  "metadata": {
    "utm": {
      "source": "google",
      "medium": "cpc",
      "campaign": "vendas"
    },
    "additional_info": "Informações extras"
  },
  "ip": "192.168.1.1"
}
```

**Parâmetros:**
- `productId` (string, obrigatório): ID do produto
- `customer` (object, obrigatório): Dados do cliente
  - `name` (string): Nome completo
  - `email` (string): E-mail válido
  - `phone` (string): Telefone/WhatsApp
  - `document` (object): Documento do cliente
    - `number` (string): Número do CPF/CNPJ
    - `type` (string): Tipo do documento (cpf/cnpj)
- `preferredAcquirer` (string, opcional): Adquirente preferida
- `externalCode` (string, opcional): Código de referência externa
- `discount` (number, opcional): Valor do desconto em reais
- `metadata` (object, opcional): Dados adicionais
- `ip` (string, opcional): IP do cliente

**Resposta de Sucesso (201):**
```json
{
  "status": "success",
  "message": "Pagamento PIX criado com sucesso",
  "order_id": "uuid-pedido",
  "payment": {
    "pix_code": "00020126580014br.gov.bcb.pix...",
    "qr_code": "data:image/png;base64,iVBORw0KGgoAAAA...",
    "transaction_id": "TXN-2024-001",
    "expiration_date": "2024-01-15T23:59:59Z",
    "amount": 450.00,
    "acquirer": {
      "name": "CartWave Hub",
      "slug": "cartwavehub"
    }
  },
  "product": {
    "id": "uuid-produto",
    "name": "Curso de Marketing Digital",
    "seller": "João Producer"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Resposta com Fallback (201):**
```json
{
  "status": "success",
  "message": "Pagamento PIX criado com sucesso (fallback)",
  "order_id": "uuid-pedido",
  "payment": {
    "pix_code": "00020126580014br.gov.bcb.pix...",
    "qr_code": "data:image/png;base64,iVBORw0KGgoAAAA...",
    "transaction_id": "TXN-FALLBACK-001",
    "expiration_date": "2024-01-15T23:59:59Z",
    "amount": 450.00,
    "acquirer": {
      "name": "Adquirente Padrão",
      "slug": "default"
    }
  },
  "product": {
    "id": "uuid-produto",
    "name": "Curso de Marketing Digital",
    "seller": "João Producer"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Erros Possíveis:**
- `400` - Dados obrigatórios ausentes
- `400` - Dados do cliente incompletos
- `404` - Produto não encontrado
- `400` - Valor final inválido
- `400` - Falha ao criar pagamento PIX

### 2. Verificar Status do Pagamento
**GET** `/api/pix-payment/status/:orderId`

**Autenticação:** Não requerida

**Descrição:** Verifica o status atual do pagamento PIX consultando a adquirente em tempo real.

**Parâmetros de Rota:**
- `orderId` (string): ID do pedido

**Resposta de Sucesso (200):**
```json
{
  "status": "success",
  "order_id": "uuid-pedido",
  "payment_status": "completed",
  "acquirer_status": "paid",
  "acquirer": {
    "name": "CartWave Hub",
    "slug": "cartwavehub"
  },
  "product": {
    "name": "Curso de Marketing Digital"
  },
  "amount": 450.00,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:35:00Z",
  "timestamp": "2024-01-15T10:40:00Z"
}
```

**Status Possíveis:**
- `pending`: Aguardando pagamento
- `completed`: Pagamento confirmado
- `failed`: Pagamento falhou
- `refunded`: Pagamento estornado
- `cancelled`: Pagamento cancelado

**Resposta Pagamento Pendente (200):**
```json
{
  "status": "pending",
  "order_id": "uuid-pedido",
  "payment_status": "pending",
  "message": "Pagamento ainda não processado",
  "timestamp": "2024-01-15T10:40:00Z"
}
```

**Erros Possíveis:**
- `404` - Pedido não encontrado
- `500` - Erro ao verificar status

### 3. Webhook de Notificações
**POST** `/api/pix-payment/webhook/:acquirerSlug`

**Autenticação:** Não requerida

**Descrição:** Recebe notificações automáticas das adquirentes sobre mudanças de status dos pagamentos.

**Parâmetros de Rota:**
- `acquirerSlug` (string): Identificador da adquirente

**Body da Requisição (CartWaveHub):**
```json
{
  "orderId": "TXN-2024-001",
  "status": "paid",
  "paymentMethod": "pix",
  "code": "TXN-2024-001",
  "externalCode": "REF-2024-001",
  "amount": 45000,
  "customer": {
    "name": "João Silva",
    "email": "joao@email.com"
  },
  "processedAt": "2024-01-15T10:35:00Z"
}
```

**Resposta de Sucesso (200):**
```json
{
  "status": "success",
  "message": "Webhook processado",
  "order_id": "uuid-pedido",
  "new_status": "completed"
}
```

**Erros Possíveis:**
- `400` - orderId ou externalCode obrigatório
- `404` - Pedido não encontrado
- `500` - Erro ao processar webhook

## Fluxo de Pagamento

### 1. Seleção de Adquirente
```javascript
// O sistema seleciona automaticamente a melhor adquirente baseado em:
// - Adquirente preferida (se especificada)
// - Disponibilidade da adquirente
// - Taxa de aprovação histórica
// - Valor da transação

const acquirer = await paymentAcquirerService.selectAcquirer({
  preferred_acquirer: preferredAcquirer,
  amount: finalAmount,
  payment_method: 'pix'
});
```

### 2. Fallback Automático
```javascript
// Se a adquirente preferida falhar, tenta a adquirente padrão
if (!transactionResult.success && acquirer.slug !== 'cartwavehub') {
  const defaultAcquirer = await paymentAcquirerService.getDefaultAcquirer();
  const fallbackResult = await paymentAcquirerService.createPixTransaction(
    defaultAcquirer.slug,
    transactionData
  );
}
```

### 3. Processamento de Webhook
```javascript
// Mapeamento de status da adquirente para status interno
const statusMapping = {
  'paid': 'completed',
  'pending': 'pending',
  'refused': 'failed',
  'refunded': 'refunded',
  'cancelled': 'cancelled'
};
```

## Tratamento de Clientes

### Cliente Existente
Se o cliente já existir no sistema (por email), seus dados são reutilizados.

### Cliente Novo
```javascript
// Criação automática de novo cliente
const customerUser = await User.create({
  id: uuidv4(),
  name: customer.name,
  email: customer.email,
  cpf: customer.document.number,
  whatsapp: customer.phone,
  role: 'user',
  profile_completed: false,
  email_verified: false
});
```

## Cálculo de Valores

### Processamento de Valores
```javascript
// Conversão para centavos (precisão de cálculo)
const baseAmount = Math.round(product.price * 100);
const discountAmount = discount ? Math.round(discount * 100) : 0;
const finalAmount = baseAmount - discountAmount;

// Validação de valor mínimo
if (finalAmount <= 0) {
  throw new Error('Valor final deve ser maior que zero');
}
```

### Estrutura de Dados da Transação
```javascript
const transactionData = {
  customer: {
    name: customer.name,
    email: customer.email,
    phone: customer.phone,
    document: {
      number: customer.document.number,
      type: customer.document.type || 'cpf'
    }
  },
  items: [{
    title: product.name,
    description: product.description || product.name,
    unitPrice: finalAmount,
    quantity: 1,
    tangible: false
  }],
  amount: finalAmount,
  postbackUrl: `${process.env.APP_URL}/api/pix-payment/webhook/${acquirer.slug}`,
  externalCode: externalCode || orderId,
  isInfoProducts: true,
  discount: discountAmount,
  ip: ip || req.ip || '127.0.0.1',
  metadata: {
    ...metadata,
    order_id: orderId,
    product_id: productId,
    seller_id: product.seller_id,
    platform: 'checkoutpro'
  }
};
```

## Segurança e Validações

### Validações de Input
1. **Dados Obrigatórios**: productId, customer.name, customer.email, customer.document.number
2. **Formato de Email**: Validação de email válido
3. **Documento**: CPF/CNPJ válido
4. **Valor Mínimo**: Valor final deve ser maior que zero

### Autenticação Opcional
```javascript
// Middleware que permite uso com ou sem autenticação
const optionalAuthMiddleware = async (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;
    const token = authHeader && authHeader.split(' ')[1];

    if (token) {
      const decoded = jwt.verify(token, JWT_SECRET);
      const user = await User.findOne({ where: { id: decoded.userId } });
      req.user = user;
    }

    next();
  } catch (error) {
    // Continuar sem autenticação
    next();
  }
};
```

### Logs de Auditoria
- Criação de pagamento com detalhes completos
- Seleção de adquirente
- Tentativas de fallback
- Recebimento de webhooks
- Mudanças de status

## Integração com Adquirentes

### Adquirentes Suportadas
- **CartWave Hub**: Adquirente principal
- **Adquirente Padrão**: Fallback automático
- Sistema extensível para novas adquirentes

### Configuração de Webhook
```javascript
// URL automática para cada adquirente
const postbackUrl = `${process.env.APP_URL}/api/pix-payment/webhook/${acquirer.slug}`;
```

## Exemplos de Uso

### Criar Pagamento PIX Simples
```javascript
const paymentData = {
  productId: 'prod-123',
  customer: {
    name: 'João Silva',
    email: 'joao@email.com',
    phone: '11999999999',
    document: {
      number: '12345678901',
      type: 'cpf'
    }
  }
};

const response = await fetch('/api/pix-payment/create', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(paymentData)
});
```

### Verificar Status
```javascript
const response = await fetch(`/api/pix-payment/status/${orderId}`);
const status = await response.json();

if (status.payment_status === 'completed') {
  console.log('Pagamento confirmado!');
}
```

### Monitorar Pagamento
```javascript
// Polling para verificar status
const checkPaymentStatus = async (orderId) => {
  const response = await fetch(`/api/pix-payment/status/${orderId}`);
  const data = await response.json();
  
  if (data.payment_status === 'completed') {
    // Redirecionar para página de sucesso
    window.location.href = '/success';
  } else if (data.payment_status === 'failed') {
    // Mostrar erro
    alert('Pagamento falhou');
  } else {
    // Continuar verificando
    setTimeout(() => checkPaymentStatus(orderId), 5000);
  }
};
```

## Observações Técnicas

1. **Resilência**: Sistema com fallback automático para alta disponibilidade
2. **Precisão**: Cálculos em centavos para evitar problemas de arredondamento
3. **Auditoria**: Logs detalhados de todas as operações
4. **Extensibilidade**: Arquitetura preparada para novas adquirentes
5. **Performance**: Consultas otimizadas e cache de dados
6. **Segurança**: Validação rigorosa de inputs e autenticação opcional
