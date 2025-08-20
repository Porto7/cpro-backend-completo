# Documentação API - Sistema de Upload de Arquivos

## Visão Geral
Sistema completo para upload de vídeos e imagens com processamento automático, geração de thumbnails, validação de formato e armazenamento seguro.

## Endpoints Disponíveis

### 1. Upload de Vídeo
**POST** `/api/upload/video`

**Autenticação:** Opcional (recomendada para produção)

**Descrição:** Faz upload de vídeo com extração automática de thumbnail e informações de metadata.

**Content-Type:** `multipart/form-data`

**Parâmetros:**
- **Form Data:**
  - `video` (file, obrigatório): Arquivo de vídeo
- **Query Parameters:**
  - `maxSize` (number, opcional): Tamanho máximo em MB (padrão: 500MB)

**Formatos Suportados:**
- MP4 (video/mp4)
- AVI (video/x-msvideo)
- MOV (video/quicktime)
- WMV (video/x-ms-wmv)
- WEBM (video/webm)
- MPEG (video/mpeg)

**Exemplo de Requisição:**
```javascript
const formData = new FormData();
formData.append('video', videoFile);

const response = await fetch('/api/upload/video?maxSize=1000', {
  method: 'POST',
  body: formData
});
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Vídeo uploadado com sucesso",
  "video": {
    "filename": "uuid-timestamp.mp4",
    "originalName": "meu-video.mp4",
    "size": 52428800,
    "mimeType": "video/mp4",
    "url": "/uploads/videos/uuid-timestamp.mp4",
    "duration": 1800,
    "width": 1920,
    "height": 1080,
    "format": "mov,mp4,m4a,3gp,3g2,mj2"
  },
  "thumbnail": {
    "filename": "thumb_uuid-timestamp.jpg",
    "url": "/uploads/thumbnails/thumb_uuid-timestamp.jpg"
  }
}
```

**Informações Extraídas do Vídeo:**
- `duration`: Duração em segundos
- `width/height`: Resolução do vídeo
- `format`: Formato detectado pelo FFmpeg
- `fps`: Taxa de quadros por segundo
- `bitrate`: Taxa de bits do arquivo
- `hasAudio`: Se possui trilha de áudio

**Erros Possíveis:**
- `400` - Arquivo muito grande
- `400` - Formato não suportado
- `400` - Nenhum arquivo enviado
- `500` - Erro no processamento do vídeo

### 2. Upload de Imagem
**POST** `/api/upload/image`

**Autenticação:** Opcional

**Descrição:** Faz upload de imagem com geração automática de thumbnail.

**Content-Type:** `multipart/form-data`

**Parâmetros:**
- **Form Data:**
  - `image` ou `file` (file, obrigatório): Arquivo de imagem
  - `type` (string, opcional): Tipo da imagem (`product`, `logo`, `banner`)
- **Query Parameters:**
  - `maxSize` (number, opcional): Tamanho máximo em MB (padrão: 10MB)

**Formatos Suportados:**
- JPEG (image/jpeg)
- PNG (image/png)
- GIF (image/gif)
- WEBP (image/webp)
- BMP (image/bmp)

**Exemplo de Requisição:**
```javascript
const formData = new FormData();
formData.append('file', imageFile);
formData.append('type', 'product');

const response = await fetch('/api/upload/image?maxSize=5', {
  method: 'POST',
  body: formData
});
```

**Resposta de Sucesso (201):**
```json
{
  "success": true,
  "message": "Imagem enviada com sucesso",
  "data": {
    "url": "https://api.example.com/uploads/images/uuid-timestamp.jpg",
    "thumbnail": "https://api.example.com/uploads/images/thumb-uuid-timestamp.jpg",
    "type": "product",
    "filename": "uuid-timestamp.jpg",
    "size": 2048576,
    "mimetype": "image/jpeg",
    "originalName": "produto-foto.jpg"
  }
}
```

**Geração de Thumbnail:**
- Thumbnail redimensionado para 300x300px (mantendo proporção)
- Qualidade JPEG 80%
- Fallback para imagem original se processamento falhar

### 3. Upload Múltiplo de Imagens
**POST** `/api/upload/multiple-images`

**Autenticação:** Opcional

**Descrição:** Faz upload de múltiplas imagens simultaneamente.

**Content-Type:** `multipart/form-data`

**Parâmetros:**
- **Form Data:**
  - `images` (files, obrigatório): Array de arquivos de imagem
- **Query Parameters:**
  - `maxSize` (number, opcional): Tamanho máximo por arquivo em MB (padrão: 10MB)
  - `maxFiles` (number, opcional): Número máximo de arquivos (padrão: 5)

