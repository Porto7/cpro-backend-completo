# 🎓 Documentação Completa - Rotas de Cursos

## Visão Geral
O arquivo `routes/courses.js` contém o sistema completo de gerenciamento de cursos online, incluindo CRUD de cursos, módulos, aulas, sistema de matrículas, progresso dos alunos, upload de mídias e controle de acesso. É o coração da plataforma educacional.

## Estrutura de Dados
- **Cursos**: Conteúdo principal com instrutor, módulos e configurações
- **Módulos**: Agrupamento lógico de aulas dentro de um curso
- **Aulas**: Conteúdo individual (vídeo, texto, quiz) com progresso tracking
- **Matrículas**: Ligação entre aluno e curso com dados de acesso
- **Progresso**: Tracking detalhado por aula e curso completo

## Funcionalidades Principais
- **CRUD Completo**: Cursos, módulos e aulas
- **Sistema de Matrículas**: Automático via webhook de pagamento
- **Progresso Avançado**: Tracking por aula e curso
- **Controle de Acesso**: Verificações de permissão
- **Upload de Mídia**: Vídeos, materiais e thumbnails
- **Streaming Seguro**: URLs temporárias com token

---

## 📚 GESTÃO DE CURSOS

### GET /api/courses/my-courses
**Descrição**: Busca cursos do aluno logado (cursos matriculados)

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "courses": [
    {
      "enrollment": {
        "id": "uuid-matricula-123",
        "enrolledAt": "2024-12-31T18:30:00.000Z",
        "status": "active",
        "progress": 65,
        "expiresAt": "2025-12-31T18:30:00.000Z"
      },
      "course": {
        "id": "uuid-curso-456",
        "title": "Curso de JavaScript Avançado",
        "description": "Aprenda JavaScript do zero ao avançado",
        "thumbnailUrl": "https://exemplo.com/thumb.jpg",
        "trailerUrl": "https://exemplo.com/trailer.mp4",
        "duration": 1800,
        "level": "intermediate",
        "category": "programacao",
        "status": "published",
        "product": {
          "id": "uuid-produto-789",
          "name": "Curso de JavaScript Avançado",
          "price": 197.50,
          "imageUrl": "https://exemplo.com/image.jpg"
        },
        "instructor": {
          "id": "uuid-instrutor-321",
          "name": "João Silva",
          "email": "joao@email.com"
        },
        "modules": [
          {
            "id": "uuid-modulo-111",
            "title": "Fundamentos",
            "orderIndex": 0,
            "duration": 450,
            "isFree": false,
            "lessonsCount": 8,
            "lessons": [
              {
                "id": "uuid-aula-222",
                "title": "Introdução ao JavaScript",
                "duration": 30,
                "isFree": true,
                "orderIndex": 0
              }
            ]
          }
        ]
      },
      "purchase": {
        "id": "uuid-pedido-555",
        "amount": 197.50,
        "status": "completed",
        "paymentMethod": "credit_card"
      }
    }
  ],
  "total": 1,
  "user": {
    "id": "uuid-usuario-666",
    "name": "Maria Santos",
    "email": "maria@email.com"
  }
}
```

**Campos da Matrícula**:
- `enrolledAt`: Data de matrícula
- `status`: Status da matrícula (active, suspended, expired)
- `progress`: Progresso em % (0-100)
- `expiresAt`: Data de expiração do acesso

**Uso**: Área do aluno para ver todos os cursos matriculados

**Erros Possíveis**:
- `401`: Token inválido ou expirado
- `500`: Erro interno do servidor

---

### GET /api/courses
**Descrição**: Busca cursos do instrutor logado ou cursos matriculados (se aluno)

**Autenticação**: Obrigatória

**Query Parameters**:
- `category` (opcional): Filtrar por categoria
- `status` (opcional): Filtrar por status (draft, published, archived)
- `student_id` (opcional): Filtrar por aluno específico

**Comportamento por Role**:
- **Instrutor/Seller**: Retorna apenas cursos criados por ele
- **Aluno**: Retorna cursos onde está matriculado

**Resposta de Sucesso (200)**:
```json
[
  {
    "id": "uuid-curso-123",
    "title": "Curso de React Avançado",
    "description": "Domine React e suas principais bibliotecas",
    "thumbnailUrl": "https://exemplo.com/react-thumb.jpg",
    "trailerUrl": "https://exemplo.com/react-trailer.mp4",
    "durationMinutes": 2400,
    "level": "advanced",
    "category": "frontend",
    "tags": ["react", "javascript", "frontend"],
    "status": "published",
    "createdAt": "2024-12-01T10:00:00.000Z",
    "product": {
      "id": "uuid-produto-456",
      "name": "Curso de React Avançado",
      "price": 297.50,
      "imageUrl": "https://exemplo.com/react-image.jpg"
    },
    "instructor": {
      "id": "uuid-instrutor-789",
      "name": "Pedro Oliveira",
      "email": "pedro@email.com"
    },
    "enrollment": null
  }
]
```

**Filtros Disponíveis**:
- **Por Categoria**: `?category=programacao`
- **Por Status**: `?status=published`
- **Por Aluno**: `?student_id=uuid-aluno-123`

**Uso**: Dashboard do instrutor ou biblioteca de cursos do aluno

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### GET /api/courses/:course_id
**Descrição**: Busca curso específico com módulos e aulas completos

**Parâmetros**:
- `course_id` (string): UUID do curso

**Query Parameters**:
- `student_id` (opcional): ID do aluno para incluir progresso

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "course": {
    "id": "uuid-curso-123",
    "title": "Curso de Node.js Completo",
    "description": "Aprenda backend com Node.js, Express e MongoDB",
    "thumbnailUrl": "https://exemplo.com/nodejs-thumb.jpg",
    "trailerUrl": "https://exemplo.com/nodejs-trailer.mp4",
    "durationMinutes": 3600,
    "level": "intermediate",
    "category": "backend",
    "tags": ["nodejs", "express", "mongodb"],
    "settings": {
      "allowDownloads": false,
      "hasCertificate": true,
      "accessDuration": 365
    },
    "status": "published",
    "createdAt": "2024-11-15T14:00:00.000Z",
    "product": {
      "id": "uuid-produto-789",
      "name": "Curso de Node.js Completo",
      "price": 247.50,
      "imageUrl": "https://exemplo.com/nodejs-image.jpg"
    },
    "instructor": {
      "id": "uuid-instrutor-456",
      "name": "Ana Silva",
      "email": "ana@email.com"
    },
    "modules": [
      {
        "id": "uuid-modulo-111",
        "title": "Introdução ao Node.js",
        "description": "Fundamentos e configuração",
        "orderIndex": 0,
        "durationMinutes": 240,
        "isFree": false,
        "lessons": [
          {
            "id": "uuid-aula-222",
            "title": "O que é Node.js",
            "description": "Conceitos fundamentais",
            "contentType": "video",
            "contentUrl": "https://exemplo.com/aula1.mp4",
            "contentData": {
              "quality": "1080p",
              "hasSubtitles": true
            },
            "durationMinutes": 45,
            "orderIndex": 0,
            "isFree": true,
            "transcript": "Transcrição da aula..."
          }
        ]
      }
    ],
    "enrollment": {
      "id": "uuid-matricula-333",
      "enrolledAt": "2024-12-20T09:00:00.000Z",
      "status": "active",
      "progressPercentage": 25,
      "progress": [
        {
          "lessonId": "uuid-aula-222",
          "status": "completed",
          "progressPercentage": 100,
          "timeSpentMinutes": 45,
          "lastPosition": 2700,
          "completedAt": "2024-12-21T10:30:00.000Z"
        }
      ]
    }
  }
}
```

