# Documentação API - Área de Membros do Estudante

## Visão Geral
Sistema completo de área de membros para estudantes, incluindo dashboard, player de vídeo, controle de progresso, anotações e materiais complementares.

## Endpoints Disponíveis

### 1. Dashboard do Estudante
**GET** `/api/student/dashboard`

**Autenticação:** Requerida

**Descrição:** Dashboard principal do estudante com estatísticas, cursos em andamento e atividades recentes.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "student": {
    "id": "uuid-estudante",
    "name": "João Silva",
    "email": "joao@email.com",
    "avatar": "https://cdn.example.com/avatars/joao.jpg",
    "memberSince": "2024-01-01T00:00:00Z"
  },
  "stats": {
    "totalCourses": 5,
    "completedCourses": 2,
    "inProgressCourses": 3,
    "totalTimeSpent": 1450,
    "certificatesEarned": 2,
    "currentStreak": 7,
    "longestStreak": 15
  },
  "recentActivity": [
    {
      "type": "lesson_completed",
      "courseId": "uuid-curso",
      "courseTitle": "Curso de Marketing Digital",
      "lessonId": "uuid-aula",
      "lessonTitle": "Introdução ao Marketing",
      "timestamp": "2024-01-15T10:30:00Z"
    }
  ],
  "continueWatching": [
    {
      "courseId": "uuid-curso",
      "courseTitle": "Curso de Marketing Digital",
      "courseThumbnail": "https://cdn.example.com/courses/thumb.jpg",
      "progress": 45.5,
      "lastAccessed": "2024-01-15T09:15:00Z"
    }
  ],
  "achievements": []
}
```

**Métricas do Dashboard:**
- `totalTimeSpent`: Tempo em minutos estudados
- `currentStreak`: Dias consecutivos estudando
- `longestStreak`: Maior sequência de dias estudando
- `certificatesEarned`: Certificados obtidos

### 2. Página do Curso na Área de Membros
**GET** `/api/student/courses/:courseId`

**Autenticação:** Requerida

**Descrição:** Carrega dados completos do curso para a área de membros, incluindo módulos, aulas e progresso.

**Parâmetros de Rota:**
- `courseId` (string): ID do curso

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "course": {
    "id": "uuid-curso",
    "title": "Curso de Marketing Digital",
    "description": "Aprenda marketing digital do zero ao avançado",
    "thumbnailUrl": "https://cdn.example.com/courses/thumb.jpg",
    "totalLessons": 45,
    "totalDuration": 1800,
    "difficulty": "intermediate",
    "language": "pt-BR"
  },
  "instructor": {
    "id": "uuid-instrutor",
    "name": "Professor João",
    "avatar": "https://cdn.example.com/avatars/prof.jpg",
    "bio": "Especialista em marketing digital"
  },
  "enrollment": {
    "enrolledAt": "2024-01-01T10:00:00Z",
    "lastAccessed": "2024-01-15T09:15:00Z",
    "progressPercentage": 45.5,
    "timeSpent": 850,
    "status": "active"
  },
  "currentLesson": {
    "id": "uuid-aula",
    "moduleId": "uuid-modulo",
    "title": "Fundamentos do Marketing Digital",
    "watchedPercentage": 25.5,
    "lastPosition": 128
  },
  "nextLesson": {
    "id": "uuid-proxima-aula",
    "moduleId": "uuid-modulo",
    "title": "Criando Campanhas Eficazes",
    "isLocked": false
  },
  "modules": [
    {
      "id": "uuid-modulo",
      "title": "Módulo 1: Fundamentos",
      "description": "Base do marketing digital",
      "order": 1,
      "totalLessons": 8,
      "completedLessons": 3,
      "progressPercentage": 37.5,
      "lessons": [
        {
          "id": "uuid-aula",
          "title": "Introdução ao Marketing",
          "description": "Conceitos básicos",
          "type": "video",
          "duration": 15,
          "order": 1,
          "isCompleted": true,
          "completedAt": "2024-01-10T14:30:00Z",
          "timeSpent": 18,
          "watchedPercentage": 100,
          "isPreview": false,
          "isLocked": false,
          "hasAccess": true,
          "thumbnailUrl": "https://cdn.example.com/lessons/thumb.jpg",
          "materials": [
            {
              "id": "uuid-material",
              "title": "Slides da Aula",
              "type": "pdf",
              "fileUrl": "https://cdn.example.com/materials/slides.pdf"
            }
          ]
        }
      ]
    }
  ]
}
```

