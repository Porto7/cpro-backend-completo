# Documentação - Sistema de Planos (routes/plans.js)

## Visão Geral
Sistema de gerenciamento de planos de assinatura da plataforma. Define os diferentes níveis de serviço, características incluídas, preços e opções de pagamento.

## Características Principais
- **Múltiplos Planos**: Básico, Profissional e Empresarial
- **Preços Flexíveis**: Opções mensais e anuais com desconto
- **Features Detalhadas**: Lista completa de recursos por plano
- **Plano Popular**: Destacar o plano recomendado
- **Estrutura Escalável**: Fácil adição de novos planos

## Planos Disponíveis

### Básico (R$ 29,90/mês)
- **Público-alvo**: Iniciantes e pequenos negócios
- **Características**:
  - Até 100 produtos
  - Checkout básico
  - Suporte por email
  - Relatórios básicos

### Profissional (R$ 59,90/mês) - **POPULAR**
- **Público-alvo**: Negócios em crescimento
- **Características**:
  - Produtos ilimitados
  - Checkout avançado
  - Suporte prioritário
  - Relatórios avançados
  - Integrações
  - Domínio personalizado

### Empresarial (R$ 149,90/mês)
- **Público-alvo**: Grandes empresas
- **Características**:
  - Tudo do Profissional
  - API completa
  - Suporte 24/7
  - Manager dedicado
  - Customizações
  - SLA garantido

## Opções de Pagamento

### Mensal
- **Descrição**: Cobrança mensal recorrente
- **Desconto**: Nenhum
- **Flexibilidade**: Cancelar a qualquer momento

### Anual
- **Descrição**: Pagamento único anual
- **Desconto**: 17% (equivale a 2 meses grátis)
- **Economia**: Melhor custo-benefício

## Rotas Implementadas

### 1. Listar Todos os Planos

#### `GET /api/plans`
**Descrição**: Obter todos os planos disponíveis com detalhes completos
**Acesso**: Público

**Response Success**:
```json
{
  "success": true,
  "plans": [
    {
      "id": "basic",
      "name": "Básico",
      "description": "Ideal para começar",
      "price": 29.90,
      "currency": "BRL",
      "features": [
        "Até 100 produtos",
        "Checkout básico",
        "Suporte por email",
        "Relatórios básicos"
      ],
      "popular": false,
      "paymentPlans": [
        {
          "id": "monthly",
          "name": "Mensal",
          "price": 29.90,
          "discount": 0,
          "billingCycle": "monthly"
        },
        {
          "id": "yearly",
          "name": "Anual",
          "price": 299.00,
          "discount": 17,
          "billingCycle": "yearly"
        }
      ]
    },
    {
      "id": "pro",
      "name": "Profissional",
      "description": "Para negócios em crescimento",
      "price": 59.90,
      "currency": "BRL",
      "features": [
        "Produtos ilimitados",
        "Checkout avançado",
        "Suporte prioritário",
        "Relatórios avançados",
        "Integrações",
        "Domínio personalizado"
      ],
      "popular": true,
      "paymentPlans": [
        {
          "id": "monthly",
          "name": "Mensal",
          "price": 59.90,
          "discount": 0,
          "billingCycle": "monthly"
        },
        {
          "id": "yearly",
          "name": "Anual",
          "price": 599.00,
          "discount": 17,
          "billingCycle": "yearly"
        }
      ]
    },
    {
      "id": "enterprise",
      "name": "Empresarial",
      "description": "Para grandes empresas",
      "price": 149.90,
      "currency": "BRL",
      "features": [
        "Tudo do Profissional",
        "API completa",
        "Suporte 24/7",
        "Manager dedicado",
        "Customizações",
        "SLA garantido"
      ],
      "popular": false,
      "paymentPlans": [
        {
          "id": "monthly",
          "name": "Mensal",
          "price": 149.90,
          "discount": 0,
          "billingCycle": "monthly"
        },
        {
          "id": "yearly",
          "name": "Anual",
          "price": 1499.00,
          "discount": 17,
          "billingCycle": "yearly"
        }
      ]
    }
  ],
  "total": 3
}
```

### 2. Obter Plano Específico

#### `GET /api/plans/:planId`
**Descrição**: Obter detalhes de um plano específico
**Acesso**: Público

**Parâmetros**:
- `planId`: ID do plano (basic, pro, enterprise)

**Response Success - Plano Básico**:
```json
{
  "success": true,
  "plan": {
    "id": "basic",
    "name": "Básico",
    "description": "Ideal para começar",
    "price": 29.90,
    "currency": "BRL",
    "features": [
      "Até 100 produtos",
      "Checkout básico",
      "Suporte por email",
      "Relatórios básicos"
    ],
    "popular": false
  }
}
```

