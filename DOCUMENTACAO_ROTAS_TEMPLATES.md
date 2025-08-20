# Documentação API - Sistema de Templates de Produtos

## Visão Geral
Sistema para gerenciar templates pré-configurados para diferentes tipos de produtos (cursos, e-books, eventos, área de membros, software), com configurações padrão e personalizações.

## Endpoints Disponíveis

### 1. Listar Templates
**GET** `/api/templates`

**Autenticação:** Requerida

**Descrição:** Lista todos os templates disponíveis com paginação e filtros.

**Parâmetros de Query:**
- `type` (string, opcional): Filtrar por tipo (`course`, `ebook`, `event`, `membership`, `software`)
- `status` (string, opcional): Filtrar por status (`active`, `inactive`)
- `page` (number, opcional): Página (padrão: 1)
- `limit` (number, opcional): Itens por página (padrão: 20)

**Exemplo de Requisição:**
```
GET /api/templates?type=course&status=active&page=1&limit=10
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-template",
      "name": "Curso Online Padrão",
      "description": "Template padrão para cursos online com estrutura completa",
      "type": "course",
      "template_data": {
        "sections": ["Apresentação", "Conteúdo Principal", "Bônus", "Garantia"],
        "features": ["Acesso vitalício", "Certificado de conclusão", "Suporte direto"],
        "pricing_style": "destacado",
        "layout": "clean"
      },
      "preview_image": "https://cdn.example.com/templates/course-default.jpg",
      "settings": {
        "show_instructor": true,
        "show_testimonials": true,
        "show_faq": true,
        "show_guarantee": true
      },
      "is_default": true,
      "status": "active",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "currentPage": 1,
    "totalPages": 3,
    "totalItems": 25,
    "itemsPerPage": 10
  }
}
```

### 2. Tipos de Templates Disponíveis
**GET** `/api/templates/types`

**Autenticação:** Requerida

**Descrição:** Lista os tipos de templates disponíveis com suas descrições.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "value": "course",
      "label": "Curso Online",
      "description": "Template otimizado para cursos e treinamentos"
    },
    {
      "value": "ebook",
      "label": "E-book",
      "description": "Template para livros digitais e materiais escritos"
    },
    {
      "value": "event",
      "label": "Evento",
      "description": "Template para workshops, webinars e eventos"
    },
    {
      "value": "membership",
      "label": "Área de Membros",
      "description": "Template para assinaturas e conteúdo exclusivo"
    },
    {
      "value": "software",
      "label": "Software/Ferramenta",
      "description": "Template para aplicativos e ferramentas digitais"
    }
  ]
}
```

### 3. Buscar Template Específico
**GET** `/api/templates/:id`

**Autenticação:** Requerida

**Descrição:** Retorna dados completos de um template específico.

**Parâmetros de Rota:**
- `id` (string): ID do template

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid-template",
    "name": "Curso Online Padrão",
    "description": "Template padrão para cursos online com estrutura completa",
    "type": "course",
    "template_data": {
      "sections": ["Apresentação", "Conteúdo Principal", "Bônus", "Garantia"],
      "features": ["Acesso vitalício", "Certificado de conclusão", "Suporte direto"],
      "pricing_style": "destacado",
      "layout": "clean",
      "colors": {
        "primary": "#3B82F6",
        "secondary": "#10B981",
        "accent": "#F59E0B"
      },
      "fonts": {
        "heading": "Inter",
        "body": "Open Sans"
      }
    },
    "preview_image": "https://cdn.example.com/templates/course-default.jpg",
    "settings": {
      "show_instructor": true,
      "show_testimonials": true,
      "show_faq": true,
      "show_guarantee": true,
      "enable_countdown": false,
      "enable_social_proof": true
    },
    "is_default": true,
    "status": "active",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
  }
}
```

### 4. Buscar Templates por Tipo
**GET** `/api/templates/type/:type`

**Autenticação:** Requerida

**Descrição:** Lista templates filtrados por tipo específico.

**Parâmetros de Rota:**
- `type` (string): Tipo do template

**Parâmetros de Query:**
- `status` (string, opcional): Status dos templates (padrão: `active`)

**Exemplo de Requisição:**
```
GET /api/templates/type/course?status=active
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-template-1",
      "name": "Curso Online Padrão",
      "description": "Template padrão para cursos online",
      "type": "course",
      "is_default": true,
      "status": "active"
    },
    {
      "id": "uuid-template-2",
      "name": "Curso Premium",
      "description": "Template premium com animações",
      "type": "course",
      "is_default": false,
      "status": "active"
    }
  ]
}
```

### 5. Criar Novo Template
**POST** `/api/templates`

**Autenticação:** Requerida

**Descrição:** Cria um novo template personalizado.

