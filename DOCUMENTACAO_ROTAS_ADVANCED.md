# Documentação - Sistema Avançado (routes/advanced.js)

## Visão Geral
Sistema de funcionalidades avançadas que inclui certificados de conclusão, avaliações de cursos, configurações de player e fórum de discussões. Complementa o sistema EAD com recursos de engagement e validação.

## Características Principais
- **Certificados Digitais**: Geração automática com código de verificação
- **Sistema de Avaliações**: Reviews e ratings com aspectos detalhados
- **Player Personalizado**: Configurações de reprodução e aprendizado
- **Fórum de Discussões**: Comunidade e suporte entre estudantes
- **Verificação de Autenticidade**: Sistema de validação de certificados

## Funcionalidades Implementadas

### 🏆 Certificados Digitais
- Geração automática ao completar 100% do curso
- Código de verificação único
- PDF downloadável
- Verificação pública de autenticidade
- Assinatura digital do instrutor

### ⭐ Sistema de Avaliações
- Rating de 1 a 5 estrelas
- Comentários detalhados
- Avaliação por aspectos (conteúdo, instrutor, dificuldade, suporte)
- Reviews verificadas (apenas quem completou)
- Estatísticas agregadas

### ⚙️ Configurações do Player
- Velocidade de reprodução
- Qualidade de vídeo
- Legendas e idiomas
- Autoplay e lembrar posição
- Modo escuro
- Notificações personalizadas

### 💬 Fórum de Discussões
- Tópicos e perguntas
- Sistema de respostas
- Votes up/down
- Tópicos fixados
- Filtragem por tipo e lição

## Rotas Implementadas

### 1. Certificados

#### `GET /api/advanced/courses/:courseId/certificate`
**Descrição**: Obter certificado de conclusão do curso
**Acesso**: Usuário autenticado e matriculado

**Response - Usuário Elegível**:
```json
{
  "certificate": {
    "id": "cert-12345",
    "courseId": "course-123",
    "courseTitle": "Desenvolvimento Web Completo",
    "studentName": "João Silva",
    "completedAt": "2024-01-01T15:30:00Z",
    "issueDate": "2024-01-01T15:30:00Z",
    "certificateUrl": "https://api.exemplo.com/certificates/DEV-2024-abc12345.pdf",
    "verificationCode": "DEV-2024-abc12345",
    "verificationUrl": "https://api.exemplo.com/verify/DEV-2024-abc12345",
    "instructorName": "Prof. Maria Santos",
    "instructorSignature": "https://exemplo.com/signatures/maria.png",
    "courseDuration": "40 horas",
    "grade": "A+",
    "skills": [
      "HTML5 e CSS3",
      "JavaScript ES6+",
      "React.js",
      "Node.js",
      "Banco de Dados"
    ]
  },
  "isEligible": true,
  "requirements": {
    "minProgress": 100,
    "currentProgress": 100,
    "minTimeSpent": 0,
    "currentTimeSpent": 2400,
    "allLessonsCompleted": true
  }
}
```

**Response - Usuário Não Elegível**:
```json
{
  "certificate": null,
  "isEligible": false,
  "requirements": {
    "minProgress": 100,
    "currentProgress": 85,
    "minTimeSpent": 0,
    "currentTimeSpent": 1800,
    "allLessonsCompleted": false
  }
}
```

**Response Error - Curso Sem Certificado**:
```json
{
  "error": "Este curso não oferece certificado",
  "type": "certificate_disabled"
}
```

#### `GET /api/advanced/verify/:verificationCode`
**Descrição**: Verificar autenticidade de um certificado
**Acesso**: Público (não requer autenticação)

**Response - Certificado Válido**:
```json
{
  "valid": true,
  "certificate": {
    "verificationCode": "DEV-2024-abc12345",
    "studentName": "João Silva",
    "courseTitle": "Desenvolvimento Web Completo",
    "instructorName": "Prof. Maria Santos",
    "issueDate": "2024-01-01T15:30:00Z",
    "courseDuration": "40 horas",
    "grade": "A+",
    "skills": [
      "HTML5 e CSS3",
      "JavaScript ES6+",
      "React.js",
      "Node.js",
      "Banco de Dados"
    ]
  }
}
```

**Response - Certificado Inválido**:
```json
{
  "valid": false,
  "error": "Certificado não encontrado ou inválido"
}
```

### 2. Sistema de Avaliações