**Erros Possíveis:**
- `403` - Sem acesso ao curso
- `404` - Curso não encontrado

### 3. Player de Vídeo da Aula
**GET** `/api/student/courses/:courseId/lessons/:lessonId/player`

**Autenticação:** Requerida

**Descrição:** Carrega dados para o player de vídeo, incluindo URLs seguras e progresso salvo.

**Parâmetros de Rota:**
- `courseId` (string): ID do curso
- `lessonId` (string): ID da aula

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "lesson": {
    "id": "uuid-aula",
    "title": "Fundamentos do Marketing Digital",
    "description": "Aprenda os conceitos básicos",
    "type": "video",
    "duration": 900,
    "videoUrl": "https://secure.videohost.com/video/encrypted-url",
    "videoProvider": "vimeo",
    "videoId": "123456789",
    "thumbnailUrl": "https://cdn.example.com/lessons/thumb.jpg",
    "hasTranscription": true,
    "allowDownload": true,
    "allowSpeed": true,
    "qualityOptions": ["360p", "720p", "1080p"]
  },
  "module": {
    "id": "uuid-modulo",
    "title": "Módulo 1: Fundamentos",
    "course": {
      "id": "uuid-curso",
      "title": "Curso de Marketing Digital",
      "allowDownloads": true
    }
  },
  "progress": {
    "watchedPercentage": 25.5,
    "lastPosition": 230,
    "timeSpent": 45,
    "completed": false,
    "completedAt": null,
    "rating": null,
    "feedback": null,
    "lastWatched": "2024-01-15T09:15:00Z"
  },
  "materials": [
    {
      "id": "uuid-material",
      "title": "Slides da Aula",
      "description": "Material complementar",
      "type": "pdf",
      "fileSize": 2048576,
      "downloadUrl": "/api/student/courses/uuid-curso/lessons/uuid-aula/materials/uuid-material/download"
    }
  ],
  "navigation": {
    "previousLesson": {
      "id": "uuid-anterior",
      "title": "Aula Anterior"
    },
    "nextLesson": {
      "id": "uuid-proxima",
      "title": "Próxima Aula",
      "isLocked": false
    }
  },
  "playerSettings": {
    "autoPlay": false,
    "continueFromLastPosition": true,
    "trackProgress": true,
    "enableNotes": true,
    "showTranscription": true
  }
}
```

### 4. Marcar Aula como Concluída
**POST** `/api/student/courses/:courseId/lessons/:lessonId/complete`

**Autenticação:** Requerida

**Descrição:** Marca uma aula como concluída e atualiza o progresso do curso.

**Parâmetros de Rota:**
- `courseId` (string): ID do curso
- `lessonId` (string): ID da aula

**Body da Requisição:**
```json
{
  "timeSpent": 870,
  "watchedPercentage": 95.5,
  "finalPosition": 865,
  "rating": 5,
  "feedback": "Excelente aula, muito didática!"
}
```

**Parâmetros:**
- `timeSpent` (number, opcional): Tempo gasto em segundos
- `watchedPercentage` (number, opcional): Percentual assistido
- `finalPosition` (number, opcional): Posição final em segundos
- `rating` (number, opcional): Avaliação de 1 a 5
- `feedback` (string, opcional): Comentário sobre a aula

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Aula marcada como concluída",
  "progress": {
    "lessonId": "uuid-aula",
    "completed": true,
    "completedAt": "2024-01-15T14:30:00Z",
    "timeSpent": 870,
    "watchedPercentage": 95.5,
    "rating": 5
  },
  "courseProgress": {
    "totalLessons": 45,
    "completedLessons": 23,
    "progressPercentage": 51.1,
    "totalTimeSpent": 2150
  },
  "nextLesson": {
    "id": "uuid-proxima",
    "title": "Próxima Aula",
    "moduleTitle": "Módulo 2"
  },
  "achievements": [
    {
      "type": "first_completion",
      "title": "Primeira Aula Concluída!",
      "description": "Você completou sua primeira aula"
    }
  ]
}
```