**Exemplo de Requisição:**
```javascript
const formData = new FormData();
imageFiles.forEach(file => {
  formData.append('images', file);
});

const response = await fetch('/api/upload/multiple-images?maxFiles=10', {
  method: 'POST',
  body: formData
});
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "3 imagem(ns) enviada(s) com sucesso",
  "data": [
    {
      "filename": "uuid-1-timestamp.jpg",
      "originalName": "foto1.jpg",
      "size": 1024000,
      "mimetype": "image/jpeg",
      "url": "/uploads/images/uuid-1-timestamp.jpg",
      "path": "/app/uploads/images/uuid-1-timestamp.jpg"
    },
    {
      "filename": "uuid-2-timestamp.png",
      "originalName": "foto2.png",
      "size": 2048000,
      "mimetype": "image/png",
      "url": "/uploads/images/uuid-2-timestamp.png",
      "path": "/app/uploads/images/uuid-2-timestamp.png"
    }
  ]
}
```

### 4. Obter Informações do Vídeo
**GET** `/api/upload/info/:filename`

**Autenticação:** Opcional

**Descrição:** Retorna informações detalhadas de um vídeo uploadado.

**Parâmetros de Rota:**
- `filename` (string): Nome do arquivo de vídeo

**Resposta de Sucesso (200):**
```json
{
  "filename": "uuid-timestamp.mp4",
  "size": 52428800,
  "created": "2024-01-15T10:30:00Z",
  "modified": "2024-01-15T10:30:00Z",
  "duration": 1800.5,
  "bitrate": 2000000,
  "width": 1920,
  "height": 1080,
  "fps": "30/1",
  "hasAudio": true,
  "format": "mov,mp4,m4a,3gp,3g2,mj2"
}
```

### 5. Deletar Vídeo
**DELETE** `/api/upload/video/:filename`

**Autenticação:** Recomendada

**Descrição:** Remove vídeo e thumbnail associado do servidor.

**Parâmetros de Rota:**
- `filename` (string): Nome do arquivo de vídeo

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Vídeo e thumbnail deletados com sucesso"
}
```

### 6. Deletar Imagem
**DELETE** `/api/upload/image/:filename`

**Autenticação:** Recomendada

**Descrição:** Remove imagem do servidor.

**Parâmetros de Rota:**
- `filename` (string): Nome do arquivo de imagem

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Imagem deletada com sucesso"
}
```

**Erros Possíveis:**
- `404` - Arquivo não encontrado

## Estrutura de Arquivos

### Organização de Diretórios
```
uploads/
├── videos/                 # Vídeos de aulas
│   ├── uuid-timestamp.mp4
│   └── uuid-timestamp.mov
├── images/                 # Imagens de produtos
│   ├── uuid-timestamp.jpg
│   ├── thumb-uuid-timestamp.jpg
│   └── uuid-timestamp.png
└── thumbnails/             # Thumbnails de vídeos
    ├── thumb_uuid-timestamp.jpg
    └── thumb_uuid-timestamp.png
```

### Nomenclatura de Arquivos
- **Padrão:** `{uuid}-{timestamp}.{extensão}`
- **Thumbnails de vídeo:** `thumb_{nome-original}.jpg`
- **Thumbnails de imagem:** `thumb-{nome-original}.{extensão}`

## Processamento de Vídeo

### Extração de Thumbnail
```javascript
// Thumbnail extraída aos 10% da duração do vídeo
ffmpeg(videoPath)
  .screenshots({
    timestamps: ['10%'],
    filename: 'thumb_' + originalName + '.jpg',
    folder: thumbnailsDir
  });
```

### Metadados Extraídos
- **Duração total** em segundos
- **Resolução** (largura x altura)
- **Taxa de quadros** (FPS)
- **Taxa de bits** do arquivo
- **Presença de áudio**
- **Formato detectado**

## Processamento de Imagem

### Geração de Thumbnail
```javascript
// Redimensionamento com Sharp (se disponível)
sharp(imagePath)
  .resize(300, 300, { 
    fit: 'inside',
    withoutEnlargement: true 
  })
  .jpeg({ quality: 80 })
  .toFile(thumbnailPath);
```

### Fallback sem Sharp
- Se a biblioteca Sharp não estiver disponível
- Sistema usa a imagem original como thumbnail
- Mantém funcionalidade completa

## Validações e Limites

### Limites de Arquivo
- **Vídeos:** 500MB (padrão, configurável)
- **Imagens:** 10MB (padrão, configurável)
- **Upload múltiplo:** 5 arquivos (padrão, configurável)

