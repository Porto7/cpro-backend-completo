# 📚 Documentação Completa das Rotas API - CheckoutPro Backend

## 🎯 Visão Geral
Este documento consolida toda a documentação das rotas do sistema CheckoutPro Backend, um sistema completo de e-commerce e gestão de produtos digitais.

## 📋 Índice de Rotas Documentadas

### 🔐 Autenticação e Usuários
- [**auth.js**](./DOCUMENTACAO_ROTAS_AUTH.md) - Sistema de autenticação, login, registro, 2FA
- [**users.js**](./DOCUMENTACAO_ROTAS_USERS.md) - Gerenciamento de perfis de usuário

### 🛍️ E-commerce Core
- [**products.js**](./DOCUMENTACAO_ROTAS_PRODUCTS.md) - CRUD completo de produtos digitais
- [**checkout.js**](./DOCUMENTACAO_ROTAS_CHECKOUT.md) - Sistema de checkout e finalização de compras
- [**orders.js**](./DOCUMENTACAO_ROTAS_ORDERS.md) - Gerenciamento de pedidos e status
- [**transactions.js**](./DOCUMENTACAO_ROTAS_TRANSACTIONS.md) - Histórico e gestão de transações

### 💳 Pagamentos
- [**cardPayment.js**](./DOCUMENTACAO_ROTAS_CARD_PAYMENT.md) - Processamento de cartão de crédito
- [**pix.js**](./DOCUMENTACAO_ROTAS_PIX.md) - Sistema PIX básico
- [**pixPayment.js**](./DOCUMENTACAO_ROTAS_PIX_PAYMENT.md) - Sistema PIX avançado multi-adquirente
- [**wallet.js**](./DOCUMENTACAO_ROTAS_WALLET.md) - Carteira digital e saldo do usuário

### 📊 Analytics e Tracking
- [**analytics.js**](./DOCUMENTACAO_ROTAS_ANALYTICS.md) - Analytics de vendas e conversões
- [**tracking.js**](./DOCUMENTACAO_ROTAS_TRACKING.md) - Sistema de rastreamento de eventos
- [**utm.js**](./DOCUMENTACAO_ROTAS_UTM.md) - Rastreamento de campanhas UTM
- [**facebookTracking.js**](./DOCUMENTACAO_ROTAS_FACEBOOK_TRACKING.md) - Integração com Facebook Pixel

### 🎓 Educação Digital
- [**courses.js**](./DOCUMENTACAO_ROTAS_COURSES.md) - Sistema de cursos online
- [**courseAdmin.js**](./DOCUMENTACAO_ROTAS_COURSE_ADMIN.md) - Administração de cursos
- [**student.js**](./DOCUMENTACAO_ROTAS_STUDENT.md) - Área do aluno e progresso
- [**members.js**](./DOCUMENTACAO_ROTAS_MEMBERS.md) - Sistema de membros e acesso

### 📧 Marketing e Comunicação
- [**email.js**](./DOCUMENTACAO_ROTAS_EMAIL.md) - Sistema de emails transacionais
- [**notifications.js**](./DOCUMENTACAO_ROTAS_NOTIFICATIONS.md) - Sistema de notificações
- [**salesRecovery.js**](./DOCUMENTACAO_ROTAS_SALES_RECOVERY.md) - Recuperação de carrinho abandonado
- [**funnel.js**](./DOCUMENTACAO_ROTAS_FUNNEL.md) - Funis de vendas e conversão

### 🎨 Frontend e Templates
- [**frontend.js**](./DOCUMENTACAO_ROTAS_FRONTEND.md) - Rotas do frontend e SPA
- [**templates.js**](./DOCUMENTACAO_ROTAS_TEMPLATES.md) - Sistema de templates personalizáveis
- [**urlHandlers.js**](./DOCUMENTACAO_ROTAS_URL_HANDLERS.md) - Gerenciamento de URLs e compartilhamento

### 📁 Upload e Mídia
- [**upload.js**](./DOCUMENTACAO_ROTAS_UPLOAD.md) - Sistema de upload de arquivos (vídeos/imagens)

