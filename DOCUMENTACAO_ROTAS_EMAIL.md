# 📧 Documentação Completa - Rotas de Email

## Visão Geral
O arquivo `routes/email.js` contém rotas específicas para funcionalidades de email, incluindo verificação de status do serviço, envio de emails de teste e notificações automáticas de produtos. O sistema é baseado no Nodemailer e integrado com SMTP.

## Configuração do Sistema
- **Serviço**: Nodemailer com SMTP
- **Host Padrão**: smtp.hostinger.com
- **Porta**: 465 (SSL) ou configurável via variáveis
- **Segurança**: SSL/TLS automático
- **From Email**: Configurável via variáveis de ambiente

## Funcionalidades
- **Status do Serviço**: Verificação de configuração SMTP
- **Emails de Teste**: Envio em ambiente de desenvolvimento
- **Notificações de Produto**: Emails automáticos para vendedores

---

## 🔍 STATUS DO SERVIÇO

### GET /api/email/status
**Descrição**: Verifica o status e configuração do serviço de email

**Autenticação**: Não obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "configured": true,
  "service": "nodemailer",
  "host": "smtp.hostinger.com",
  "port": 465,
  "secure": true,
  "from": "noreply@seudominio.com"
}
```

**Campos de Resposta**:
- `configured`: Se o serviço está configurado corretamente
- `service`: Tipo de serviço (sempre "nodemailer")
- `host`: Servidor SMTP configurado
- `port`: Porta SMTP (465 para SSL, 587 para TLS)
- `secure`: Se está usando SSL/TLS
- `from`: Email remetente configurado

**Configuração via Variáveis**:
```env
SMTP_HOST=smtp.hostinger.com
SMTP_PORT=465
SMTP_SECURE=true
SMTP_USER=seu-email@dominio.com
SMTP_PASS=sua-senha
FROM_EMAIL=noreply@dominio.com
```

**Uso**: Verificar se emails podem ser enviados

**Erros Possíveis**:
- `500`: Erro interno do servidor

---

## 🧪 TESTE DE EMAIL (DESENVOLVIMENTO)

### POST /api/email/test-send
**Descrição**: Envia email de teste para validação do sistema

**Restrição**: Disponível apenas em ambiente de desenvolvimento

**Body Parameters**:
```json
{
  "to": "teste@email.com",
  "type": "verification"
}
```

**Parâmetros**:
- `to` (string, obrigatório): Email de destino
- `type` (string, opcional): Tipo de email
  - `verification`: Email de verificação (padrão)
  - `password_recovery`: Recuperação de senha
  - `welcome`: Email de boas-vindas

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Email de teste (verification) enviado com sucesso",
  "to": "teste@email.com",
  "type": "verification",
  "result": {
    "messageId": "<msg-id@servidor.com>",
    "response": "250 Message accepted"
  }
}
```

**Resposta de Erro - Ambiente Incorreto (403)**:
```json
{
  "error": "Endpoint disponível apenas em desenvolvimento",
  "type": "access_denied"
}
```

**Resposta de Erro - Validação (400)**:
```json
{
  "error": "Email de destino é obrigatório",
  "type": "validation_error"
}
```

**Tipos de Email Disponíveis**:

1. **Verification** (`verification`):
   - Email de verificação de conta
   - Link de confirmação incluído
   - Template padrão do sistema

2. **Password Recovery** (`password_recovery`):
   - Email de recuperação de senha
   - Token de reset incluído
   - Instruções de redefinição

3. **Welcome** (`welcome`):
   - Email de boas-vindas
   - Informações da plataforma
   - Próximos passos

**Uso**: Testar configuração SMTP e templates

**Erros Possíveis**:
- `403`: Não está em desenvolvimento
- `400`: Parâmetros inválidos
- `500`: Erro no envio do email

---

## 📧 NOTIFICAÇÃO DE PRODUTO

### POST /api/email/send-product-notification
**Descrição**: Envia notificação automática quando produto é criado

**Autenticação**: Não obrigatória (endpoint interno)

**Body Parameters**:
```json
{
  "sellerEmail": "vendedor@email.com",
  "sellerName": "João Silva",
  "product": {
    "id": "uuid-produto-123",
    "title": "Curso de JavaScript Avançado",
    "price": 197.50
  }
}
```