**Tipos de Conteúdo**:
- `video`: Vídeo-aula
- `text`: Conteúdo textual
- `quiz`: Questionário
- `file`: Arquivo para download

**Configurações do Curso**:
- `allowDownloads`: Permite download das aulas
- `hasCertificate`: Curso oferece certificado
- `accessDuration`: Duração do acesso em dias

**Uso**: Página detalhada do curso ou player de aulas

**Erros Possíveis**:
- `404`: Curso não encontrado
- `500`: Erro interno do servidor

---

### POST /api/courses
**Descrição**: Cria novo curso (usuário logado como instrutor)

**Autenticação**: Obrigatória

**Body Parameters**:
```json
{
  "title": "Curso de Python Avançado",
  "description": "Domine Python com projetos práticos",
  "thumbnailUrl": "https://exemplo.com/python-thumb.jpg",
  "trailerUrl": "https://exemplo.com/python-trailer.mp4",
  "durationMinutes": 2000,
  "level": "advanced",
  "category": "programacao",
  "tags": ["python", "django", "data-science"],
  "settings": {
    "allowDownloads": true,
    "hasCertificate": true,
    "accessDuration": 365
  },
  "status": "draft",
  "productId": "uuid-produto-existente"
}
```

**Parâmetros Obrigatórios**:
- `title` (string): Nome do curso

**Parâmetros Opcionais**:
- `description` (string): Descrição detalhada
- `thumbnailUrl` (string): URL da imagem de capa
- `trailerUrl` (string): URL do vídeo de apresentação
- `durationMinutes` (number): Duração total estimada
- `level` (string): Nível (beginner, intermediate, advanced)
- `category` (string): Categoria do curso
- `tags` (array): Tags para busca
- `settings` (object): Configurações específicas
- `status` (string): Status inicial (draft, published)
- `productId` (string): ID de produto existente

