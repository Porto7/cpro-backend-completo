# Documentação - Sistema de Gamificação (routes/gamification.js)

## Visão Geral
Sistema completo de gamificação que adiciona elementos de jogo ao aprendizado, incluindo pontos, conquistas, rankings, desafios diários e sistema de níveis para aumentar o engajamento dos estudantes.

## Características Principais
- **Sistema de Pontos**: Pontuação baseada em atividades de aprendizado
- **Níveis**: Progressão através de níveis baseada em pontos
- **Conquistas**: Sistema de badges e achievements
- **Ranking**: Leaderboard competitivo
- **Desafios Diários**: Missões diárias para engajamento
- **Streak**: Sistema de dias consecutivos
- **Atividades Trackadas**: Registro detalhado de ações

## Rotas Implementadas

### 1. Perfil de Gamificação

#### `GET /api/gamification/profile`
**Descrição**: Obtém perfil completo de gamificação do usuário
**Acesso**: Autenticado

**Response Success**:
```json
{
  "user": {
    "id": "user-123",
    "name": "João Silva",
    "avatar": "https://exemplo.com/avatar.jpg",
    "memberSince": "2024-01-01T10:00:00Z"
  },
  "gamification": {
    "level": 5,
    "totalPoints": 2350,
    "pointsToNextLevel": 150,
    "pointsBreakdown": {
      "courseCompletion": 1000,
      "studyTime": 830,
      "lessonCompletion": 375,
      "streakDays": 70,
      "firstLogin": 50,
      "profileComplete": 25
    },
    "achievements": {
      "total": 15,
      "unlocked": 8,
      "list": [
        {
          "id": "first_course",
          "title": "Primeiro Passo",
          "description": "Complete seu primeiro curso",
          "icon": "🎯",
          "unlocked": true,
          "points": 100
        },
        {
          "id": "course_master",
          "title": "Mestre dos Cursos",
          "description": "Complete 5 cursos",
          "icon": "🏆",
          "unlocked": true,
          "points": 500
        },
        {
          "id": "study_warrior",
          "title": "Guerreiro dos Estudos",
          "description": "Estude por 50 horas",
          "icon": "⚔️",
          "unlocked": true,
          "points": 200
        },
        {
          "id": "quick_learner",
          "title": "Aprendiz Rápido",
          "description": "Complete um curso em menos de 24 horas",
          "icon": "⚡",
          "unlocked": false,
          "points": 150
        },
        {
          "id": "night_owl",
          "title": "Coruja Noturna",
          "description": "Estude depois das 22h",
          "icon": "🦉",
          "unlocked": false,
          "points": 75
        }
      ]
    },
    "stats": {
      "coursesCompleted": 10,
      "coursesInProgress": 3,
      "totalHoursStudied": 83.5,
      "currentStreak": 7,
      "longestStreak": 15
    },
    "badges": [
      {
        "id": "first_course",
        "title": "Primeiro Passo",
        "icon": "🎯"
      },
      {
        "id": "course_master",
        "title": "Mestre dos Cursos",
        "icon": "🏆"
      }
    ]
  }
}
```

**Sistema de Pontos**:
- **Course Completion**: 100 pontos por curso completo
- **Study Time**: 10 pontos por hora estudada
- **Lesson Completion**: 25 pontos por aula completada
- **Streak Days**: 10 pontos por dia consecutivo
- **First Login**: 50 pontos de boas-vindas
- **Profile Complete**: 25 pontos por perfil completo

**Sistema de Níveis**:
- **Nível 1**: 0-499 pontos
- **Nível 2**: 500-999 pontos  
- **Nível 3**: 1000-1499 pontos
- **E assim por diante** (500 pontos por nível)

### 2. Ranking/Leaderboard

#### `GET /api/gamification/leaderboard`
**Descrição**: Obtém ranking de pontos entre usuários
**Acesso**: Autenticado

**Query Parameters**:
- `period`: Período do ranking (all, month, week) - padrão: all
- `limit`: Número de usuários no ranking - padrão: 50