**Response Success - Plano Profissional**:
```json
{
  "success": true,
  "plan": {
    "id": "pro",
    "name": "Profissional",
    "description": "Para negócios em crescimento",
    "price": 59.90,
    "currency": "BRL",
    "features": [
      "Produtos ilimitados",
      "Checkout avançado",
      "Suporte prioritário",
      "Relatórios avançados",
      "Integrações",
      "Domínio personalizado"
    ],
    "popular": true
  }
}
```

**Response Success - Plano Empresarial**:
```json
{
  "success": true,
  "plan": {
    "id": "enterprise",
    "name": "Empresarial",
    "description": "Para grandes empresas",
    "price": 149.90,
    "currency": "BRL",
    "features": [
      "Tudo do Profissional",
      "API completa",
      "Suporte 24/7",
      "Manager dedicado",
      "Customizações",
      "SLA garantido"
    ],
    "popular": false
  }
}
```

**Response Error - Plano não encontrado**:
```json
{
  "error": "Plano não encontrado",
  "type": "not_found_error"
}
```

## Integração Frontend

### Componente de Seleção de Planos
```jsx
import React, { useState, useEffect } from 'react';

const PricingPlans = ({ onSelectPlan }) => {
  const [plans, setPlans] = useState([]);
  const [billingCycle, setBillingCycle] = useState('monthly');
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    loadPlans();
  }, []);
  
  const loadPlans = async () => {
    try {
      const response = await fetch('/api/plans');
      const data = await response.json();
      setPlans(data.plans);
    } catch (error) {
      console.error('Erro ao carregar planos:', error);
    } finally {
      setLoading(false);
    }
  };
  
  const getPlanPrice = (plan) => {
    const paymentPlan = plan.paymentPlans.find(p => p.billingCycle === billingCycle);
    return paymentPlan || plan.paymentPlans[0];
  };
  
  const getDiscountText = (plan) => {
    const yearlyPlan = plan.paymentPlans.find(p => p.billingCycle === 'yearly');
    if (billingCycle === 'yearly' && yearlyPlan && yearlyPlan.discount > 0) {
      return `${yearlyPlan.discount}% OFF`;
    }
    return null;
  };
  
  if (loading) {
    return <div className="pricing-loading">Carregando planos...</div>;
  }
  
  return (
    <div className="pricing-plans">
      <div className="pricing-header">
        <h2>🚀 Escolha seu Plano</h2>
        <p>Encontre o plano perfeito para o seu negócio</p>
        
        <div className="billing-toggle">
          <label className={billingCycle === 'monthly' ? 'active' : ''}>
            <input
              type="radio"
              name="billing"
              value="monthly"
              checked={billingCycle === 'monthly'}
              onChange={(e) => setBillingCycle(e.target.value)}
            />
            Mensal
          </label>
          <label className={billingCycle === 'yearly' ? 'active' : ''}>
            <input
              type="radio"
              name="billing"
              value="yearly"
              checked={billingCycle === 'yearly'}
              onChange={(e) => setBillingCycle(e.target.value)}
            />
            Anual
            <span className="discount-badge">2 meses grátis!</span>
          </label>
        </div>
      </div>
      
      <div className="plans-grid">
        {plans.map(plan => {
          const currentPrice = getPlanPrice(plan);
          const discountText = getDiscountText(plan);
          
          return (
            <div 
              key={plan.id} 
              className={`plan-card ${plan.popular ? 'popular' : ''}`}
            >
              {plan.popular && (
                <div className="popular-badge">✨ Mais Popular</div>
              )}
              
              <div className="plan-header">
                <h3>{plan.name}</h3>
                <p className="plan-description">{plan.description}</p>
              </div>
              
              <div className="plan-pricing">
                <div className="price">
                  <span className="currency">R$</span>
                  <span className="amount">{currentPrice.price.toFixed(2)}</span>
                  <span className="period">
                    /{billingCycle === 'monthly' ? 'mês' : 'ano'}
                  </span>
                </div>
                
                {discountText && (
                  <div className="discount-text">{discountText}</div>
                )}
                
                {billingCycle === 'yearly' && (
                  <div className="monthly-equivalent">
                    R$ {(currentPrice.price / 12).toFixed(2)}/mês
                  </div>
                )}
              </div>
              
              <div className="plan-features">
                <ul>
                  {plan.features.map((feature, index) => (
                    <li key={index}>
                      <span className="check-icon">✅</span>
                      {feature}
                    </li>
                  ))}
                </ul>
              </div>
              
              <div className="plan-action">
                <button 
                  onClick={() => onSelectPlan(plan, currentPrice)}
                  className={`btn ${plan.popular ? 'btn-primary' : 'btn-secondary'}`}
                >
                  {plan.popular ? '🎯 Escolher Plano' : 'Selecionar'}
                </button>
              </div>
            </div>
          );
        })}
      </div>
      
      <div className="pricing-footer">
        <p>💳 Todos os planos incluem 7 dias de teste grátis</p>
        <p>🔒 Cancele a qualquer momento</p>
        <p>💬 Suporte especializado incluído</p>
      </div>
    </div>
  );
};
```