**Criação Automática de Produto**:
Se `productId` não fornecido, sistema cria produto automaticamente:
```javascript
{
  "name": "Curso de Python Avançado",
  "description": "Curso online: Curso de Python Avançado",
  "price": 0,
  "slug": "curso-de-python-avancado",
  "category": "curso",
  "seller_id": "uuid-instrutor-logado"
}
```

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Curso criado com sucesso",
  "course": {
    "id": "uuid-curso-novo",
    "title": "Curso de Python Avançado",
    "description": "Domine Python com projetos práticos",
    "thumbnailUrl": "https://exemplo.com/python-thumb.jpg",
    "trailerUrl": "https://exemplo.com/python-trailer.mp4",
    "durationMinutes": 2000,
    "level": "advanced",
    "category": "programacao",
    "tags": ["python", "django", "data-science"],
    "settings": {
      "allowDownloads": true,
      "hasCertificate": true,
      "accessDuration": 365
    },
    "status": "draft",
    "createdAt": "2024-12-31T18:30:00.000Z"
  }
}
```

**Segurança**: `instructor_id` sempre definido como usuário logado

**Uso**: Criação de novos cursos na plataforma

**Erros Possíveis**:
- `400`: Título obrigatório ausente
- `404`: Produto não encontrado (se productId fornecido)
- `400`: Já existe curso para o produto
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### PUT /api/courses/:course_id
**Descrição**: Atualiza curso (apenas se for do instrutor logado)

**Parâmetros**:
- `course_id` (string): UUID do curso

**Autenticação**: Obrigatória

**Body Parameters** (todos opcionais):
```json
{
  "title": "Novo Título do Curso",
  "description": "Nova descrição",
  "thumbnailUrl": "https://exemplo.com/nova-thumb.jpg",
  "trailerUrl": "https://exemplo.com/novo-trailer.mp4",
  "durationMinutes": 2500,
  "level": "intermediate",
  "category": "nova-categoria",
  "tags": ["tag1", "tag2"],
  "settings": {
    "allowDownloads": false,
    "hasCertificate": false
  },
  "status": "published"
}
```

**Segurança**: Verifica se curso pertence ao instrutor logado

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Curso atualizado com sucesso",
  "course": {
    "id": "uuid-curso-123",
    "title": "Novo Título do Curso",
    "description": "Nova descrição",
    "thumbnailUrl": "https://exemplo.com/nova-thumb.jpg",
    "trailerUrl": "https://exemplo.com/novo-trailer.mp4",
    "durationMinutes": 2500,
    "level": "intermediate",
    "category": "nova-categoria",
    "tags": ["tag1", "tag2"],
    "settings": {
      "allowDownloads": false,
      "hasCertificate": false
    },
    "status": "published",
    "updatedAt": "2024-12-31T18:35:00.000Z"
  }
}
```

**Uso**: Edição de cursos existentes

**Erros Possíveis**:
- `404`: Curso não encontrado ou sem permissão
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### DELETE /api/courses/:course_id
**Descrição**: Deleta curso completo (apenas se for do instrutor logado)

**Parâmetros**:
- `course_id` (string): UUID do curso

**Autenticação**: Obrigatória

**Segurança**: Verifica se curso pertence ao instrutor logado

**Efeito Cascade**: Deleta automaticamente:
- Todos os módulos do curso
- Todas as aulas dos módulos
- Todas as matrículas
- Todo o progresso dos alunos

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Curso deletado com sucesso"
}
```

**Uso**: Remoção definitiva de cursos

**Erros Possíveis**:
- `404`: Curso não encontrado ou sem permissão
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 📂 GESTÃO DE MÓDULOS

### POST /api/courses/:course_id/modules
**Descrição**: Cria módulo no curso

**Parâmetros**:
- `course_id` (string): UUID do curso

**Autenticação**: Obrigatória

**Body Parameters**:
```json
{
  "title": "Módulo de APIs REST",
  "description": "Aprenda a criar APIs com Express",
  "orderIndex": 2,
  "durationMinutes": 480,
  "isFree": false
}
```

**Parâmetros Obrigatórios**:
- `title` (string): Nome do módulo

**Parâmetros Opcionais**:
- `description` (string): Descrição do módulo
- `orderIndex` (number): Ordem de exibição (auto se omitido)
- `durationMinutes` (number): Duração estimada
- `isFree` (boolean): Se é módulo gratuito

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Módulo criado com sucesso",
  "module": {
    "id": "uuid-modulo-novo",
    "title": "Módulo de APIs REST",
    "description": "Aprenda a criar APIs com Express",
    "orderIndex": 2,
    "durationMinutes": 480,
    "isFree": false,
    "createdAt": "2024-12-31T18:40:00.000Z"
  }
}
```

**Auto-Ordering**: Se `orderIndex` omitido, usa próximo índice disponível

**Uso**: Organização de conteúdo em módulos

**Erros Possíveis**:
- `404`: Curso não encontrado
- `400`: Título obrigatório
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### PUT /api/courses/:course_id/modules/:module_id
**Descrição**: Atualiza módulo existente