**Response Success**:
```json
{
  "leaderboard": [
    {
      "id": "user-456",
      "name": "Maria Santos",
      "avatar": "https://exemplo.com/maria.jpg",
      "level": 8,
      "totalPoints": 3750,
      "completedCourses": 15,
      "studyHours": 125.3,
      "position": 1
    },
    {
      "id": "user-789",
      "name": "Pedro Costa",
      "avatar": "https://exemplo.com/pedro.jpg",
      "level": 6,
      "totalPoints": 2890,
      "completedCourses": 12,
      "studyHours": 98.7,
      "position": 2
    },
    {
      "id": "user-123",
      "name": "João Silva",
      "avatar": "https://exemplo.com/joao.jpg",
      "level": 5,
      "totalPoints": 2350,
      "completedCourses": 10,
      "studyHours": 83.5,
      "position": 3
    }
  ],
  "currentUser": {
    "position": 3,
    "data": {
      "id": "user-123",
      "name": "João Silva",
      "level": 5,
      "totalPoints": 2350,
      "position": 3
    }
  },
  "stats": {
    "totalUsers": 50,
    "period": "all",
    "lastUpdated": "2024-01-01T16:00:00Z"
  }
}
```

### 3. Conquistas

#### `POST /api/gamification/achievement/:achievementId/claim`
**Descrição**: Resgata conquista desbloqueada
**Acesso**: Autenticado

**Response Success**:
```json
{
  "success": true,
  "message": "Conquista resgatada com sucesso!",
  "achievement": {
    "id": "course_master",
    "title": "Mestre dos Cursos",
    "points": 500,
    "claimedAt": "2024-01-01T16:00:00Z"
  },
  "pointsAwarded": 500
}
```

**Conquistas Disponíveis**:

| ID | Título | Descrição | Pontos | Ícone |
|----|--------|-----------|--------|-------|
| first_course | Primeiro Passo | Complete seu primeiro curso | 100 | 🎯 |
| course_master | Mestre dos Cursos | Complete 5 cursos | 500 | 🏆 |
| study_warrior | Guerreiro dos Estudos | Estude por 50 horas | 200 | ⚔️ |
| quick_learner | Aprendiz Rápido | Complete um curso em menos de 24h | 150 | ⚡ |
| night_owl | Coruja Noturna | Estude depois das 22h | 75 | 🦉 |
| early_bird | Madrugador | Estude antes das 6h | 75 | 🐦 |
| weekend_warrior | Guerreiro de Fim de Semana | Estude no fim de semana | 50 | 🗓️ |
| note_taker | Anotador | Crie 10 anotações | 100 | 📝 |
| perfect_week | Semana Perfeita | Estude todos os dias da semana | 200 | ✨ |
| speed_runner | Velocista | Complete 3 aulas em 1 dia | 125 | 🏃 |

### 4. Desafio Diário

#### `GET /api/gamification/daily-challenge`
**Descrição**: Obtém desafio diário atual
**Acesso**: Autenticado

**Response Success**:
```json
{
  "challenge": {
    "id": "complete_lesson",
    "title": "Complete uma aula",
    "description": "Termine qualquer aula hoje",
    "points": 25,
    "icon": "📚",
    "type": "lesson_completion",
    "date": "2024-01-01",
    "completed": false,
    "progress": 0
  },
  "streak": {
    "current": 7,
    "longest": 15
  }
}
```

**Tipos de Desafios**:

| Tipo | Título | Descrição | Pontos | Ícone |
|------|--------|-----------|--------|-------|
| complete_lesson | Complete uma aula | Termine qualquer aula hoje | 25 | 📚 |
| study_30_minutes | Estude por 30 minutos | Dedique pelo menos 30 min aos estudos | 50 | ⏰ |
| watch_video | Assista um vídeo completo | Assista qualquer vídeo do início ao fim | 35 | 🎥 |
| take_notes | Faça anotações | Crie pelo menos uma anotação | 40 | 📝 |
| quiz_perfect | Quiz Perfeito | Acerte 100% em qualquer quiz | 60 | 🎯 |
| early_study | Estudo Matinal | Estude antes das 8h | 45 | 🌅 |

### 5. Registro de Atividades

#### `POST /api/gamification/activity`
**Descrição**: Registra atividade para pontuação
**Acesso**: Autenticado

```javascript
// Payload de exemplo
{
  "type": "lesson_completed",
  "courseId": "course-123",
  "lessonId": "lesson-456",
  "duration": 1800, // segundos
  "metadata": {
    "quiz_score": 95,
    "notes_taken": true,
    "video_watched": true
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "activity": {
    "type": "lesson_completed",
    "points": 25,
    "timestamp": "2024-01-01T16:00:00Z"
  },
  "message": "+25 pontos ganhos!"
}
```