### Hook para Gerenciamento de Planos
```jsx
const usePlans = () => {
  const [plans, setPlans] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const loadPlans = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch('/api/plans');
      const data = await response.json();
      
      if (data.success) {
        setPlans(data.plans);
      } else {
        throw new Error('Erro ao carregar planos');
      }
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  const getPlan = async (planId) => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(`/api/plans/${planId}`);
      const data = await response.json();
      
      if (data.success) {
        return data.plan;
      } else {
        throw new Error(data.error || 'Plano não encontrado');
      }
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  const calculateSavings = (plan) => {
    const monthlyPrice = plan.paymentPlans.find(p => p.billingCycle === 'monthly')?.price || 0;
    const yearlyPrice = plan.paymentPlans.find(p => p.billingCycle === 'yearly')?.price || 0;
    
    if (monthlyPrice && yearlyPrice) {
      const monthlyTotal = monthlyPrice * 12;
      const savings = monthlyTotal - yearlyPrice;
      const savingsPercentage = (savings / monthlyTotal) * 100;
      
      return {
        amount: savings,
        percentage: Math.round(savingsPercentage),
        monthlyTotal,
        yearlyPrice
      };
    }
    
    return null;
  };
  
  const comparePlans = (planA, planB) => {
    return {
      priceDifference: planB.price - planA.price,
      featureDifference: planB.features.length - planA.features.length
    };
  };
  
  return {
    plans,
    loading,
    error,
    loadPlans,
    getPlan,
    calculateSavings,
    comparePlans
  };
};
```

### Componente de Comparação de Planos
```jsx
const PlanComparison = () => {
  const { plans, loadPlans } = usePlans();
  const [selectedPlans, setSelectedPlans] = useState(['basic', 'pro']);
  
  useEffect(() => {
    loadPlans();
  }, []);
  
  const features = [
    { id: 'products', name: 'Produtos', basic: 'Até 100', pro: 'Ilimitados', enterprise: 'Ilimitados' },
    { id: 'checkout', name: 'Checkout', basic: 'Básico', pro: 'Avançado', enterprise: 'Avançado' },
    { id: 'support', name: 'Suporte', basic: 'Email', pro: 'Prioritário', enterprise: '24/7' },
    { id: 'reports', name: 'Relatórios', basic: 'Básicos', pro: 'Avançados', enterprise: 'Avançados' },
    { id: 'integrations', name: 'Integrações', basic: '❌', pro: '✅', enterprise: '✅' },
    { id: 'domain', name: 'Domínio', basic: '❌', pro: '✅', enterprise: '✅' },
    { id: 'api', name: 'API', basic: '❌', pro: 'Limitada', enterprise: 'Completa' },
    { id: 'manager', name: 'Manager', basic: '❌', pro: '❌', enterprise: '✅' },
    { id: 'sla', name: 'SLA', basic: '❌', pro: '❌', enterprise: '✅' }
  ];
  
  return (
    <div className="plan-comparison">
      <h2>📊 Compare os Planos</h2>
      
      <div className="comparison-table">
        <table>
          <thead>
            <tr>
              <th>Recursos</th>
              {plans.map(plan => (
                <th key={plan.id} className={plan.popular ? 'popular' : ''}>
                  <div className="plan-header">
                    <h3>{plan.name}</h3>
                    <div className="price">
                      R$ {plan.price.toFixed(2)}/mês
                    </div>
                    {plan.popular && <span className="badge">Popular</span>}
                  </div>
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {features.map(feature => (
              <tr key={feature.id}>
                <td className="feature-name">{feature.name}</td>
                <td className="feature-value">{feature.basic}</td>
                <td className="feature-value">{feature.pro}</td>
                <td className="feature-value">{feature.enterprise}</td>
              </tr>
            ))}
          </tbody>
          <tfoot>
            <tr>
              <td></td>
              {plans.map(plan => (
                <td key={plan.id}>
                  <button className={`btn ${plan.popular ? 'btn-primary' : 'btn-secondary'}`}>
                    Escolher {plan.name}
                  </button>
                </td>
              ))}
            </tr>
          </tfoot>
        </table>
      </div>
    </div>
  );
};
```