**Parâmetros**:
- `course_id` (string): UUID do curso
- `module_id` (string): UUID do módulo

**Autenticação**: Obrigatória

**Body Parameters** (todos opcionais):
```json
{
  "title": "Novo Título do Módulo",
  "description": "Nova descrição",
  "orderIndex": 1,
  "durationMinutes": 600,
  "isFree": true
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Módulo atualizado com sucesso",
  "module": {
    "id": "uuid-modulo-123",
    "title": "Novo Título do Módulo",
    "description": "Nova descrição",
    "orderIndex": 1,
    "durationMinutes": 600,
    "isFree": true,
    "updatedAt": "2024-12-31T18:45:00.000Z"
  }
}
```

**Uso**: Edição de módulos existentes

**Erros Possíveis**:
- `404`: Módulo não encontrado
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### DELETE /api/courses/:course_id/modules/:module_id
**Descrição**: Deleta módulo e todas suas aulas

**Parâmetros**:
- `course_id` (string): UUID do curso
- `module_id` (string): UUID do módulo

**Autenticação**: Obrigatória

**Efeito Cascade**: Deleta todas as aulas do módulo

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Módulo excluído com sucesso"
}
```

**Uso**: Remoção de módulos completos

**Erros Possíveis**:
- `404`: Módulo não encontrado
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 🎥 GESTÃO DE AULAS

### POST /api/courses/:course_id/modules/:module_id/lessons
**Descrição**: Cria aula no módulo

**Parâmetros**:
- `course_id` (string): UUID do curso
- `module_id` (string): UUID do módulo

**Autenticação**: Obrigatória

**Body Parameters**:
```json
{
  "title": "Criando sua primeira API",
  "description": "Passo a passo para criar uma API REST",
  "contentType": "video",
  "contentUrl": "https://exemplo.com/aula-api.mp4",
  "contentData": {
    "quality": "1080p",
    "hasSubtitles": true,
    "chapters": [
      { "time": 0, "title": "Introdução" },
      { "time": 300, "title": "Configuração" }
    ]
  },
  "durationMinutes": 60,
  "orderIndex": 1,
  "isFree": false,
  "transcript": "Nesta aula veremos como..."
}
```

**Parâmetros Obrigatórios**:
- `title` (string): Nome da aula

**Parâmetros Opcionais**:
- `description` (string): Descrição da aula
- `contentType` (string): Tipo (video, text, quiz, file)
- `contentUrl` (string): URL do conteúdo
- `contentData` (object): Metadados do conteúdo
- `durationMinutes` (number): Duração da aula
- `orderIndex` (number): Ordem de exibição
- `isFree` (boolean): Se é aula gratuita
- `transcript` (string): Transcrição da aula

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Aula criada com sucesso",
  "lesson": {
    "id": "uuid-aula-nova",
    "title": "Criando sua primeira API",
    "description": "Passo a passo para criar uma API REST",
    "contentType": "video",
    "contentUrl": "https://exemplo.com/aula-api.mp4",
    "contentData": {
      "quality": "1080p",
      "hasSubtitles": true,
      "chapters": [
        { "time": 0, "title": "Introdução" },
        { "time": 300, "title": "Configuração" }
      ]
    },
    "durationMinutes": 60,
    "orderIndex": 1,
    "isFree": false,
    "transcript": "Nesta aula veremos como...",
    "createdAt": "2024-12-31T19:00:00.000Z"
  }
}
```

**Tipos de Conteúdo Suportados**:
- `video`: Vídeo-aula
- `text`: Artigo ou texto
- `quiz`: Questionário interativo
- `file`: Arquivo para download

**Uso**: Criação de conteúdo educacional

**Erros Possíveis**:
- `404`: Módulo não encontrado
- `400`: Título obrigatório
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### PUT /api/courses/:course_id/modules/:module_id/lessons/:lesson_id
**Descrição**: Atualiza aula existente

**Parâmetros**:
- `course_id` (string): UUID do curso
- `module_id` (string): UUID do módulo
- `lesson_id` (string): UUID da aula

**Autenticação**: Obrigatória

**Body Parameters** (todos opcionais):
```json
{
  "title": "Novo Título da Aula",
  "description": "Nova descrição",
  "contentType": "video",
  "contentUrl": "https://exemplo.com/nova-aula.mp4",
  "contentData": {
    "quality": "4K",
    "hasSubtitles": true
  },
  "durationMinutes": 75,
  "orderIndex": 2,
  "isFree": true,
  "transcript": "Nova transcrição..."
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Aula atualizada com sucesso",
  "lesson": {
    "id": "uuid-aula-123",
    "title": "Novo Título da Aula",
    "description": "Nova descrição",
    "contentType": "video",
    "contentUrl": "https://exemplo.com/nova-aula.mp4",
    "contentData": {
      "quality": "4K",
      "hasSubtitles": true
    },
    "durationMinutes": 75,
    "orderIndex": 2,
    "isFree": true,
    "transcript": "Nova transcrição...",
    "updatedAt": "2024-12-31T19:10:00.000Z"
  }
}
```