**Tipos de Atividades e Pontos**:

| Tipo | Pontos | Descrição |
|------|--------|-----------|
| lesson_completed | 25 | Aula completada |
| video_watched | 15 | Vídeo assistido completamente |
| note_created | 10 | Anotação criada |
| course_completed | 100 | Curso completado |
| quiz_completed | 20 | Quiz completado |
| daily_goal_reached | 50 | Meta diária alcançada |
| forum_post | 5 | Post no fórum |
| assignment_submitted | 30 | Tarefa entregue |

## Funcionalidades Avançadas

### 1. Sistema de Streak
```javascript
// Cálculo de dias consecutivos
{
  "streak": {
    "current": 7, // Dias consecutivos atuais
    "longest": 15, // Maior sequência já alcançada
    "today_completed": true,
    "next_milestone": 10, // Próxima meta de streak
    "milestone_reward": 100 // Pontos da próxima meta
  }
}
```

### 2. Badges Dinâmicos
```javascript
// Sistema de badges baseado em comportamento
{
  "badges": [
    {
      "id": "early_bird_week",
      "title": "Madrugador da Semana",
      "description": "Estudou de manhã todos os dias",
      "rarity": "rare",
      "earned_at": "2024-01-01T06:00:00Z"
    }
  ]
}
```

### 3. Metas Personalizadas
```javascript
// Metas definidas pelo usuário
{
  "personalGoals": [
    {
      "id": "goal-123",
      "title": "Concluir 3 cursos este mês",
      "type": "course_completion",
      "target": 3,
      "current": 1,
      "deadline": "2024-01-31",
      "reward_points": 300
    }
  ]
}
```

## Integração Frontend

### React Component Example
```jsx
import React, { useState, useEffect } from 'react';

const GamificationDashboard = ({ userToken }) => {
  const [profile, setProfile] = useState(null);
  const [leaderboard, setLeaderboard] = useState([]);
  const [dailyChallenge, setDailyChallenge] = useState(null);
  
  useEffect(() => {
    fetchGamificationData();
  }, []);
  
  const fetchGamificationData = async () => {
    try {
      const [profileRes, leaderboardRes, challengeRes] = await Promise.all([
        fetch('/api/gamification/profile', {
          headers: { 'Authorization': `Bearer ${userToken}` }
        }),
        fetch('/api/gamification/leaderboard?limit=10', {
          headers: { 'Authorization': `Bearer ${userToken}` }
        }),
        fetch('/api/gamification/daily-challenge', {
          headers: { 'Authorization': `Bearer ${userToken}` }
        })
      ]);
      
      const [profileData, leaderboardData, challengeData] = await Promise.all([
        profileRes.json(),
        leaderboardRes.json(),
        challengeRes.json()
      ]);
      
      setProfile(profileData);
      setLeaderboard(leaderboardData.leaderboard);
      setDailyChallenge(challengeData.challenge);
    } catch (error) {
      console.error('Erro ao carregar dados de gamificação:', error);
    }
  };
  
  const trackActivity = async (type, metadata = {}) => {
    try {
      await fetch('/api/gamification/activity', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${userToken}`
        },
        body: JSON.stringify({
          type,
          ...metadata
        })
      });
      
      // Recarregar dados
      fetchGamificationData();
    } catch (error) {
      console.error('Erro ao registrar atividade:', error);
    }
  };
  
  if (!profile) return <div>Carregando...</div>;
  
  return (
    <div className="gamification-dashboard">
      {/* Perfil do Usuário */}
      <div className="user-profile">
        <img src={profile.user.avatar} alt={profile.user.name} />
        <h2>{profile.user.name}</h2>
        <div className="level-badge">Nível {profile.gamification.level}</div>
        <div className="points">{profile.gamification.totalPoints} pontos</div>
        <div className="progress-bar">
          <div 
            className="progress" 
            style={{
              width: `${(profile.gamification.totalPoints % 500) / 5}%`
            }}
          />
        </div>
        <p>{profile.gamification.pointsToNextLevel} pontos para o próximo nível</p>
      </div>
      
      {/* Desafio Diário */}
      <div className="daily-challenge">
        <h3>Desafio de Hoje</h3>
        <div className={`challenge ${dailyChallenge.completed ? 'completed' : ''}`}>
          <span className="icon">{dailyChallenge.icon}</span>
          <div>
            <h4>{dailyChallenge.title}</h4>
            <p>{dailyChallenge.description}</p>
            <span className="points">+{dailyChallenge.points} pontos</span>
          </div>
        </div>
      </div>
      
      {/* Conquistas */}
      <div className="achievements">
        <h3>Conquistas</h3>
        <div className="achievements-grid">
          {profile.gamification.achievements.list.map(achievement => (
            <div 
              key={achievement.id} 
              className={`achievement ${achievement.unlocked ? 'unlocked' : 'locked'}`}
            >
              <span className="icon">{achievement.icon}</span>
              <h4>{achievement.title}</h4>
              <p>{achievement.description}</p>
              <span className="points">+{achievement.points}</span>
            </div>
          ))}
        </div>
      </div>
      
      {/* Ranking */}
      <div className="leaderboard">
        <h3>Top 10 Ranking</h3>
        <div className="leaderboard-list">
          {leaderboard.map(user => (
            <div key={user.id} className="leaderboard-item">
              <span className="position">#{user.position}</span>
              <img src={user.avatar} alt={user.name} />
              <span className="name">{user.name}</span>
              <span className="level">Nv. {user.level}</span>
              <span className="points">{user.totalPoints} pts</span>
            </div>
          ))}
        </div>
      </div>
      
      {/* Estatísticas */}
      <div className="stats">
        <h3>Suas Estatísticas</h3>
        <div className="stats-grid">
          <div className="stat">
            <h4>{profile.gamification.stats.coursesCompleted}</h4>
            <p>Cursos Completos</p>
          </div>
          <div className="stat">
            <h4>{profile.gamification.stats.totalHoursStudied}h</h4>
            <p>Horas Estudadas</p>
          </div>
          <div className="stat">
            <h4>{profile.gamification.stats.currentStreak}</h4>
            <p>Dias Consecutivos</p>
          </div>
        </div>
      </div>
    </div>
  );
};
```

### Activity Tracking Hook
```javascript
// Hook para tracking automático
import { useEffect } from 'react';

