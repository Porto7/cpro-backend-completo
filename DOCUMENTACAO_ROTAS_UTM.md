# Documentação API - Rastreamento UTM

## Visão Geral
Sistema completo de rastreamento de campanhas UTM para análise de origem de tráfego e conversões.

## Endpoints Disponíveis

### 1. Registrar Clique UTM
**POST** `/api/utm/track`

**Autenticação:** Não requerida

**Rate Limit:** 100 requests por minuto (skip para usuários autenticados)

**Descrição:** Registra um clique com parâmetros UTM para rastreamento de campanha.

**Body da Requisição:**
```json
{
  "productId": "prod_123456789",
  "utm_source": "facebook",
  "utm_medium": "social",
  "utm_campaign": "summer_sale_2024",
  "utm_content": "banner_top",
  "utm_term": "desconto"
}
```

**Parâmetros:**
- `productId` (string, obrigatório): ID do produto sendo rastreado
- `utm_source` (string, obrigatório): Origem do tráfego (ex: google, facebook)
- `utm_medium` (string, obrigatório): Meio/canal (ex: cpc, social, email)
- `utm_campaign` (string, obrigatório): Nome da campanha
- `utm_content` (string, opcional): Conteúdo específico ou variação
- `utm_term` (string, opcional): Termo da palavra-chave (para busca paga)

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "UTM tracking registrado com sucesso",
  "data": {
    "id": "utm_987654321",
    "clicks": 15
  }
}
```

**Lógica de Funcionamento:**
- Se combinação UTM já existe: incrementa contador de clicks
- Se é nova combinação: cria novo registro com 1 click
- Todos os parâmetros UTM formam uma chave única

**Erros Possíveis:**
- `400` - Product ID obrigatório
- `400` - utm_source, utm_medium e utm_campaign são obrigatórios
- `404` - Produto não encontrado
- `429` - Rate limit excedido

### 2. Analytics UTM do Produto
**GET** `/api/utm/analytics/:productId`

**Autenticação:** Requerida

**Descrição:** Retorna dados analíticos completos de UTM para um produto específico.

**Parâmetros de URL:**
- `productId` (string): ID do produto

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": {
    "campaigns": [
      {
        "name": "summer_sale_2024",
        "clicks": 1500,
        "conversions": 45,
        "revenue": 13500.00
      },
      {
        "name": "black_friday_2024",
        "clicks": 2300,
        "conversions": 78,
        "revenue": 23400.00
      }
    ],
    "sources": [
      {
        "name": "facebook",
        "clicks": 2100
      },
      {
        "name": "google",
        "clicks": 1700
      }
    ],
    "mediums": [
      {
        "name": "social",
        "clicks": 2100
      },
      {
        "name": "cpc",
        "clicks": 1700
      }
    ],
    "summary": {
      "total_clicks": 3800,
      "total_conversions": 123,
      "total_revenue": 36900.00,
      "conversion_rate": "3.24"
    },
    "raw_data": [
      {
        "id": "utm_123",
        "product_id": "prod_123456789",
        "utm_source": "facebook",
        "utm_medium": "social",
        "utm_campaign": "summer_sale_2024",
        "utm_content": "banner_top",
        "utm_term": "desconto",
        "clicks": 750,
        "conversions": 22,
        "revenue": 6600.00,
        "created_at": "2024-01-15T10:30:00Z",
        "updated_at": "2024-01-20T14:45:00Z"
      }
    ]
  }
}
```

**Estrutura dos Dados:**
- **campaigns**: Agrupamento por nome da campanha
- **sources**: Agrupamento por origem do tráfego
- **mediums**: Agrupamento por canal/meio
- **summary**: Estatísticas consolidadas
- **raw_data**: Dados detalhados de cada combinação UTM

**Cálculos Automáticos:**
- `conversion_rate`: (conversions / clicks) * 100
- Ordenação por clicks (decrescente)
- Somatórias automáticas por agrupamento

### 3. Registrar Conversão
**POST** `/api/utm/conversion`

**Autenticação:** Não requerida

**Descrição:** Registra uma conversão para uma combinação específica de parâmetros UTM.