### Validações de Formato
```javascript
const allowedVideoFormats = [
  'video/mp4', 'video/mpeg', 'video/quicktime',
  'video/x-msvideo', 'video/x-ms-wmv', 'video/webm'
];

const allowedImageFormats = [
  'image/jpeg', 'image/jpg', 'image/png',
  'image/gif', 'image/webp', 'image/bmp'
];
```

### Segurança
- Validação rigorosa de MIME types
- Nomes de arquivo únicos com UUID
- Limpeza automática em caso de erro
- Validação de extensão de arquivo

## URLs de Acesso

### Estrutura de URLs
- **Vídeos:** `{base_url}/uploads/videos/{filename}`
- **Imagens:** `{base_url}/uploads/images/{filename}`
- **Thumbnails de vídeo:** `{base_url}/uploads/thumbnails/{filename}`

### Servir Arquivos Estáticos
```javascript
// Configuração no Express
app.use('/uploads', express.static('uploads'));
```

## Exemplos de Uso

### Upload de Vídeo Completo
```javascript
const uploadVideo = async (videoFile) => {
  const formData = new FormData();
  formData.append('video', videoFile);
  
  try {
    const response = await fetch('/api/upload/video', {
      method: 'POST',
      body: formData
    });
    
    const result = await response.json();
    
    if (result.success) {
      console.log('Vídeo URL:', result.video.url);
      console.log('Thumbnail URL:', result.thumbnail.url);
      console.log('Duração:', result.video.duration, 'segundos');
    }
    
    return result;
  } catch (error) {
    console.error('Erro no upload:', error);
    throw error;
  }
};
```

### Upload com Progress
```javascript
const uploadWithProgress = async (file, onProgress) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    const formData = new FormData();
    formData.append('video', file);
    
    xhr.upload.addEventListener('progress', (e) => {
      if (e.lengthComputable) {
        const percentComplete = (e.loaded / e.total) * 100;
        onProgress(percentComplete);
      }
    });
    
    xhr.addEventListener('load', () => {
      if (xhr.status === 200) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error('Upload falhou'));
      }
    });
    
    xhr.open('POST', '/api/upload/video');
    xhr.send(formData);
  });
};

// Uso
await uploadWithProgress(videoFile, (progress) => {
  console.log(`Upload: ${progress.toFixed(1)}%`);
});
```

### Upload Múltiplo de Imagens
```javascript
const uploadMultipleImages = async (imageFiles) => {
  const formData = new FormData();
  
  imageFiles.forEach(file => {
    formData.append('images', file);
  });
  
  const response = await fetch('/api/upload/multiple-images?maxFiles=10', {
    method: 'POST',
    body: formData
  });
  
  return await response.json();
};
```

### Validação no Frontend
```javascript
const validateFile = (file, type = 'video') => {
  const limits = {
    video: {
      maxSize: 500 * 1024 * 1024, // 500MB
      formats: ['video/mp4', 'video/mpeg', 'video/quicktime']
    },
    image: {
      maxSize: 10 * 1024 * 1024, // 10MB
      formats: ['image/jpeg', 'image/png', 'image/gif']
    }
  };
  
  const config = limits[type];
  
  if (file.size > config.maxSize) {
    throw new Error(`Arquivo muito grande. Máximo: ${config.maxSize / (1024*1024)}MB`);
  }
  
  if (!config.formats.includes(file.type)) {
    throw new Error(`Formato não suportado: ${file.type}`);
  }
  
  return true;
};
```

## Configuração e Dependências

### Dependências Necessárias
```json
{
  "multer": "^1.4.5",
  "ffmpeg": "^0.0.4",
  "sharp": "^0.32.0"
}
```

### Configuração do FFmpeg
- **Necessário para:** Extração de thumbnails e metadados de vídeo
- **Instalação:** Deve estar disponível no PATH do sistema
- **Alternativa:** Usar FFmpeg em container Docker

### Configuração do Sharp
- **Necessário para:** Processamento e redimensionamento de imagens
- **Opcional:** Sistema funciona sem Sharp (usa imagem original)
- **Performance:** Recomendado para melhor performance

## Observações Técnicas

1. **Armazenamento**: Arquivos salvos localmente (considerar cloud storage para produção)
2. **Cleanup**: Limpeza automática de arquivos em caso de erro
3. **Atomic Operations**: Upload e processamento tratados atomicamente
4. **Error Handling**: Tratamento robusto de erros em todas as etapas
5. **Logging**: Logs detalhados para debugging e auditoria
6. **Extensibilidade**: Arquitetura preparada para novos tipos de arquivo