**Uso**: Edição de aulas existentes

**Erros Possíveis**:
- `404`: Aula não encontrada
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### DELETE /api/courses/:course_id/modules/:module_id/lessons/:lesson_id
**Descrição**: Deleta aula específica

**Parâmetros**:
- `course_id` (string): UUID do curso
- `module_id` (string): UUID do módulo
- `lesson_id` (string): UUID da aula

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Aula excluída com sucesso"
}
```

**Uso**: Remoção de aulas específicas

**Erros Possíveis**:
- `404`: Aula não encontrada
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 📝 SISTEMA DE MATRÍCULAS

### POST /api/courses/:course_id/enroll
**Descrição**: Matricula aluno no curso (usado após compra)

**Parâmetros**:
- `course_id` (string): UUID do curso

**Autenticação**: Obrigatória

**Body Parameters**:
```json
{
  "studentId": "uuid-aluno-123",
  "orderId": "uuid-pedido-456"
}
```

**Parâmetros Obrigatórios**:
- `studentId` (string): ID do aluno
- `orderId` (string): ID do pedido de compra

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Matrícula realizada com sucesso",
  "enrollment": {
    "id": "uuid-matricula-789",
    "courseId": "uuid-curso-123",
    "studentId": "uuid-aluno-123",
    "orderId": "uuid-pedido-456",
    "enrolledAt": "2024-12-31T19:15:00.000Z",
    "status": "active",
    "progressPercentage": 0
  }
}
```

**Validações**:
- Verifica se curso existe
- Verifica se aluno existe
- Verifica se pedido existe
- Evita matrículas duplicadas

**Status da Matrícula**:
- `active`: Acesso liberado
- `suspended`: Acesso suspenso
- `expired`: Acesso expirado

**Uso**: Automação via webhook de pagamento

**Erros Possíveis**:
- `404`: Curso, aluno ou pedido não encontrado
- `400`: Aluno já matriculado
- `400`: Campos obrigatórios ausentes
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### GET /api/courses/:course_id/enrollments
**Descrição**: Lista matrículas do curso (para instrutor)

**Parâmetros**:
- `course_id` (string): UUID do curso

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "enrollments": [
    {
      "id": "uuid-matricula-123",
      "enrolledAt": "2024-12-20T10:00:00.000Z",
      "status": "active",
      "progressPercentage": 75,
      "completedAt": null,
      "student": {
        "id": "uuid-aluno-456",
        "name": "Ana Silva",
        "email": "ana@email.com"
      },
      "order": {
        "id": "uuid-pedido-789",
        "amount": 197.50,
        "createdAt": "2024-12-20T09:45:00.000Z"
      }
    }
  ]
}
```

**Ordenação**: Por data de matrícula (mais recentes primeiro)

**Uso**: Dashboard do instrutor para acompanhar alunos

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### GET /api/courses/enrollment-status/:courseId/:studentEmail
**Descrição**: Verifica status de matrícula de um aluno específico

**Parâmetros**:
- `courseId` (string): UUID do curso
- `studentEmail` (string): Email do aluno

**Autenticação**: Obrigatória

**Resposta - Não Matriculado (200)**:
```json
{
  "enrolled": false,
  "hasAccount": true,
  "student": {
    "id": "uuid-aluno-123",
    "name": "João Silva",
    "email": "joao@email.com"
  },
  "message": "Não matriculado neste curso"
}
```

**Resposta - Matriculado (200)**:
```json
{
  "enrolled": true,
  "hasAccount": true,
  "enrollment": {
    "id": "uuid-matricula-456",
    "status": "active",
    "enrolledAt": "2024-12-15T14:30:00.000Z",
    "progressPercentage": 45,
    "accessData": {
      "enrolledVia": "purchase",
      "email": "joao@email.com"
    }
  },
  "student": {
    "id": "uuid-aluno-123",
    "name": "João Silva",
    "email": "joao@email.com"
  },
  "course": {
    "id": "uuid-curso-789",
    "title": "Curso de JavaScript",
    "status": "published"
  },
  "order": {
    "id": "uuid-pedido-321",
    "status": "completed",
    "createdAt": "2024-12-15T14:25:00.000Z"
  }
}
```

**Uso**: Verificação de acesso antes de liberar conteúdo

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 📊 SISTEMA DE PROGRESSO

### PUT /api/courses/:course_id/progress
**Descrição**: Atualiza progresso da aula (versão legacy)

**Parâmetros**:
- `course_id` (string): UUID do curso

**Autenticação**: Obrigatória

**Body Parameters**:
```json
{
  "studentId": "uuid-aluno-123",
  "lessonId": "uuid-aula-456",
  "progressPercentage": 85,
  "timeSpentMinutes": 25,
  "lastPosition": 1530,
  "status": "in_progress"
}
```

**Parâmetros Obrigatórios**:
- `studentId` (string): ID do aluno
- `lessonId` (string): ID da aula

**Parâmetros Opcionais**:
- `progressPercentage` (number): % assistido (0-100)
- `timeSpentMinutes` (number): Tempo gasto em minutos
- `lastPosition` (number): Última posição em segundos
- `status` (string): Status (in_progress, completed)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Progresso atualizado com sucesso",
  "progress": {
    "lessonId": "uuid-aula-456",
    "status": "in_progress",
    "progressPercentage": 85,
    "timeSpentMinutes": 25,
    "lastPosition": 1530,
    "completedAt": null
  },
  "courseProgress": {
    "percentage": 45,
    "completedLessons": 8,
    "totalLessons": 18
  }
}
```