**Body da Requisição:**
```json
{
  "productId": "prod_123456789",
  "utm_source": "facebook",
  "utm_medium": "social",
  "utm_campaign": "summer_sale_2024",
  "utm_content": "banner_top",
  "utm_term": "desconto",
  "revenue": 300.00
}
```

**Parâmetros:**
- `productId` (string, obrigatório): ID do produto
- `utm_source`, `utm_medium`, `utm_campaign` (strings, obrigatórios): Parâmetros UTM
- `utm_content`, `utm_term` (strings, opcionais): Parâmetros UTM adicionais
- `revenue` (number, opcional): Valor da conversão em reais

**Resposta Registro Existente (200):**
```json
{
  "success": true,
  "message": "Conversão registrada com sucesso",
  "data": {
    "id": "utm_987654321",
    "conversions": 23,
    "revenue": 6900.00
  }
}
```

**Resposta Novo Registro (200):**
```json
{
  "success": true,
  "message": "Nova conversão UTM criada",
  "data": {
    "id": "utm_new123",
    "conversions": 1,
    "revenue": 300.00
  }
}
```

**Lógica de Funcionamento:**
- Se combinação UTM existe: incrementa conversions e soma revenue
- Se não existe: cria novo registro com clicks=0, conversions=1
- Permite conversões sem clicks prévios registrados

### 4. Limpar Dados UTM
**DELETE** `/api/utm/analytics/:productId`

**Autenticação:** Requerida

**Descrição:** Remove todos os dados de rastreamento UTM de um produto.

**Parâmetros de URL:**
- `productId` (string): ID do produto

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Dados UTM limpos com sucesso",
  "data": {
    "deleted_records": 15
  }
}
```

**Comportamento:**
- Remove todos os registros UTM do produto
- Operação irreversível
- Retorna quantidade de registros deletados

## Modelo de Dados UTM

### Estrutura do Registro
```javascript
const UtmTracking = {
  id: 'uuid',
  product_id: 'string',
  utm_source: 'string',      // Obrigatório
  utm_medium: 'string',      // Obrigatório  
  utm_campaign: 'string',    // Obrigatório
  utm_content: 'string',     // Opcional
  utm_term: 'string',        // Opcional
  clicks: 'integer',         // Contador de clicks
  conversions: 'integer',    // Contador de conversões
  revenue: 'decimal',        // Valor total de conversões
  created_at: 'timestamp',
  updated_at: 'timestamp'
};
```

### Chave Única
A combinação única é formada por:
- `product_id`
- `utm_source`
- `utm_medium`
- `utm_campaign`
- `utm_content` (ou null)
- `utm_term` (ou null)

## Implementação do Rate Limiting

### Configuração
```javascript
const utmLimiter = rateLimit({
  windowMs: 1 * 60 * 1000,     // 1 minuto
  max: 100,                    // 100 requests
  message: {
    error: 'Muitas tentativas de tracking. Tente novamente em 1 minuto.',
    type: 'utm_rate_limit'
  },
  skip: (req) => {
    // Skip para usuários autenticados
    return req.headers.authorization && 
           req.headers.authorization.startsWith('Bearer ');
  }
});
```

### Aplicação
- Aplicado apenas no endpoint `/track`
- Usuários autenticados não sofrem limitação
- Proteção contra abuso de tracking

## Relatórios e Analytics

### Dashboard de Campanhas
```javascript
// Exemplo de uso dos dados para dashboard
const createCampaignDashboard = async (productId) => {
  const response = await fetch(`/api/utm/analytics/${productId}`, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  
  const data = await response.json();
  
  // Top campanhas por conversão
  const topCampaigns = data.data.campaigns
    .sort((a, b) => b.conversions - a.conversions)
    .slice(0, 5);
  
  // ROI por campanha
  const campaignROI = data.data.campaigns.map(campaign => ({
    name: campaign.name,
    roi: campaign.revenue > 0 ? (campaign.revenue / campaign.clicks).toFixed(2) : '0.00',
    conversion_rate: ((campaign.conversions / campaign.clicks) * 100).toFixed(2)
  }));
  
  return {
    topCampaigns,
    campaignROI,
    summary: data.data.summary
  };
};
```

### Comparação de Fontes
```javascript
// Análise de performance por fonte
const analyzeTrafficSources = (analyticsData) => {
  const sources = analyticsData.data.sources.map(source => {
    // Encontrar dados detalhados desta fonte
    const sourceData = analyticsData.data.raw_data
      .filter(item => item.utm_source === source.name)
      .reduce((acc, item) => {
        acc.conversions += item.conversions;
        acc.revenue += parseFloat(item.revenue || 0);
        return acc;
      }, { conversions: 0, revenue: 0 });
    
    return {
      name: source.name,
      clicks: source.clicks,
      conversions: sourceData.conversions,
      revenue: sourceData.revenue,
      ctr: ((sourceData.conversions / source.clicks) * 100).toFixed(2),
      revenue_per_click: (sourceData.revenue / source.clicks).toFixed(2)
    };
  });
  
  return sources.sort((a, b) => b.revenue - a.revenue);
};
```

## Integração com Analytics

### Rastreamento Automático
```javascript
// Função para rastrear automaticamente clicks
const trackUTMClick = async (productId, utmParams) => {
  try {
    await fetch('/api/utm/track', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        productId,
        ...utmParams
      })
    });
  } catch (error) {
    console.error('Erro ao rastrear UTM:', error);
  }
};