#### `POST /api/advanced/courses/:courseId/review`
**Descrição**: Criar ou atualizar avaliação do curso
**Acesso**: Usuário autenticado e matriculado

```javascript
// Payload de exemplo
{
  "rating": 5,
  "comment": "Excelente curso! Conteúdo muito bem estruturado e professor didático.",
  "wouldRecommend": true,
  "aspects": {
    "content": 5,
    "instructor": 5,
    "difficulty": 4,
    "support": 5
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Avaliação criada com sucesso",
  "review": {
    "id": "review-123",
    "rating": 5,
    "comment": "Excelente curso! Conteúdo muito bem estruturado e professor didático.",
    "wouldRecommend": true,
    "isVerified": true,
    "createdAt": "2024-01-01T16:00:00Z"
  }
}
```

**Response Error - Rating Inválido**:
```json
{
  "error": "Rating deve ser entre 1 e 5",
  "type": "invalid_rating"
}
```

**Response Error - Não Matriculado**:
```json
{
  "error": "Você precisa estar matriculado para avaliar este curso",
  "type": "not_enrolled"
}
```

#### `GET /api/advanced/courses/:courseId/reviews`
**Descrição**: Buscar avaliações de um curso
**Acesso**: Público

**Query Parameters**:
- `page`: Página (padrão: 1)
- `limit`: Itens por página (padrão: 10)
- `sort`: Ordenação (recent, rating_high, rating_low)

**Response Success**:
```json
{
  "reviews": [
    {
      "id": "review-123",
      "rating": 5,
      "comment": "Excelente curso! Conteúdo muito bem estruturado.",
      "wouldRecommend": true,
      "isVerified": true,
      "isFeatured": false,
      "createdAt": "2024-01-01T16:00:00Z",
      "user": {
        "name": "João Silva",
        "avatar": "https://exemplo.com/avatars/joao.jpg"
      },
      "aspects": {
        "content": 5,
        "instructor": 5,
        "difficulty": 4,
        "support": 5
      }
    },
    {
      "id": "review-124",
      "rating": 4,
      "comment": "Bom curso, mas poderia ter mais exercícios práticos.",
      "wouldRecommend": true,
      "isVerified": true,
      "isFeatured": true,
      "createdAt": "2024-01-01T14:30:00Z",
      "user": {
        "name": "Maria Oliveira",
        "avatar": "https://exemplo.com/avatars/maria.jpg"
      },
      "aspects": {
        "content": 4,
        "instructor": 5,
        "difficulty": 3,
        "support": 4
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 47,
    "totalPages": 5
  },
  "stats": {
    "averageRating": "4.6",
    "totalReviews": 47,
    "distribution": {
      "1": 1,
      "2": 2,
      "3": 5,
      "4": 15,
      "5": 24
    }
  }
}
```

### 3. Configurações do Player

#### `GET /api/advanced/player-settings`
**Descrição**: Obter configurações do player do estudante
**Acesso**: Usuário autenticado

**Response Success**:
```json
{
  "success": true,
  "settings": {
    "autoPlay": false,
    "defaultQuality": "720p",
    "playbackSpeed": 1.0,
    "subtitlesEnabled": true,
    "subtitlesLanguage": "pt-BR",
    "rememberPosition": true,
    "skipIntro": false,
    "darkMode": true,
    "notifications": {
      "emailNewLessons": true,
      "emailProgressReminder": true,
      "emailCertificateReady": true,
      "emailCommunityMessages": false,
      "pushStudyReminder": true,
      "pushNewContent": true,
      "pushAchievements": true,
      "frequency": "daily"
    },
    "study": {
      "dailyGoalMinutes": 60,
      "reminderTime": "19:00",
      "timezone": "America/Sao_Paulo"
    }
  }
}
```

#### `PUT /api/advanced/player-settings`
**Descrição**: Atualizar configurações do player
**Acesso**: Usuário autenticado

```javascript
// Payload de exemplo
{
  "autoPlay": true,
  "defaultQuality": "1080p",
  "playbackSpeed": 1.25,
  "subtitlesEnabled": true,
  "subtitlesLanguage": "pt-BR",
  "rememberPosition": true,
  "skipIntro": true,
  "darkMode": false,
  "notifications": {
    "emailNewLessons": true,
    "emailProgressReminder": false,
    "emailCertificateReady": true,
    "emailCommunityMessages": true,
    "pushStudyReminder": true,
    "pushNewContent": false,
    "pushAchievements": true,
    "frequency": "weekly"
  },
  "study": {
    "dailyGoalMinutes": 90,
    "reminderTime": "20:30",
    "timezone": "America/Sao_Paulo"
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Configurações atualizadas com sucesso"
}
```