export const useGamificationTracking = (userToken) => {
  const trackActivity = async (type, metadata = {}) => {
    try {
      await fetch('/api/gamification/activity', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${userToken}`
        },
        body: JSON.stringify({
          type,
          timestamp: new Date().toISOString(),
          ...metadata
        })
      });
    } catch (error) {
      console.error('Erro ao registrar atividade:', error);
    }
  };
  
  return { trackActivity };
};

// Uso em componentes de curso
const LessonComponent = ({ lesson, userToken }) => {
  const { trackActivity } = useGamificationTracking(userToken);
  
  const handleLessonComplete = async () => {
    await markLessonAsComplete(lesson.id);
    
    // Tracking automático
    trackActivity('lesson_completed', {
      courseId: lesson.courseId,
      lessonId: lesson.id,
      duration: lesson.watchTime
    });
  };
  
  return (
    <div>
      {/* Conteúdo da aula */}
      <button onClick={handleLessonComplete}>
        Marcar como Concluída
      </button>
    </div>
  );
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 404 | Usuário ou conquista não encontrada |
| 401 | Token de autenticação inválido |
| 400 | Dados da atividade inválidos |
| 500 | Erro interno do servidor |

## Considerações de Performance

1. **Caching**: Cache de rankings e perfis
2. **Batch Processing**: Processamento em lote de atividades
3. **Indexes**: Índices otimizados para consultas
4. **Agregações**: Cálculos pré-computados
5. **Real-time**: Updates em tempo real via WebSocket

## Roadmap Futuro

### Funcionalidades Planejadas
- [ ] Tabela dedicada de pontos e atividades
- [ ] Sistema de moedas virtuais
- [ ] Loja de recompensas
- [ ] Torneios e competições
- [ ] Grupos e times
- [ ] Conquistas temporárias/sazonais
- [ ] Sistema de mentoria
- [ ] Integração com redes sociais

### Melhorias de Performance
- [ ] Cache Redis para rankings
- [ ] Worker jobs para cálculos
- [ ] WebSocket para atualizações
- [ ] Analytics detalhadas

Este sistema de gamificação oferece uma experiência envolvente que motiva os estudantes através de elementos de jogo, aumentando significativamente o engajamento e a retenção no aprendizado.