**Parâmetros Obrigatórios**:
- `sellerEmail` (string): Email do vendedor
- `sellerName` (string): Nome do vendedor
- `product` (object): Dados do produto
  - `title|name|productName` (string): Nome do produto
  - `price|valor` (number, opcional): Preço do produto
  - `id|productId` (string, opcional): ID do produto

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Notificação de produto enviada com sucesso",
  "data": {
    "seller_email": "vendedor@email.com",
    "product_id": "uuid-produto-123",
    "product_title": "Curso de JavaScript Avançado",
    "product_price": 197.50,
    "sent_at": "2024-12-31T18:30:00.000Z"
  }
}
```

**Template do Email**:
```html
<!-- Email responsivo com design profissional -->
<div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
  
  <!-- Header -->
  <h1 style="color: #3B82F6;">🎉 Produto Criado!</h1>
  
  <!-- Saudação personalizada -->
  <p>Olá <strong>João Silva</strong>,</p>
  <p>Seu produto foi criado com sucesso em nossa plataforma! 🚀</p>
  
  <!-- Detalhes do produto -->
  <div style="background: #f8f9fa; padding: 25px; border-radius: 8px;">
    <h2>📦 Detalhes do Produto</h2>
    <p><strong>Nome:</strong> Curso de JavaScript Avançado</p>
    <p><strong>Preço:</strong> R$ 197,50</p>
    <p><strong>ID:</strong> uuid-produto-123</p>
  </div>
  
  <!-- Próximos passos -->
  <h3>🎯 Próximos Passos</h3>
  <ul>
    <li>Configure sua página de checkout personalizada</li>
    <li>Adicione pixels de tracking para analytics</li>
    <li>Compartilhe o link do produto com seus clientes</li>
    <li>Monitore as vendas no painel administrativo</li>
  </ul>
  
  <!-- Botão de ação -->
  <a href="http://localhost:3000/admin/products" 
     style="background: #3B82F6; color: white; padding: 15px 30px;">
    Ver Produtos no Painel
  </a>
  
</div>
```

**Funcionalidades Avançadas**:

1. **Tolerância a Dados**:
   - Aceita diferentes nomes para campos do produto
   - Valores padrão para campos opcionais
   - Validação flexível de entrada

2. **Debug Detalhado**:
   - Logs completos de dados recebidos
   - Rastreamento de processamento
   - Identificação de problemas

3. **Template Responsivo**:
   - Design mobile-friendly
   - Cores e tipografia profissionais
   - Call-to-action destacado

**Mapeamento de Campos**:
```javascript
// Nomes aceitos para o produto
const productId = product.id || product.productId || `temp-${Date.now()}`;
const productTitle = product.title || product.name || product.productName || 'Produto sem nome';
const productPrice = product.price !== undefined ? product.price : 
                    (product.valor !== undefined ? product.valor : 0);
```

**Resposta de Erro - Validação (400)**:
```json
{
  "success": false,
  "message": "sellerEmail e sellerName são obrigatórios",
  "received": {
    "sellerEmail": null,
    "sellerName": null
  }
}
```

**Resposta de Erro - Produto Inválido (400)**:
```json
{
  "success": false,
  "message": "Produto deve ter um nome (title, name ou productName)",
  "received": {
    "title": null,
    "name": null,
    "productName": null,
    "fullProduct": {}
  }
}
```

**Uso**: Automação de notificações quando produtos são criados

**Erros Possíveis**:
- `400`: Dados obrigatórios ausentes
- `500`: Erro no envio do email

---

## 🔧 Configuração Técnica

### Variáveis de Ambiente Necessárias
```env
# SMTP Configuration
SMTP_HOST=smtp.hostinger.com
SMTP_PORT=465
SMTP_SECURE=true
SMTP_USER=seu-email@dominio.com
SMTP_PASS=sua-senha-app

# Email Settings
FROM_EMAIL=noreply@seudominio.com
FROM_NAME=Sua Plataforma

# Frontend URL (para links nos emails)
FRONTEND_URL=https://seusite.com

# Environment
NODE_ENV=production
```

### Dependências
```javascript
import express from 'express';
import { emailService } from '../services/emailService.js';
```

### Configuração SMTP Recomendada
```javascript
// Para Hostinger
SMTP_HOST=smtp.hostinger.com
SMTP_PORT=465
SMTP_SECURE=true

// Para Gmail
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=false

// Para SendGrid
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_SECURE=false
```

---

## 📊 Monitoramento e Logs

### Logs de Debug
```javascript
console.log('📧 [DEBUG] Recebendo notificação de produto:', JSON.stringify(req.body, null, 2));
console.log('📧 [DEBUG] Valores recebidos:', {
  sellerEmail: sellerEmail,
  sellerName: sellerName,
  product: product
});
```

### Tratamento de Erros
- Logs detalhados para debugging
- Mensagens de erro específicas
- Fallbacks para dados ausentes
- Validação robusta de entrada

### Performance
- Templates HTML otimizados
- Envio assíncrono de emails
- Timeouts configuráveis
- Retry automático em caso de falha

---

## 🎯 Casos de Uso

### Para Desenvolvedores
- Testar configuração SMTP em desenvolvimento
- Validar templates de email
- Debug de problemas de entrega
- Configurar novos provedores

### Para Vendedores
- Receber confirmação de produto criado
- Instruções de próximos passos
- Links diretos para painel administrativo
- Suporte visual com design profissional

### Para Sistema
- Notificações automáticas
- Integração com criação de produtos
- Tracking de emails enviados
- Logs para auditoria

---

## 🚀 Melhorias Futuras

### Templates Avançados
- Sistema de templates configuráveis
- Personalizações por vendedor
- A/B testing de templates
- Multi-idioma

### Analytics de Email
- Taxa de abertura
- Taxa de clique
- Bounce rate
- Unsubscribe tracking

### Automações
- Sequências de email marketing
- Triggers baseados em ações
- Segmentação de usuários
- Campanhas sazonais

---

**Próxima documentação**: `routes/webhook.js` - Sistema de webhooks e integrações