### 5. Download de Material da Aula
**GET** `/api/student/courses/:courseId/lessons/:lessonId/materials/:materialId/download`

**Autenticação:** Requerida

**Descrição:** Faz download de material complementar da aula (PDF, arquivos, etc.).

**Parâmetros de Rota:**
- `courseId` (string): ID do curso
- `lessonId` (string): ID da aula
- `materialId` (string): ID do material

**Resposta de Sucesso (200):**
- Redirect para URL de download segura ou stream do arquivo
- Headers apropriados para download (Content-Disposition, Content-Type)

**Erros Possíveis:**
- `403` - Sem acesso ao curso ou downloads desabilitados
- `404` - Material não encontrado

### 6. Buscar no Curso
**GET** `/api/student/courses/:courseId/search`

**Autenticação:** Requerida

**Descrição:** Busca por conteúdo dentro do curso (aulas, transcrições, materiais).

**Parâmetros de Rota:**
- `courseId` (string): ID do curso

**Parâmetros de Query:**
- `q` (string): Termo de busca
- `type` (string, opcional): Tipo de conteúdo (`lesson`, `material`, `transcript`)

**Exemplo de Requisição:**
```
GET /api/student/courses/uuid-curso/search?q=marketing&type=lesson
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "query": "marketing",
  "results": {
    "lessons": [
      {
        "id": "uuid-aula",
        "title": "Fundamentos do Marketing Digital",
        "moduleTitle": "Módulo 1",
        "matches": [
          {
            "type": "title",
            "text": "Fundamentos do <mark>Marketing</mark> Digital"
          },
          {
            "type": "description",
            "text": "Aprenda os conceitos de <mark>marketing</mark>"
          }
        ]
      }
    ],
    "materials": [
      {
        "id": "uuid-material",
        "title": "Guia de Marketing",
        "lessonTitle": "Aula 1",
        "type": "pdf"
      }
    ],
    "transcripts": [
      {
        "lessonId": "uuid-aula",
        "lessonTitle": "Aula 2",
        "timestamp": 120,
        "text": "O <mark>marketing</mark> digital é fundamental..."
      }
    ]
  },
  "totalResults": 15
}
```

### 7. Criar Anotação na Aula
**POST** `/api/student/courses/:courseId/lessons/:lessonId/notes`

**Autenticação:** Requerida

**Descrição:** Cria uma anotação pessoal em uma posição específica da aula.

**Parâmetros de Rota:**
- `courseId` (string): ID do curso
- `lessonId` (string): ID da aula

**Body da Requisição:**
```json
{
  "content": "Ponto importante sobre segmentação de público",
  "timestamp": 245,
  "isPrivate": true
}
```

**Parâmetros:**
- `content` (string, obrigatório): Conteúdo da anotação
- `timestamp` (number, opcional): Posição em segundos no vídeo
- `isPrivate` (boolean, opcional): Se a anotação é privada (padrão: true)

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "note": {
    "id": "uuid-nota",
    "content": "Ponto importante sobre segmentação de público",
    "timestamp": 245,
    "isPrivate": true,
    "createdAt": "2024-01-15T14:30:00Z"
  }
}
```

### 8. Listar Anotações do Curso
**GET** `/api/student/courses/:courseId/notes`

**Autenticação:** Requerida

**Descrição:** Lista todas as anotações do usuário no curso.

**Parâmetros de Rota:**
- `courseId` (string): ID do curso

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "notes": [
    {
      "id": "uuid-nota",
      "content": "Ponto importante sobre segmentação",
      "timestamp": 245,
      "lessonId": "uuid-aula",
      "lessonTitle": "Fundamentos do Marketing",
      "moduleTitle": "Módulo 1",
      "isPrivate": true,
      "createdAt": "2024-01-15T14:30:00Z",
      "updatedAt": "2024-01-15T14:30:00Z"
    }
  ],
  "total": 15
}
```