### Calculadora de Economia
```jsx
const SavingsCalculator = ({ plans }) => {
  const [selectedPlan, setSelectedPlan] = useState('pro');
  
  const calculateYearlySavings = (plan) => {
    const monthlyPrice = plan.paymentPlans.find(p => p.billingCycle === 'monthly')?.price || 0;
    const yearlyPrice = plan.paymentPlans.find(p => p.billingCycle === 'yearly')?.price || 0;
    
    const monthlyTotal = monthlyPrice * 12;
    const savings = monthlyTotal - yearlyPrice;
    const percentage = Math.round((savings / monthlyTotal) * 100);
    
    return { savings, percentage, monthlyTotal, yearlyPrice };
  };
  
  const selectedPlanData = plans.find(p => p.id === selectedPlan);
  const savingsData = selectedPlanData ? calculateYearlySavings(selectedPlanData) : null;
  
  return (
    <div className="savings-calculator">
      <h3>💰 Calculadora de Economia</h3>
      
      <div className="plan-selector">
        <label>Selecione um plano:</label>
        <select 
          value={selectedPlan}
          onChange={(e) => setSelectedPlan(e.target.value)}
        >
          {plans.map(plan => (
            <option key={plan.id} value={plan.id}>
              {plan.name}
            </option>
          ))}
        </select>
      </div>
      
      {savingsData && (
        <div className="savings-breakdown">
          <div className="savings-item">
            <span className="label">Pagamento mensal (12x):</span>
            <span className="value">R$ {savingsData.monthlyTotal.toFixed(2)}</span>
          </div>
          
          <div className="savings-item">
            <span className="label">Pagamento anual (1x):</span>
            <span className="value">R$ {savingsData.yearlyPrice.toFixed(2)}</span>
          </div>
          
          <div className="savings-item highlight">
            <span className="label">Sua economia:</span>
            <span className="value">
              R$ {savingsData.savings.toFixed(2)} ({savingsData.percentage}%)
            </span>
          </div>
          
          <div className="savings-visual">
            <div className="progress-bar">
              <div 
                className="progress-fill"
                style={{ width: `${savingsData.percentage}%` }}
              ></div>
            </div>
            <p>Equivale a {Math.round(savingsData.savings / (savingsData.monthlyTotal / 12))} meses grátis!</p>
          </div>
        </div>
      )}
    </div>
  );
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 404 | Plano não encontrado |
| 500 | Erro interno do servidor |

## Considerações de Implementação

### Estado Atual
- **Dados Mockados**: Atualmente usa dados estáticos
- **Estrutura Pronta**: Preparado para integração com banco de dados
- **API Simples**: Endpoints básicos para listagem e detalhes

### Próximos Passos Recomendados
1. **Modelo de Banco**: Criar tabelas para planos e features
2. **Admin Panel**: Interface para gerenciar planos
3. **Assinaturas**: Integração com sistema de pagamentos recorrentes
4. **Trials**: Implementar períodos de teste
5. **Upgrades/Downgrades**: Sistema de mudança de planos

### Estrutura de Banco Sugerida
```sql
-- Tabela de planos
CREATE TABLE plans (
  id VARCHAR(50) PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  price DECIMAL(10,2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'BRL',
  popular BOOLEAN DEFAULT FALSE,
  active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela de features por plano
CREATE TABLE plan_features (
  id INT AUTO_INCREMENT PRIMARY KEY,
  plan_id VARCHAR(50),
  feature_name VARCHAR(200) NOT NULL,
  feature_value TEXT,
  display_order INT DEFAULT 0,
  FOREIGN KEY (plan_id) REFERENCES plans(id)
);

-- Tabela de opções de pagamento
CREATE TABLE plan_payment_options (
  id INT AUTO_INCREMENT PRIMARY KEY,
  plan_id VARCHAR(50),
  billing_cycle ENUM('monthly', 'yearly', 'quarterly') NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  discount_percentage INT DEFAULT 0,
  FOREIGN KEY (plan_id) REFERENCES plans(id)
);
```

## Melhores Práticas

1. **Preços Transparentes**: Sempre mostrar valor total e economia
2. **Destaque Popular**: Guiar usuários para o plano recomendado
3. **Trial Gratuito**: Oferecer teste sem compromisso
4. **Comparação Fácil**: Facilitar comparação entre planos
5. **Upgrade Path**: Tornar fácil a evolução entre planos

Este sistema de planos oferece uma base sólida para monetização da plataforma, com estrutura escalável e interface amigável para conversão de usuários.