**Cálculo Automático**:
- Progresso geral do curso
- Contagem de aulas completadas
- Atualização de tempo total gasto

**Uso**: Tracking durante assistir aulas

**Erros Possíveis**:
- `404`: Matrícula não encontrada
- `400`: Campos obrigatórios ausentes
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### GET /api/courses/:courseId/progress
**Descrição**: Progresso completo do curso para o aluno logado

**Parâmetros**:
- `courseId` (string): UUID do curso

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "course": {
    "id": "uuid-curso-123",
    "title": "Curso de React Avançado",
    "description": "Domine React e suas principais bibliotecas",
    "thumbnailUrl": "https://exemplo.com/react-thumb.jpg",
    "instructor": {
      "name": "Pedro Silva",
      "avatar": "https://exemplo.com/pedro-avatar.jpg"
    },
    "totalDuration": 7200,
    "certificateAvailable": true
  },
  "enrollment": {
    "enrolledAt": "2024-12-01T10:00:00.000Z",
    "expiresAt": "2025-12-01T10:00:00.000Z",
    "status": "active"
  },
  "progress": {
    "totalLessons": 24,
    "completedLessons": 15,
    "progressPercentage": 63,
    "timeSpent": 10800,
    "estimatedTimeToComplete": 3240,
    "lastAccessed": "2024-12-30T15:30:00.000Z",
    "completionDate": null,
    "certificateEarned": false
  },
  "modules": [
    {
      "id": "uuid-modulo-111",
      "title": "Fundamentos do React",
      "description": "Conceitos básicos e componentes",
      "order": 0,
      "totalLessons": 6,
      "completedLessons": 6,
      "isCompleted": true,
      "lessons": [
        {
          "id": "uuid-aula-222",
          "title": "Introdução ao React",
          "type": "video",
          "duration": 45,
          "order": 0,
          "isCompleted": true,
          "completedAt": "2024-12-02T11:30:00.000Z",
          "timeSpent": 45,
          "watchedPercentage": 100,
          "isPreview": true,
          "hasAccess": true
        }
      ]
    }
  ],
  "currentLesson": {
    "id": "uuid-aula-333",
    "moduleId": "uuid-modulo-222",
    "title": "Hooks Avançados",
    "watchedPercentage": 25
  },
  "nextLesson": {
    "id": "uuid-aula-444",
    "moduleId": "uuid-modulo-222",
    "title": "Context API"
  }
}
```

**Métricas Calculadas**:
- **Progresso Geral**: % de aulas completadas
- **Tempo Gasto**: Total de minutos assistidos
- **Tempo Estimado**: Para completar o curso
- **Certificado**: Se disponível e elegível

**Aula Atual**: Primeira aula não completada
**Próxima Aula**: Aula seguinte à atual

**Uso**: Dashboard do aluno e player de curso

**Erros Possíveis**:
- `404`: Matrícula não encontrada
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### POST /api/courses/:courseId/lessons/:lessonId/progress
**Descrição**: Atualiza progresso específico de uma aula (versão moderna)

**Parâmetros**:
- `courseId` (string): UUID do curso
- `lessonId` (string): UUID da aula

**Autenticação**: Obrigatória

**Body Parameters**:
```json
{
  "completed": true,
  "timeSpent": 5,
  "watchedPercentage": 100,
  "lastPosition": 2700
}
```

**Parâmetros**:
- `completed` (boolean): Se aula foi completada
- `timeSpent` (number): Tempo gasto nesta sessão (minutos)
- `watchedPercentage` (number): % assistido (0-100)
- `lastPosition` (number): Posição final em segundos

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "lesson": {
    "id": "uuid-aula-123",
    "completed": true,
    "completedAt": "2024-12-31T20:00:00.000Z",
    "timeSpent": 45,
    "watchedPercentage": 100
  },
  "course": {
    "progressPercentage": 68,
    "completedLessons": 16,
    "totalLessons": 24
  },
  "achievements": [
    {
      "type": "lesson_completed",
      "message": "Parabéns! Você completou mais uma aula!"
    }
  ],
  "nextLesson": {
    "id": "uuid-aula-124",
    "title": "Próxima Aula: Redux Toolkit",
    "hasAccess": true
  },
  "unlockedContent": []
}
```