**Body da Requisição:**
```json
{
  "name": "Meu Template Personalizado",
  "description": "Template customizado para meus cursos",
  "type": "course",
  "template_data": {
    "sections": ["Introdução", "Módulos", "Bônus", "Certificado"],
    "features": ["Acesso vitalício", "Suporte 24/7", "Garantia 30 dias"],
    "pricing_style": "destacado",
    "layout": "modern",
    "colors": {
      "primary": "#6366F1",
      "secondary": "#EC4899",
      "accent": "#F59E0B"
    }
  },
  "preview_image": "https://cdn.example.com/my-template-preview.jpg",
  "settings": {
    "show_instructor": true,
    "show_testimonials": true,
    "show_faq": true,
    "show_guarantee": true,
    "enable_countdown": true
  },
  "is_default": false
}
```

**Parâmetros:**
- `name` (string, obrigatório): Nome do template
- `description` (string, opcional): Descrição do template
- `type` (string, obrigatório): Tipo do template
- `template_data` (object, opcional): Dados de configuração do template
- `preview_image` (string, opcional): URL da imagem de preview
- `settings` (object, opcional): Configurações do template
- `is_default` (boolean, opcional): Se é template padrão (padrão: false)

**Resposta de Sucesso (201):**
```json
{
  "success": true,
  "message": "Template criado com sucesso",
  "data": {
    "id": "uuid-novo-template",
    "name": "Meu Template Personalizado",
    "type": "course",
    "is_default": false,
    "status": "active",
    "created_at": "2024-01-15T14:30:00Z"
  }
}
```

**Erros Possíveis:**
- `400` - Nome e tipo obrigatórios
- `400` - Tipo de template inválido

### 6. Atualizar Template
**PUT** `/api/templates/:id`

**Autenticação:** Requerida

**Descrição:** Atualiza um template existente.

**Parâmetros de Rota:**
- `id` (string): ID do template

**Body da Requisição:**
```json
{
  "name": "Template Atualizado",
  "description": "Nova descrição",
  "template_data": {
    "sections": ["Nova Seção", "Conteúdo", "Final"],
    "layout": "updated"
  },
  "settings": {
    "show_countdown": true,
    "enable_animations": true
  }
}
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Template atualizado com sucesso",
  "data": {
    "id": "uuid-template",
    "name": "Template Atualizado",
    "updated_at": "2024-01-15T15:00:00Z"
  }
}
```

**Erros Possíveis:**
- `404` - Template não encontrado

### 7. Duplicar Template
**POST** `/api/templates/:id/duplicate`

**Autenticação:** Requerida

**Descrição:** Cria uma cópia de um template existente.

**Parâmetros de Rota:**
- `id` (string): ID do template original

**Body da Requisição:**
```json
{
  "name": "Cópia do Template Original"
}
```

**Parâmetros:**
- `name` (string, opcional): Nome para a cópia (padrão: "[Nome Original] - Cópia")

**Resposta de Sucesso (201):**
```json
{
  "success": true,
  "message": "Template duplicado com sucesso",
  "data": {
    "id": "uuid-nova-copia",
    "name": "Cópia do Template Original",
    "type": "course",
    "is_default": false,
    "status": "active",
    "created_at": "2024-01-15T15:30:00Z"
  }
}
```

### 8. Alterar Status do Template
**PATCH** `/api/templates/:id/status`

**Autenticação:** Requerida

**Descrição:** Ativa ou desativa um template.

**Parâmetros de Rota:**
- `id` (string): ID do template

**Body da Requisição:**
```json
{
  "status": "inactive"
}
```

**Parâmetros:**
- `status` (string, obrigatório): Novo status (`active` ou `inactive`)

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Template desativado com sucesso",
  "data": {
    "id": "uuid-template",
    "status": "inactive",
    "updated_at": "2024-01-15T16:00:00Z"
  }
}
```

**Erros Possíveis:**
- `400` - Status inválido
- `400` - Não é possível desativar template padrão
- `404` - Template não encontrado

### 9. Deletar Template
**DELETE** `/api/templates/:id`

**Autenticação:** Requerida

**Descrição:** Remove um template do sistema.

**Parâmetros de Rota:**
- `id` (string): ID do template

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Template deletado com sucesso"
}
```

**Erros Possíveis:**
- `400` - Não é possível deletar template padrão
- `404` - Template não encontrado

## Templates Padrão do Sistema

### 1. Curso Online Padrão
```json
{
  "type": "course",
  "name": "Curso Online Padrão",
  "template_data": {
    "sections": ["Apresentação", "Conteúdo Principal", "Bônus", "Garantia"],
    "features": ["Acesso vitalício", "Certificado de conclusão", "Suporte direto"],
    "pricing_style": "destacado",
    "layout": "clean"
  },
  "settings": {
    "show_instructor": true,
    "show_testimonials": true,
    "show_faq": true,
    "show_guarantee": true
  }
}
```

### 2. E-book Clássico
```json
{
  "type": "ebook",
  "name": "E-book Clássico",
  "template_data": {
    "sections": ["Sobre o Livro", "Autor", "Conteúdo", "Depoimentos"],
    "features": ["Download imediato", "Formato PDF", "Versão mobile"],
    "pricing_style": "simples",
    "layout": "editorial"
  },
  "settings": {
    "show_author": true,
    "show_sample": true,
    "show_testimonials": true,
    "show_guarantee": false
  }
}
```