### 4. Fórum de Discussões

#### `GET /api/advanced/courses/:courseId/discussions`
**Descrição**: Buscar discussões de um curso
**Acesso**: Usuário autenticado e matriculado

**Query Parameters**:
- `lessonId`: ID da lição específica
- `page`: Página (padrão: 1)
- `limit`: Itens por página (padrão: 10)
- `type`: Tipo de discussão (all, questions, pinned)

**Response Success**:
```json
{
  "discussions": [
    {
      "id": "discussion-123",
      "title": "Como implementar autenticação JWT?",
      "content": "Estou com dificuldades para implementar a autenticação...",
      "isQuestion": true,
      "isAnswered": true,
      "isPinned": false,
      "votesUp": 5,
      "votesDown": 0,
      "createdAt": "2024-01-01T10:00:00Z",
      "user": {
        "name": "Pedro Santos",
        "avatar": "https://exemplo.com/avatars/pedro.jpg"
      },
      "repliesCount": 3,
      "latestReply": {
        "id": "reply-456",
        "content": "Você pode usar a biblioteca jsonwebtoken...",
        "createdAt": "2024-01-01T11:30:00Z",
        "user": {
          "name": "Prof. Maria Santos",
          "avatar": "https://exemplo.com/avatars/maria.jpg"
        }
      }
    },
    {
      "id": "discussion-124",
      "title": "Boas práticas para estrutura de pastas",
      "content": "Gostaria de compartilhar uma estrutura que funciona bem...",
      "isQuestion": false,
      "isAnswered": false,
      "isPinned": true,
      "votesUp": 12,
      "votesDown": 1,
      "createdAt": "2024-01-01T08:00:00Z",
      "user": {
        "name": "Ana Costa",
        "avatar": "https://exemplo.com/avatars/ana.jpg"
      },
      "repliesCount": 7,
      "latestReply": {
        "id": "reply-789",
        "content": "Muito útil! Obrigado por compartilhar.",
        "createdAt": "2024-01-01T15:00:00Z",
        "user": {
          "name": "Carlos Lima",
          "avatar": "https://exemplo.com/avatars/carlos.jpg"
        }
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 15,
    "totalPages": 2
  }
}
```

#### `POST /api/advanced/courses/:courseId/discussions`
**Descrição**: Criar nova discussão
**Acesso**: Usuário autenticado e matriculado

```javascript
// Payload de exemplo
{
  "lessonId": "lesson-123", // Opcional
  "title": "Dúvida sobre CSS Grid",
  "content": "Como posso criar um layout responsivo usando CSS Grid?",
  "isQuestion": true
}
```

**Response Success**:
```json
{
  "success": true,
  "discussion": {
    "id": "discussion-125",
    "title": "Dúvida sobre CSS Grid",
    "content": "Como posso criar um layout responsivo usando CSS Grid?",
    "isQuestion": true,
    "createdAt": "2024-01-01T17:00:00Z"
  }
}
```

**Response Error - Campos Obrigatórios**:
```json
{
  "error": "Título e conteúdo são obrigatórios",
  "type": "missing_fields"
}
```

**Response Error - Acesso Negado**:
```json
{
  "error": "Você não tem acesso a este curso",
  "type": "access_denied"
}
```

## Integração Frontend