**Sistema de Conquistas**:
- **Aula Completada**: Quando completa uma aula
- **Curso Completado**: Quando atinge 100%
- **Módulo Completado**: Quando termina módulo

**Recálculo Automático**: 
- Progresso geral do curso
- Tempo total gasto
- Data de conclusão (se 100%)

**Uso**: Player de aulas com tracking em tempo real

**Erros Possíveis**:
- `403`: Sem acesso ao curso
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 🔒 CONTROLE DE ACESSO

### GET /api/courses/:courseId/enrollment/status
**Descrição**: Verifica status de matrícula do usuário logado

**Parâmetros**:
- `courseId` (string): UUID do curso

**Autenticação**: Obrigatória

**Resposta - Sem Acesso (200)**:
```json
{
  "hasAccess": false,
  "enrolled": false,
  "message": "Você não está matriculado neste curso"
}
```

**Resposta - Com Acesso (200)**:
```json
{
  "hasAccess": true,
  "enrolled": true,
  "enrollment": {
    "id": "uuid-matricula-123",
    "status": "active",
    "enrolledAt": "2024-12-15T10:00:00.000Z",
    "expiresAt": "2025-12-15T10:00:00.000Z",
    "progressPercentage": 45
  },
  "course": {
    "id": "uuid-curso-456",
    "title": "Curso de JavaScript",
    "status": "published"
  }
}
```

**Verificações**:
- Se existe matrícula ativa
- Se acesso não expirou
- Se curso está publicado

**Uso**: Middleware de acesso a áreas restritas

**Erros Possíveis**:
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### GET /api/courses/:courseId/lessons/:lessonId/access
**Descrição**: Verifica acesso específico a uma aula com navegação

**Parâmetros**:
- `courseId` (string): UUID do curso
- `lessonId` (string): UUID da aula

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "hasAccess": true,
  "accessType": "full",
  "downloadAllowed": false,
  "streamUrl": "https://exemplo.com/aula.mp4?token=abc123&expires=1735689600",
  "lesson": {
    "id": "uuid-aula-123",
    "title": "Introdução ao React",
    "description": "Conceitos fundamentais do React",
    "type": "video",
    "duration": 2700,
    "isCompleted": false,
    "watchedPercentage": 45,
    "lastPosition": 1215,
    "materials": [
      {
        "id": "uuid-material-456",
        "name": "Slides da Aula",
        "url": "https://exemplo.com/slides.pdf",
        "type": "pdf",
        "size": 2048000
      }
    ]
  },
  "nextLesson": {
    "id": "uuid-aula-124",
    "title": "Componentes e Props",
    "hasAccess": true
  },
  "prevLesson": {
    "id": "uuid-aula-122",
    "title": "Preparação do Ambiente",
    "hasAccess": true
  }
}
```

**Tipos de Acesso**:
- `full`: Acesso completo
- `preview`: Apenas preview
- `denied`: Sem acesso

**URL de Streaming**:
- Token temporário para segurança
- Expiração em 1 hora
- Proteção contra download não autorizado

**Navegação**:
- `nextLesson`: Próxima aula na sequência
- `prevLesson`: Aula anterior na sequência

**Materiais**: Arquivos complementares da aula

**Uso**: Player de aulas com controle granular

**Erros Possíveis**:
- `403`: Sem acesso ao curso
- `404`: Aula não encontrada
- `401`: Token inválido
- `500`: Erro interno do servidor

---

### GET /api/courses/:courseId/lessons/:lessonId/media
**Descrição**: Gera URL segura de streaming/download

**Parâmetros**:
- `courseId` (string): UUID do curso
- `lessonId` (string): UUID da aula

**Query Parameters**:
- `type` (string): Tipo de acesso (stream, download)

**Autenticação**: Obrigatória

**Resposta de Sucesso (200)**:
```json
{
  "url": "https://exemplo.com/secure/aula.mp4?token=xyz789&expires=1735693200",
  "expiresAt": "2025-01-01T01:00:00.000Z",
  "type": "stream",
  "quality": "1080p",
  "watermark": true,
  "allowDownload": false
}
```

**Verificações de Segurança**:
- Verifica se usuário tem acesso ao curso
- Verifica se downloads estão habilitados (para type=download)
- Gera token único por sessão

**Configurações**:
- **Expiração**: 1 hora por padrão
- **Qualidade**: Baseada no plano do curso
- **Watermark**: Para proteção de conteúdo
- **Download**: Controlado por configuração do curso

**Uso**: Player de vídeo com URLs seguras

**Erros Possíveis**:
- `403`: Sem acesso ou downloads desabilitados
- `404`: Aula não encontrada
- `401`: Token inválido
- `500`: Erro interno do servidor

---

## 📁 UPLOAD DE MÍDIA

### POST /api/courses/upload
**Descrição**: Upload de arquivos para cursos (vídeos, thumbnails, materiais)

**Autenticação**: Obrigatória

**Content-Type**: `multipart/form-data`

**Form Data**:
```
file: [arquivo]
type: "thumbnail|video|material"
courseId: "uuid-curso-123"
```

**Parâmetros**:
- `file` (file): Arquivo a ser enviado
- `type` (string): Tipo de arquivo
- `courseId` (string, opcional): ID do curso relacionado

**Tipos Suportados**:

**Thumbnail**:
- Formatos: JPEG, PNG, WebP
- Uso: Imagem de capa do curso

**Video**:
- Formatos: MP4, WebM, AVI, MOV
- Tamanho máximo: 500MB
- Uso: Vídeo-aulas

**Material**:
- Formatos: PDF, ZIP, DOC, TXT
- Uso: Materiais complementares

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "url": "https://exemplo.com/uploads/videos/video_abc123def456.mp4",
  "type": "video",
  "filename": "video_abc123def456.mp4",
  "size": 52428800,
  "duration": 1800
}
```