## Controle de Acesso e Segurança

### Verificação de Matrícula
```javascript
// Todas as rotas verificam se o usuário tem matrícula ativa
const enrollment = await CourseEnrollment.findOne({
  where: {
    student_id: req.user.id,
    course_id: courseId,
    status: 'active'
  }
});

if (!enrollment) {
  return res.status(403).json({
    error: 'Você não tem acesso a este curso',
    type: 'access_denied'
  });
}
```

### URLs Seguras de Vídeo
- Geração de URLs temporárias para vídeos
- Proteção contra hotlinking
- Controle de acesso baseado em matrícula

### Controle de Downloads
- Verificação de permissão de download por curso
- Logs de downloads para auditoria
- Limitação de downloads por tempo/quantidade

## Sistema de Progresso

### Cálculo de Progresso
```javascript
// Progresso da aula individual
const lessonProgress = {
  watchedPercentage: 95.5,
  timeSpent: 870, // segundos
  completed: watchedPercentage >= 80, // 80% para marcar como concluída
  lastPosition: 865
};

// Progresso do curso total
const courseProgress = {
  completedLessons: 23,
  totalLessons: 45,
  progressPercentage: (completedLessons / totalLessons) * 100
};
```

### Critérios de Conclusão
- Aula considerada concluída com 80%+ assistida
- Curso concluído quando todas as aulas obrigatórias são finalizadas
- Certificado liberado após conclusão do curso

## Recursos Avançados

### Sistema de Conquistas
- Primeira aula concluída
- Módulo completo
- Curso finalizado
- Streak de dias estudando
- Tempo total de estudo

### Navegação Inteligente
- Próxima aula sugerida automaticamente
- Desbloqueio progressivo de conteúdo
- Continuação de onde parou

### Busca Avançada
- Busca em títulos, descrições e transcrições
- Highlights dos termos encontrados
- Filtros por tipo de conteúdo
- Jump para posição específica em transcrições

## Exemplos de Uso

### Carregar Dashboard
```javascript
const dashboard = await fetch('/api/student/dashboard', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
const data = await dashboard.json();
console.log(`Progresso geral: ${data.stats.completedCourses}/${data.stats.totalCourses} cursos`);
```

### Player de Vídeo
```javascript
// Carregar dados do player
const playerData = await fetch(`/api/student/courses/${courseId}/lessons/${lessonId}/player`, {
  headers: { 'Authorization': `Bearer ${token}` }
});

const lesson = await playerData.json();

// Configurar player
const player = new VideoPlayer({
  videoUrl: lesson.lesson.videoUrl,
  startTime: lesson.progress.lastPosition,
  onProgress: (currentTime, duration) => {
    // Salvar progresso periodicamente
    saveProgress(currentTime, duration);
  }
});
```

### Salvar Progresso
```javascript
const saveProgress = async (currentTime, duration) => {
  const watchedPercentage = (currentTime / duration) * 100;
  
  if (watchedPercentage >= 80) {
    // Marcar como concluída
    await fetch(`/api/student/courses/${courseId}/lessons/${lessonId}/complete`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        timeSpent: Math.floor(currentTime),
        watchedPercentage,
        finalPosition: currentTime
      })
    });
  }
};
```

### Sistema de Anotações
```javascript
// Criar anotação
const createNote = async (content, timestamp) => {
  await fetch(`/api/student/courses/${courseId}/lessons/${lessonId}/notes`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      content,
      timestamp,
      isPrivate: true
    })
  });
};

// Carregar anotações
const loadNotes = async () => {
  const response = await fetch(`/api/student/courses/${courseId}/notes`, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  return await response.json();
};
```

## Observações Técnicas

1. **Performance**: Consultas otimizadas com includes apropriados
2. **Segurança**: Verificação rigorosa de acesso em todas as rotas
3. **Escalabilidade**: Sistema preparado para milhares de estudantes simultâneos
4. **Analytics**: Tracking detalhado de progresso e engagement
5. **Offline**: Suporte a downloads para acesso offline
6. **Responsivo**: API preparada para web e mobile