### 3. Evento/Workshop
```json
{
  "type": "event",
  "name": "Evento/Workshop",
  "template_data": {
    "sections": ["Sobre o Evento", "Palestrante", "Programação", "Inscrições"],
    "features": ["Data e horário", "Link de acesso", "Material complementar"],
    "pricing_style": "urgencia",
    "layout": "evento"
  },
  "settings": {
    "show_schedule": true,
    "show_speaker": true,
    "show_countdown": true,
    "show_live_info": true
  }
}
```

### 4. Área de Membros
```json
{
  "type": "membership",
  "name": "Área de Membros",
  "template_data": {
    "sections": ["Benefícios", "Conteúdo Exclusivo", "Comunidade", "Planos"],
    "features": ["Acesso mensal", "Conteúdo atualizado", "Comunidade privada"],
    "pricing_style": "planos",
    "layout": "membership"
  },
  "settings": {
    "show_plans": true,
    "show_community": true,
    "show_updates": true,
    "show_trial": true
  }
}
```

### 5. Software/Ferramenta
```json
{
  "type": "software",
  "name": "Software/Ferramenta",
  "template_data": {
    "sections": ["Recursos", "Demonstração", "Planos", "Suporte"],
    "features": ["Licença vitalícia", "Atualizações grátis", "Suporte técnico"],
    "pricing_style": "comparativo",
    "layout": "tech"
  },
  "settings": {
    "show_demo": true,
    "show_features": true,
    "show_plans": true,
    "show_support": true
  }
}
```

## Estrutura de Dados do Template

### template_data
```json
{
  "sections": ["array", "de", "seções"],
  "features": ["lista", "de", "características"],
  "pricing_style": "destacado|simples|urgencia|planos|comparativo",
  "layout": "clean|modern|editorial|evento|membership|tech",
  "colors": {
    "primary": "#3B82F6",
    "secondary": "#10B981",
    "accent": "#F59E0B",
    "background": "#FFFFFF",
    "text": "#1F2937"
  },
  "fonts": {
    "heading": "Inter|Poppins|Roboto",
    "body": "Open Sans|Lato|Source Sans Pro"
  },
  "animations": {
    "enabled": true,
    "type": "fade|slide|bounce",
    "duration": 300
  }
}
```

### settings
```json
{
  "show_instructor": true,
  "show_testimonials": true,
  "show_faq": true,
  "show_guarantee": true,
  "show_countdown": false,
  "enable_social_proof": true,
  "enable_animations": true,
  "enable_mobile_optimization": true,
  "seo_optimized": true
}
```

## Regras de Negócio

### Templates Padrão
- Cada tipo pode ter apenas um template padrão
- Ao definir um novo padrão, o anterior é automaticamente desmarcado
- Templates padrão não podem ser deletados ou desativados
- Para remover um padrão, primeiro defina outro como padrão

### Validações
- Nome e tipo são obrigatórios na criação
- Tipos válidos: `course`, `ebook`, `event`, `membership`, `software`
- Status válidos: `active`, `inactive`

### Duplicação
- Cópias nunca são marcadas como padrão
- Nome é automaticamente gerado se não fornecido
- Todas as configurações são copiadas

## Exemplos de Uso

### Listar Templates de Curso
```javascript
const templates = await fetch('/api/templates/type/course', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
const courseTemplates = await templates.json();
```

### Criar Template Personalizado
```javascript
const newTemplate = {
  name: 'Meu Template Curso Premium',
  description: 'Template customizado para cursos premium',
  type: 'course',
  template_data: {
    sections: ['Hero', 'Módulos', 'Depoimentos', 'Preço', 'FAQ'],
    features: ['Acesso vitalício', 'Certificado', 'Suporte VIP'],
    layout: 'modern',
    colors: {
      primary: '#6366F1',
      secondary: '#EC4899'
    }
  },
  settings: {
    show_countdown: true,
    enable_animations: true
  }
};

const response = await fetch('/api/templates', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(newTemplate)
});
```

### Aplicar Template a Produto
```javascript
// Usar template ao criar produto
const productData = {
  name: 'Novo Curso',
  template_id: 'uuid-template-escolhido',
  // ... outros dados do produto
};

// O sistema aplicará automaticamente as configurações do template
```

### Duplicar e Personalizar
```javascript
// Duplicar template existente
const duplicated = await fetch('/api/templates/uuid-template/duplicate', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Minha Versão Personalizada'
  })
});

// Personalizar a cópia
const customized = await fetch(`/api/templates/${duplicated.data.id}`, {
  method: 'PUT',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    template_data: {
      colors: { primary: '#FF5722' },
      layout: 'custom'
    }
  })
});
```

## Observações Técnicas

1. **Inicialização**: Templates padrão são criados automaticamente na inicialização do sistema
2. **Performance**: Consultas otimizadas com ordenação por padrão e data
3. **Flexibilidade**: Estrutura de dados extensível para novos tipos e configurações
4. **Backup**: Templates são preservados mesmo quando produtos são deletados
5. **Versionamento**: Possibilidade futura de versionar templates
6. **Cache**: Sistema preparado para cache de templates frequentemente usados