**Para Vídeos**:
```json
{
  "success": true,
  "url": "https://exemplo.com/uploads/videos/video_abc123.mp4",
  "type": "video",
  "filename": "video_abc123.mp4",
  "size": 52428800,
  "duration": 1800,
  "quality": "1080p",
  "thumbnail": "https://exemplo.com/uploads/thumbnails/video_abc123_thumb.jpg"
}
```

**Processamento Automático**:
- **Geração de Thumbnails**: Para vídeos
- **Extração de Duração**: Para mídia
- **Compressão**: Para otimização
- **Nome Único**: UUID para evitar conflitos

**Validações**:
- Tipo de arquivo permitido
- Tamanho máximo (500MB)
- Formato válido

**Uso**: Upload de conteúdo educacional

**Erros Possíveis**:
- `400`: Arquivo ausente ou tipo inválido
- `400`: Arquivo muito grande
- `401`: Token inválido
- `500`: Erro no processamento

---

## 🔧 Configurações Avançadas

### Níveis de Curso
```javascript
const levels = {
  beginner: 'Iniciante',
  intermediate: 'Intermediário',
  advanced: 'Avançado',
  expert: 'Especialista'
};
```

### Categorias Disponíveis
```javascript
const categories = [
  'programacao',
  'design',
  'marketing',
  'negocios',
  'idiomas',
  'tecnologia',
  'saude',
  'arte'
];
```

### Status de Curso
```javascript
const statuses = {
  draft: 'Rascunho',
  published: 'Publicado',
  archived: 'Arquivado',
  maintenance: 'Em manutenção'
};
```

### Configurações de Acesso
```javascript
const settings = {
  allowDownloads: boolean,      // Permite downloads
  hasCertificate: boolean,      // Oferece certificado
  accessDuration: number,       // Dias de acesso
  maxStudents: number,          // Limite de alunos
  allowComments: boolean,       // Permite comentários
  showProgress: boolean,        // Mostra progresso
  autoplay: boolean,           // Autoplay de vídeos
  watermark: boolean           // Marca d'água
};
```

---

## 📊 Integrações e Automações

### Webhook de Matrícula
Quando pagamento é aprovado:
```javascript
// 1. Webhook recebe confirmação
// 2. Busca pedido relacionado
// 3. Cria matrícula automática
// 4. Envia email de boas-vindas
// 5. Libera acesso ao curso
```

### Sistema de Gamificação
```javascript
// TODO: Implementar
const gamification = {
  points: {
    lesson_completed: 10,
    module_completed: 50,
    course_completed: 100
  },
  badges: [
    'first_lesson',
    'fast_learner',
    'completionist'
  ]
};
```

### Certificados Automáticos
```javascript
// TODO: Implementar
const certificate = {
  template: 'certificate_template.pdf',
  variables: ['student_name', 'course_title', 'completion_date'],
  signature: 'instructor_signature.png'
};
```

---

## 🚀 Melhorias Futuras

### Sistema de Comentários
- Comentários por aula
- Respostas do instrutor
- Sistema de likes/dislikes
- Moderação automática

### Analytics Avançados
- Tempo médio por aula
- Taxa de abandono por módulo
- Engagement por tipo de conteúdo
- Relatórios de progresso

### Recursos Interativos
- Quizzes integrados
- Exercícios práticos
- Projetos finais
- Peer review

### Otimizações de Performance
- CDN para vídeos
- Streaming adaptativo
- Cache inteligente
- Compressão automática

---

**Próxima documentação**: `routes/admin.js` - Sistema administrativo
