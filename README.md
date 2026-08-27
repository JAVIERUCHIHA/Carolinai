# 🏛️ Scraper de Jurisprudencia Chilena - Render.com. 

Scraper automatizado para extraer sentencias judiciales desde [juris.pjud.cl](https://juris.pjud.cl) y almacenarlas en Supabase con embeddings vectoriales para búsquedas semánticas.

## 🚀 Características

- ✅ **Extracción automática** de sentencias de múltiples tribunales
- ✅ **Detección de duplicados** basada en ROL único
- ✅ **Embeddings vectoriales** con OpenAI para búsquedas semánticas
- ✅ **Almacenamiento en Supabase** con pgvector
- ✅ **Optimizado para Render.com** como Background Worker
- ✅ **Logging detallado** con timestamps
- ✅ **Configuración flexible** via variables de entorno

## 🏗️ Arquitectura

```
juris.pjud.cl → Puppeteer → Procesamiento → OpenAI Embeddings → Supabase + pgvector
```

## 📋 Requisitos

- Node.js >= 20.0.0
- Cuenta en [Render.com](https://render.com)
- Cuenta en [Supabase](https://supabase.com)
- API Key de [OpenAI](https://openai.com) (opcional)

## 🛠️ Instalación Local

1. **Clonar el repositorio**
```bash
git clone https://github.com/tu-usuario/jurisprudencia-scraper.git
cd jurisprudencia-scraper
```

2. **Instalar dependencias**
```bash
npm install
```

3. **Configurar variables de entorno**
```bash
cp env.example .env
# Editar .env con tus credenciales
```

4. **Ejecutar localmente**
```bash
npm start
```

## 🌐 Deploy en Render.com

### 1. Preparar el repositorio

Asegúrate de que tu repositorio contenga:
- ✅ `package.json` con script `start`
- ✅ `main.js` como punto de entrada
- ✅ `.gitignore` excluyendo `node_modules` y `.env`

### 2. Crear servicio en Render

1. Ve a [dashboard.render.com](https://dashboard.render.com)
2. Clic en **"New +"** → **"Background Worker"**
3. Conecta tu cuenta de GitHub
4. Selecciona el repositorio del scraper

### 3. Configurar el servicio

**Build Command:**
```bash
npm install
```

**Start Command:**
```bash
npm start
```

**Environment:** Node

### 4. Variables de entorno

En la sección **Environment** agrega:

```bash
# Supabase
SUPABASE_URL=https://tu-proyecto.supabase.co
SUPABASE_ANON_KEY=tu-clave-anonima

# OpenAI (opcional)
OPENAI_API_KEY=sk-tu-clave-openai
ENABLE_EMBEDDINGS=true

# Configuración del scraper
TRIBUNAL=Corte_Suprema
START_DATE=01/01/2024
END_DATE=31/12/2024
MAX_SENTENCIAS=100
DELAY_BETWEEN_REQUESTS=2000
DEBUG=false
```

### 5. Deploy

Clic en **"Create Background Worker"** y espera a que se complete el deploy.

## 📊 Configuración de Supabase

### 1. Crear tabla

Ejecuta este SQL en tu proyecto Supabase:

```sql
-- Habilitar extensión pgvector
CREATE EXTENSION IF NOT EXISTS vector;

-- Tabla de jurisprudencia
CREATE TABLE jurisprudencia_cs (
    id BIGSERIAL PRIMARY KEY,
    rol VARCHAR(255) UNIQUE NOT NULL,
    fecha DATE,
    tribunal VARCHAR(255),
    caratulado TEXT,
    texto_completo TEXT,
    considerandos TEXT,
    resolucion TEXT,
    enlace VARCHAR(500),
    hash_contenido VARCHAR(64),
    embeddings vector(1536),
    fecha_ingreso TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Índice para búsquedas vectoriales
CREATE INDEX ON jurisprudencia_cs USING ivfflat (embeddings vector_cosine_ops);

-- Índices para búsquedas tradicionales
CREATE INDEX idx_jurisprudencia_rol ON jurisprudencia_cs(rol);
CREATE INDEX idx_jurisprudencia_fecha ON jurisprudencia_cs(fecha);
CREATE INDEX idx_jurisprudencia_tribunal ON jurisprudencia_cs(tribunal);
```

### 2. Configurar RLS (Row Level Security)

```sql
-- Habilitar RLS
ALTER TABLE jurisprudencia_cs ENABLE ROW LEVEL SECURITY;

-- Política para lectura pública
CREATE POLICY "Permitir lectura pública" ON jurisprudencia_cs
    FOR SELECT USING (true);

-- Política para inserción desde el scraper
CREATE POLICY "Permitir inserción desde scraper" ON jurisprudencia_cs
    FOR INSERT WITH CHECK (true);
```

## ⚙️ Configuración

### Variables de Entorno

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `SUPABASE_URL` | URL de tu proyecto Supabase | `https://xxx.supabase.co` |
| `SUPABASE_ANON_KEY` | Clave anónima de Supabase | `eyJ...` |
| `OPENAI_API_KEY` | API Key de OpenAI | `sk-...` |
| `ENABLE_EMBEDDINGS` | Habilitar embeddings | `true` |
| `TRIBUNAL` | Tribunal a scrapear | `Corte_Suprema` |
| `START_DATE` | Fecha inicio (DD/MM/YYYY) | `01/01/2024` |
| `END_DATE` | Fecha fin (DD/MM/YYYY) | `31/12/2024` |
| `MAX_SENTENCIAS` | Máximo de sentencias | `100` |
| `SEARCH_TERM` | Término de búsqueda | `derecho laboral` |
| `DELAY_BETWEEN_REQUESTS` | Delay entre requests (ms) | `2000` |
| `DEBUG` | Modo debug | `false` |

### Tribunales Disponibles

- `Corte_Suprema` - Corte Suprema
- `Corte_de_Apelaciones` - Cortes de Apelaciones
- `Penales` - Juzgados Penales
- `Familia` - Juzgados de Familia
- `Laboral` - Juzgados Laborales
- `Civil` - Juzgados Civiles

## 📈 Monitoreo

### Logs en Render

Los logs se pueden ver en tiempo real en la pestaña **"Logs"** de tu servicio en Render.

### Estadísticas del Scraper

El scraper muestra estadísticas al finalizar:

```
=== ESTADÍSTICAS FINALES ===
Duración total: 125 segundos
Total procesadas: 50
Exitosas: 45
Duplicadas: 3
Errores: 2
============================
```

## 🔄 Programación

Para ejecutar el scraper periódicamente:

1. **Cron Job en Render:**
   - Crea un **Cron Job** en lugar de Background Worker
   - Configura el schedule (ej: `0 2 * * *` para diario a las 2 AM)

2. **Webhook desde otro servicio:**
   - Crea un **Web Service** en lugar de Background Worker
   - Configura un endpoint que ejecute el scraper

## 🛡️ Seguridad

- ✅ Variables de entorno para credenciales
- ✅ Detección de duplicados
- ✅ Rate limiting con delays
- ✅ Logging sin información sensible
- ✅ Manejo de errores robusto

## 🐛 Troubleshooting

### Error: "No se encontró el botón de búsqueda"
- Verificar que el sitio juris.pjud.cl esté accesible
- Revisar si cambiaron los selectores CSS

### Error: "Supabase no inicializado"
- Verificar variables `SUPABASE_URL` y `SUPABASE_ANON_KEY`
- Confirmar que la tabla `jurisprudencia_cs` existe

### Error: "OpenAI no inicializado"
- Verificar variable `OPENAI_API_KEY`
- Confirmar que `ENABLE_EMBEDDINGS=true`

### Error: "Puppeteer timeout"
- Aumentar `DELAY_BETWEEN_REQUESTS`
- Verificar conectividad de red

## 📝 Licencia

MIT License - ver [LICENSE](LICENSE) para detalles.

## 🤝 Contribuir

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📞 Soporte

- 📧 Email: contacto@redjudicial.cl
- 🌐 Web: [www.redjudicial.cl](https://www.redjudicial.cl)
- 📱 GitHub Issues: [Crear issue](https://github.com/tu-usuario/jurisprudencia-scraper/issues)

---

**Desarrollado con ❤️ por RedJudicial** Forzar nuevo deploy