### 🏢 Administração
- [**admin.js**](./DOCUMENTACAO_ROTAS_ADMIN.md) - Painel administrativo
- [**settings.js**](./DOCUMENTACAO_ROTAS_SETTINGS.md) - Configurações do sistema
- [**backup.js**](./DOCUMENTACAO_ROTAS_BACKUP.md) - Sistema de backup
- [**logs.js**](./DOCUMENTACAO_ROTAS_LOGS.md) - Sistema de logs e auditoria

### 🏦 Financeiro Avançado
- [**acquirers.js**](./DOCUMENTACAO_ROTAS_ACQUIRERS.md) - Gestão de adquirentes
- [**plans.js**](./DOCUMENTACAO_ROTAS_PLANS.md) - Planos de assinatura
- [**userPlans.js**](./DOCUMENTACAO_ROTAS_USER_PLANS.md) - Gerenciamento de planos do usuário
- [**subscriptions.js**](./DOCUMENTACAO_ROTAS_SUBSCRIPTIONS.md) - Sistema de assinaturas

### 🤝 Parcerias e Físico
- [**partners.js**](./DOCUMENTACAO_ROTAS_PARTNERS.md) - Sistema de afiliados e parcerias
- [**physical.js**](./DOCUMENTACAO_ROTAS_PHYSICAL.md) - Produtos físicos e entrega

### 🔗 Integrações e Webhooks
- [**webhook.js**](./DOCUMENTACAO_ROTAS_WEBHOOK.md) - Processamento automático de pagamentos e matrículas
- [**webhooks.js**](./DOCUMENTACAO_ROTAS_WEBHOOKS_CORE.md) - CRUD de webhooks do usuário (sistema core)
- [**webhooksManagement.js**](./DOCUMENTACAO_ROTAS_WEBHOOKS_MANAGEMENT.md) - Gerenciamento avançado de webhooks

### 🎮 Funcionalidades Especiais
- [**gamification.js**](./DOCUMENTACAO_ROTAS_GAMIFICATION.md) - Sistema de gamificação
- [**kyc.js**](./DOCUMENTACAO_ROTAS_KYC.md) - Verificação de identidade (KYC)
- [**advanced.js**](./DOCUMENTACAO_ROTAS_ADVANCED.md) - Funcionalidades avançadas

## 📊 Estatísticas do Projeto

### Resumo Técnico
- **Total de Arquivos de Rotas**: 41 arquivos
- **Total de Endpoints**: 200+ endpoints documentados
- **Linhas de Código**: 15,000+ linhas
- **Arquivos de Documentação**: 41 arquivos MD criados

### Funcionalidades Principais
- ✅ Sistema completo de e-commerce
- ✅ Múltiplos métodos de pagamento (Cartão, PIX, Wallets)
- ✅ Plataforma de educação digital
- ✅ Sistema de afiliados e parcerias
- ✅ Analytics avançado e tracking
- ✅ Sistema de templates personalizáveis
- ✅ Gamificação e engajamento
- ✅ Webhooks e integrações
- ✅ Sistema de backup e logs
- ✅ Área administrativa completa

### Tecnologias Utilizadas
- **Backend**: Node.js + Express.js
- **Banco de Dados**: Sequelize ORM (SQLite/PostgreSQL)
- **Autenticação**: JWT + 2FA
- **Pagamentos**: Múltiplos gateways (Stripe, PIX, etc.)
- **Upload**: Multer + FFmpeg (vídeos) + Sharp (imagens)
- **Email**: Sistema transacional completo
- **Cache/Session**: Sistema de sessões
- **Security**: Rate limiting, validações, sanitização

## 🚀 Como Usar Esta Documentação

### Para Desenvolvedores
1. **Navegue pelos arquivos**: Cada rota tem sua documentação específica
2. **Endpoints completos**: Todos os endpoints com exemplos de request/response
3. **Autenticação**: Informações sobre que rotas requerem autenticação
4. **Rate Limiting**: Limites de requisições documentados
5. **Códigos de Erro**: Todos os possíveis retornos de erro

### Para Integrações
1. **Webhooks**: Sistema completo de webhooks para integrações
2. **APIs Públicas**: Algumas rotas são públicas para integrações
3. **Eventos**: Sistema de eventos para tracking
4. **Templates**: APIs para customização de aparência