### Componente de Certificados
```jsx
import React, { useState, useEffect } from 'react';

const CertificateViewer = ({ courseId, token }) => {
  const [certificate, setCertificate] = useState(null);
  const [requirements, setRequirements] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchCertificate();
  }, [courseId]);
  
  const fetchCertificate = async () => {
    try {
      const response = await fetch(`/api/advanced/courses/${courseId}/certificate`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      
      setCertificate(data.certificate);
      setRequirements(data.requirements);
    } catch (error) {
      console.error('Erro ao buscar certificado:', error);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <div>Carregando certificado...</div>;
  
  if (!certificate) {
    return (
      <div className="certificate-requirements">
        <h3>Certificado não disponível</h3>
        <div className="requirements">
          <h4>Requisitos para obter o certificado:</h4>
          <ul>
            <li>
              Progresso: {requirements.currentProgress}% / {requirements.minProgress}%
              {requirements.currentProgress >= requirements.minProgress ? ' ✅' : ' ❌'}
            </li>
            <li>
              Todas as lições completadas: {requirements.allLessonsCompleted ? '✅' : '❌'}
            </li>
          </ul>
        </div>
      </div>
    );
  }
  
  return (
    <div className="certificate-container">
      <div className="certificate-preview">
        <h3>🏆 Certificado de Conclusão</h3>
        <div className="certificate-info">
          <h4>{certificate.courseTitle}</h4>
          <p><strong>Aluno:</strong> {certificate.studentName}</p>
          <p><strong>Instrutor:</strong> {certificate.instructorName}</p>
          <p><strong>Duração:</strong> {certificate.courseDuration}</p>
          <p><strong>Nota:</strong> {certificate.grade}</p>
          <p><strong>Concluído em:</strong> {new Date(certificate.completedAt).toLocaleDateString()}</p>
        </div>
        
        <div className="certificate-skills">
          <h5>Habilidades Adquiridas:</h5>
          <div className="skills-list">
            {certificate.skills.map((skill, index) => (
              <span key={index} className="skill-tag">
                {skill}
              </span>
            ))}
          </div>
        </div>
        
        <div className="certificate-actions">
          <a 
            href={certificate.certificateUrl} 
            target="_blank" 
            rel="noopener noreferrer"
            className="btn btn-primary"
          >
            📄 Baixar Certificado PDF
          </a>
          
          <button 
            onClick={() => navigator.clipboard.writeText(certificate.verificationUrl)}
            className="btn btn-secondary"
          >
            🔗 Copiar Link de Verificação
          </button>
        </div>
        
        <div className="verification-info">
          <p><strong>Código de Verificação:</strong> {certificate.verificationCode}</p>
          <p><small>Verifique a autenticidade em: {certificate.verificationUrl}</small></p>
        </div>
      </div>
    </div>
  );
};
```