// Extrair parâmetros UTM da URL
const extractUTMParams = () => {
  const urlParams = new URLSearchParams(window.location.search);
  return {
    utm_source: urlParams.get('utm_source'),
    utm_medium: urlParams.get('utm_medium'),
    utm_campaign: urlParams.get('utm_campaign'),
    utm_content: urlParams.get('utm_content'),
    utm_term: urlParams.get('utm_term')
  };
};

// Uso automático no carregamento da página
document.addEventListener('DOMContentLoaded', () => {
  const utmParams = extractUTMParams();
  const productId = document.querySelector('[data-product-id]')?.dataset.productId;
  
  if (utmParams.utm_source && utmParams.utm_medium && utmParams.utm_campaign && productId) {
    trackUTMClick(productId, utmParams);
  }
});
```

### Rastreamento de Conversões
```javascript
// Rastrear conversão após compra
const trackConversion = async (orderData) => {
  const utmParams = JSON.parse(localStorage.getItem('utm_params') || '{}');
  
  if (utmParams.utm_source) {
    try {
      await fetch('/api/utm/conversion', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          productId: orderData.productId,
          revenue: orderData.total,
          ...utmParams
        })
      });
    } catch (error) {
      console.error('Erro ao rastrear conversão:', error);
    }
  }
};
```

## Parâmetros UTM Padrão

### Fontes Comuns
- `google` - Tráfego do Google
- `facebook` - Facebook/Instagram
- `instagram` - Instagram específico
- `youtube` - Canal do YouTube
- `email` - Campanhas de email
- `direct` - Acesso direto
- `referral` - Sites parceiros

### Meios/Canais Comuns
- `cpc` - Custo por clique (ads pagos)
- `social` - Redes sociais orgânicas
- `email` - Email marketing
- `organic` - Busca orgânica
- `referral` - Referência de outros sites
- `display` - Banners e display ads

### Boas Práticas
1. **Consistência**: Use nomenclatura padronizada
2. **Minúsculas**: Mantenha tudo em lowercase
3. **Sem espaços**: Use underscore ou hífen
4. **Descritivo**: Nomes autoexplicativos
5. **Único**: Cada campanha deve ter nome único

## Observações Importantes

1. **Performance**: Registros UTM são consultados frequentemente, use índices apropriados
2. **Limpeza**: Considere limpeza periódica de dados antigos
3. **Privacidade**: Não armazene informações pessoais nos parâmetros UTM
4. **Rate Limiting**: Proteção contra abuso mantendo funcionalidade
5. **Backup**: Dados importantes para análise de ROI
6. **Integrações**: Facilmente integrável com Google Analytics e outras ferramentas