### Para Administradores
1. **Painel Admin**: Rotas administrativas documentadas
2. **Backup**: Sistema de backup e restore
3. **Logs**: Sistema de auditoria e logs
4. **Configurações**: Todas as configurações do sistema

## 🔧 Setup e Configuração

### Variáveis de Ambiente Principais
```env
# Database
DATABASE_URL=
DB_NAME=
DB_USER=
DB_PASS=

# JWT
JWT_SECRET=

# Payment Gateways
STRIPE_SECRET_KEY=
PIX_CLIENT_ID=
PIX_CLIENT_SECRET=

# Email
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASS=

# Storage
UPLOAD_PATH=
MAX_FILE_SIZE=

# External APIs
FACEBOOK_PIXEL_ID=
GOOGLE_ANALYTICS_ID=
```

### Modelos de Banco Principais
- **User** - Usuários do sistema
- **Product** - Produtos digitais/físicos
- **Order** - Pedidos de compra
- **Payment** - Transações de pagamento
- **Course** - Cursos online
- **Webhook** - Configurações de webhook
- **Template** - Templates personalizáveis
- **Analytics** - Dados de analytics

## 🔐 Autenticação Global

### Headers Requeridos
```javascript
// Para rotas autenticadas
{
  "Authorization": "Bearer <jwt_token>",
  "Content-Type": "application/json"
}
```

### Rate Limiting Global
- **Padrão**: 100 requests por 15 minutos
- **Autenticados**: Limites mais altos
- **Admin**: Sem limitação
- **Webhooks**: Limites específicos por endpoint

## 🚨 Códigos de Status HTTP

### Sucessos
- `200` - OK (operação bem-sucedida)
- `201` - Created (recurso criado)
- `202` - Accepted (operação aceita, processamento assíncrono)

### Erros do Cliente
- `400` - Bad Request (dados inválidos)
- `401` - Unauthorized (autenticação requerida)
- `403` - Forbidden (sem permissão)
- `404` - Not Found (recurso não encontrado)
- `409` - Conflict (conflito de dados)
- `422` - Unprocessable Entity (validação falhou)
- `429` - Too Many Requests (rate limit)

### Erros do Servidor
- `500` - Internal Server Error (erro interno)
- `502` - Bad Gateway (erro de gateway)
- `503` - Service Unavailable (serviço indisponível)

## 📈 Próximos Passos

### Implementações Futuras
- [ ] Sistema de cache Redis
- [ ] Microserviços
- [ ] GraphQL API
- [ ] WebSockets para real-time
- [ ] Sistema de filas (Bull/Bee-Queue)
- [ ] Elasticsearch para busca
- [ ] Monitoramento (Prometheus/Grafana)

### Melhorias Planejadas
- [ ] Documentação OpenAPI/Swagger
- [ ] Testes automatizados (Jest)
- [ ] CI/CD pipeline
- [ ] Docker containers
- [ ] Load balancing
- [ ] CDN integration

## 📞 Suporte

### Contato
- **Documentação**: Este repositório
- **Issues**: GitHub Issues
- **Discussões**: GitHub Discussions

### Contribuição
1. Fork do projeto
2. Criar branch de feature
3. Commit das mudanças
4. Abrir Pull Request
5. Atualizar documentação conforme necessário

---

## ⚡ Quick Start

### 1. Instalar Dependências
```bash
npm install
```

### 2. Configurar Ambiente
```bash
cp env.example .env
# Editar .env com suas configurações
```

### 3. Configurar Banco
```bash
npm run migrate
npm run seed
```

### 4. Iniciar Servidor
```bash
npm start
# ou desenvolvimento
npm run dev
```

### 5. Testar API
```bash
curl http://localhost:3000/api/health
```

---

> **Nota**: Esta é uma documentação viva. Sempre verifique a versão mais recente do código para mudanças específicas.

**Status**: ✅ Documentação Completa  
**Última Atualização**: Janeiro 2024  
**Versão da API**: v1.0  
**Total de Endpoints Documentados**: 200+