### Componente de Avaliações
```jsx
const CourseReviews = ({ courseId, token }) => {
  const [reviews, setReviews] = useState([]);
  const [stats, setStats] = useState(null);
  const [userReview, setUserReview] = useState(null);
  const [showReviewForm, setShowReviewForm] = useState(false);
  
  const [newReview, setNewReview] = useState({
    rating: 5,
    comment: '',
    wouldRecommend: true,
    aspects: {
      content: 5,
      instructor: 5,
      difficulty: 4,
      support: 5
    }
  });
  
  const submitReview = async () => {
    try {
      const response = await fetch(`/api/advanced/courses/${courseId}/review`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(newReview)
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Avaliação enviada com sucesso!');
        setShowReviewForm(false);
        fetchReviews();
      }
    } catch (error) {
      alert('Erro ao enviar avaliação: ' + error.message);
    }
  };
  
  const renderStars = (rating) => {
    return Array.from({ length: 5 }, (_, i) => (
      <span key={i} className={`star ${i < rating ? 'filled' : ''}`}>
        ⭐
      </span>
    ));
  };
  
  return (
    <div className="course-reviews">
      <div className="reviews-header">
        <h3>Avaliações do Curso</h3>
        
        {stats && (
          <div className="reviews-stats">
            <div className="average-rating">
              <span className="rating-value">{stats.averageRating}</span>
              {renderStars(Math.round(stats.averageRating))}
              <span className="total-reviews">({stats.totalReviews} avaliações)</span>
            </div>
            
            <div className="rating-distribution">
              {Object.entries(stats.distribution).reverse().map(([stars, count]) => (
                <div key={stars} className="rating-bar">
                  <span>{stars}⭐</span>
                  <div className="bar">
                    <div 
                      className="fill" 
                      style={{ width: `${(count / stats.totalReviews) * 100}%` }}
                    />
                  </div>
                  <span>{count}</span>
                </div>
              ))}
            </div>
          </div>
        )}
        
        <button 
          onClick={() => setShowReviewForm(true)}
          className="btn btn-primary"
        >
          ✍️ Avaliar Curso
        </button>
      </div>
      
      {showReviewForm && (
        <div className="review-form">
          <h4>Sua Avaliação</h4>
          
          <div className="rating-input">
            <label>Nota Geral:</label>
            {Array.from({ length: 5 }, (_, i) => (
              <button
                key={i}
                onClick={() => setNewReview({...newReview, rating: i + 1})}
                className={`star-btn ${i < newReview.rating ? 'active' : ''}`}
              >
                ⭐
              </button>
            ))}
          </div>
          
          <div className="aspects-rating">
            <h5>Avalie os Aspectos:</h5>
            {Object.entries(newReview.aspects).map(([aspect, rating]) => (
              <div key={aspect} className="aspect-rating">
                <label>{aspect}:</label>
                {Array.from({ length: 5 }, (_, i) => (
                  <button
                    key={i}
                    onClick={() => setNewReview({
                      ...newReview,
                      aspects: { ...newReview.aspects, [aspect]: i + 1 }
                    })}
                    className={`star-btn ${i < rating ? 'active' : ''}`}
                  >
                    ⭐
                  </button>
                ))}
              </div>
            ))}
          </div>
          
          <textarea
            value={newReview.comment}
            onChange={(e) => setNewReview({...newReview, comment: e.target.value})}
            placeholder="Compartilhe sua experiência com o curso..."
            rows="4"
          />
          
          <div className="recommend-input">
            <label>
              <input
                type="checkbox"
                checked={newReview.wouldRecommend}
                onChange={(e) => setNewReview({...newReview, wouldRecommend: e.target.checked})}
              />
              Recomendaria este curso
            </label>
          </div>
          
          <div className="form-actions">
            <button onClick={submitReview} className="btn btn-success">
              📝 Enviar Avaliação
            </button>
            <button 
              onClick={() => setShowReviewForm(false)} 
              className="btn btn-secondary"
            >
              Cancelar
            </button>
          </div>
        </div>
      )}
      
      <div className="reviews-list">
        {reviews.map(review => (
          <div key={review.id} className="review-item">
            <div className="review-header">
              <div className="user-info">
                <img src={review.user.avatar} alt={review.user.name} />
                <div>
                  <h5>{review.user.name}</h5>
                  {review.isVerified && <span className="verified">✅ Verificado</span>}
                </div>
              </div>
              
              <div className="review-rating">
                {renderStars(review.rating)}
                <span className="review-date">
                  {new Date(review.createdAt).toLocaleDateString()}
                </span>
              </div>
            </div>
            
            <div className="review-content">
              <p>{review.comment}</p>
              
              {review.wouldRecommend && (
                <span className="recommend-badge">👍 Recomenda</span>
              )}
            </div>
            
            <div className="review-aspects">
              <span>Conteúdo: {renderStars(review.aspects.content)}</span>
              <span>Instrutor: {renderStars(review.aspects.instructor)}</span>
              <span>Dificuldade: {renderStars(review.aspects.difficulty)}</span>
              <span>Suporte: {renderStars(review.aspects.support)}</span>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};
```

### Configurações do Player
```jsx
const PlayerSettings = ({ token }) => {
  const [settings, setSettings] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchSettings();
  }, []);
  
  const fetchSettings = async () => {
    try {
      const response = await fetch('/api/advanced/player-settings', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setSettings(data.settings);
    } catch (error) {
      console.error('Erro ao buscar configurações:', error);
    } finally {
      setLoading(false);
    }
  };
  
  const updateSettings = async (newSettings) => {
    try {
      const response = await fetch('/api/advanced/player-settings', {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(newSettings)
      });
      
      const data = await response.json();
      if (data.success) {
        setSettings(newSettings);
        alert('Configurações salvas com sucesso!');
      }
    } catch (error) {
      alert('Erro ao salvar configurações: ' + error.message);
    }
  };
  
  if (loading) return <div>Carregando configurações...</div>;
  
  return (
    <div className="player-settings">
      <h3>⚙️ Configurações do Player</h3>
      
      <div className="settings-section">
        <h4>Reprodução</h4>
        
        <label>
          <input
            type="checkbox"
            checked={settings.autoPlay}
            onChange={(e) => setSettings({...settings, autoPlay: e.target.checked})}
          />
          Reprodução automática
        </label>
        
        <label>
          Velocidade padrão:
          <select
            value={settings.playbackSpeed}
            onChange={(e) => setSettings({...settings, playbackSpeed: parseFloat(e.target.value)})}
          >
            <option value={0.5}>0.5x</option>
            <option value={0.75}>0.75x</option>
            <option value={1.0}>1.0x</option>
            <option value={1.25}>1.25x</option>
            <option value={1.5}>1.5x</option>
            <option value={2.0}>2.0x</option>
          </select>
        </label>
        
        <label>
          Qualidade padrão:
          <select
            value={settings.defaultQuality}
            onChange={(e) => setSettings({...settings, defaultQuality: e.target.value})}
          >
            <option value="360p">360p</option>
            <option value="720p">720p</option>
            <option value="1080p">1080p</option>
            <option value="auto">Automática</option>
          </select>
        </label>
        
        <label>
          <input
            type="checkbox"
            checked={settings.rememberPosition}
            onChange={(e) => setSettings({...settings, rememberPosition: e.target.checked})}
          />
          Lembrar posição do vídeo
        </label>
      </div>
      
      <div className="settings-section">
        <h4>Legendas</h4>
        
        <label>
          <input
            type="checkbox"
            checked={settings.subtitlesEnabled}
            onChange={(e) => setSettings({...settings, subtitlesEnabled: e.target.checked})}
          />
          Legendas habilitadas
        </label>
        
        <label>
          Idioma das legendas:
          <select
            value={settings.subtitlesLanguage}
            onChange={(e) => setSettings({...settings, subtitlesLanguage: e.target.value})}
          >
            <option value="pt-BR">Português (Brasil)</option>
            <option value="en">English</option>
            <option value="es">Español</option>
          </select>
        </label>
      </div>
      
      <div className="settings-section">
        <h4>Estudo</h4>
        
        <label>
          Meta diária (minutos):
          <input
            type="number"
            value={settings.study.dailyGoalMinutes}
            onChange={(e) => setSettings({
              ...settings,
              study: { ...settings.study, dailyGoalMinutes: parseInt(e.target.value) }
            })}
            min="15"
            max="480"
          />
        </label>
        
        <label>
          Horário de lembrete:
          <input
            type="time"
            value={settings.study.reminderTime}
            onChange={(e) => setSettings({
              ...settings,
              study: { ...settings.study, reminderTime: e.target.value }
            })}
          />
        </label>
      </div>
      
      <div className="settings-section">
        <h4>Notificações</h4>
        
        <label>
          <input
            type="checkbox"
            checked={settings.notifications.emailNewLessons}
            onChange={(e) => setSettings({
              ...settings,
              notifications: { ...settings.notifications, emailNewLessons: e.target.checked }
            })}
          />
          Email para novas aulas
        </label>
        
        <label>
          <input
            type="checkbox"
            checked={settings.notifications.pushStudyReminder}
            onChange={(e) => setSettings({
              ...settings,
              notifications: { ...settings.notifications, pushStudyReminder: e.target.checked }
            })}
          />
          Lembrete de estudos
        </label>
        
        <label>
          <input
            type="checkbox"
            checked={settings.notifications.emailCertificateReady}
            onChange={(e) => setSettings({
              ...settings,
              notifications: { ...settings.notifications, emailCertificateReady: e.target.checked }
            })}
          />
          Email quando certificado estiver pronto
        </label>
      </div>
      
      <button 
        onClick={() => updateSettings(settings)}
        className="btn btn-primary"
      >
        💾 Salvar Configurações
      </button>
    </div>
  );
};
```

## Códigos de Erro

| Código | Tipo | Descrição |
|--------|------|-----------|
| 400 | invalid_rating | Rating deve ser entre 1 e 5 |
| 400 | missing_fields | Campos obrigatórios ausentes |
| 403 | certificate_disabled | Curso não oferece certificado |
| 403 | not_enrolled | Usuário não matriculado no curso |
| 403 | access_denied | Usuário sem acesso ao recurso |
| 404 | not_found | Recurso não encontrado |
| 500 | server_error | Erro interno do servidor |

## Considerações de Segurança

1. **Verificação de Matrícula**: Apenas estudantes matriculados podem acessar recursos
2. **Certificados Únicos**: Código de verificação único e não falsificável
3. **Reviews Verificadas**: Apenas estudantes que completaram podem avaliar
4. **Rate Limiting**: Proteção contra spam nas discussões
5. **Validação de Dados**: Sanitização de inputs em avaliações e discussões

## Melhores Práticas

1. **Certificados**: Validar progresso antes de gerar certificado
2. **Avaliações**: Moderar reviews para evitar spam
3. **Configurações**: Salvar automaticamente para melhor UX
4. **Discussões**: Implementar sistema de moderação
5. **Performance**: Cache de estatísticas de avaliações

Este sistema avançado complementa perfeitamente o EAD, oferecendo funcionalidades que aumentam o engagement e validam o aprendizado dos estudantes.
