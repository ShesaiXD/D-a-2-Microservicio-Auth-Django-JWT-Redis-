🧭 Microservicio Auth — Django + DRF + JWT + PostgreSQL + Redis
🎯 Objetivo General

Construir un microservicio de autenticación independiente que gestione usuarios, login y tokens JWT.
El servicio corre en su propio contenedor Docker, conectado a PostgreSQL (base de datos) y Redis (cache y sesiones).

🧩 Tecnologías Utilizadas

🐍 Python 3.11

🧱 Django 5.0

⚙️ Django REST Framework (DRF)

🔐 JWT (JSON Web Tokens)

🐘 PostgreSQL

🚀 Redis

🐳 Docker / Docker Compose

🌐 CORS Headers

⚙️ Estructura del Proyecto
auth-service/
├── auth_service/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│
├── users/
│   ├── models.py
│   ├── views.py
│   ├── serializers.py
│   ├── urls.py
│
├── Dockerfile
├── requirements.txt
├── manage.py

🐳 Configuración con Docker
Dockerfile
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "auth_service.wsgi:application", "--bind", "0.0.0.0:8000"]

docker-compose.yml
auth:
  build: ./auth-service
  container_name: auth_service
  restart: always
  environment:
    - DEBUG=1
    - DB_HOST=postgres
    - DB_NAME=main_db
    - DB_USER=devuser
    - DB_PASS=devpass
    - REDIS_HOST=redis
    - REDIS_PORT=6379
  depends_on:
    - postgres
    - redis
  ports:
    - "8000:8000"

📦 Dependencias (requirements.txt)
django==5.0
djangorestframework==3.15
djangorestframework-simplejwt==5.3
psycopg2-binary
redis
django-cors-headers

⚙️ Configuración principal (settings.py)

Registro de apps:

INSTALLED_APPS = [
    'rest_framework',
    'corsheaders',
    'users',
]


Variables de entorno para PostgreSQL:

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME'),
        'USER': os.getenv('DB_USER'),
        'PASSWORD': os.getenv('DB_PASS'),
        'HOST': os.getenv('DB_HOST'),
        'PORT': '5432',
    }
}


Cache con Redis:

CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.redis.RedisCache",
        "LOCATION": f"redis://{os.getenv('REDIS_HOST')}:{os.getenv('REDIS_PORT')}/1",
    }
}


Autenticación JWT:

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
}


Modelo de usuario personalizado:

AUTH_USER_MODEL = 'users.User'

👤 Modelo de Usuario (users/models.py)
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager
from django.db import models

class UserManager(BaseUserManager):
    def create_user(self, email, password=None):
        if not email:
            raise ValueError("Email obligatorio")
        user = self.model(email=self.normalize_email(email))
        user.set_password(password)
        user.save(using=self._db)
        return user

class User(AbstractBaseUser):
    email = models.EmailField(unique=True)
    is_active = models.BooleanField(default=True)
    is_admin = models.BooleanField(default=False)

    USERNAME_FIELD = 'email'
    objects = UserManager()

    def __str__(self):
        return self.email

🚀 Endpoints Principales
Método	Endpoint	Descripción
POST	/api/register/	Crear nuevo usuario
POST	/api/token/	Obtener access y refresh token
POST	/api/token/refresh/	Renovar el token
GET	/api/me/ (opcional)	Ver información del usuario autenticado
🔑 Ejemplo de Uso en Postman

1️⃣ Registro de usuario
POST /api/register/

{
  "email": "user@example.com",
  "password": "12345678"
}


2️⃣ Obtener token
POST /api/token/

{
  "email": "user@example.com",
  "password": "12345678"
}


Respuesta:

{
  "access": "eyJhbGciOiJIUzI1...",
  "refresh": "eyJhbGciOiJIUzI1NiIsInR..."
}


3️⃣ Renovar token
POST /api/token/refresh/

{
  "refresh": "eyJhbGciOiJIUzI1NiIsInR..."
}


4️⃣ Ver perfil autenticado
GET /api/me/ con encabezado
Authorization: Bearer <access_token>

🧪 Verificación Interna

Conexión a la base de datos y Redis:

docker exec -it auth_service python manage.py shell

📸 Evidencias

📷 Capturas en Postman mostrando:

Registro exitoso

Login y obtención de tokens

Refresh token

Consulta a /api/me/

🧠 Evaluación Teórica — Día 2
Nº	Pregunta	Puntos
1	Propósito de un microservicio de autenticación separado	2
2	Diferencia entre autenticación por sesiones y JWT	2
3	Estructura del token JWT (header, payload, signature)	2
4	Ventaja de usar Redis (ejemplo práctico)	2
5	Función del Dockerfile	2
6	Por qué usar archivo .env	2
7	Librería usada para JWT (rest_framework_simplejwt)	2
8	Comandos para migraciones (makemigrations / migrate)	2
9	Flujo completo de login con JWT	2
10	Buenas prácticas de seguridad (tokens, CORS, HTTPS)	2

Total: 20 pts

📊 Escala de Evaluación
Puntaje	Nivel	Descripción
18 – 20	⭐ Avanzado	Configuración y seguridad impecables
15 – 17	✅ Competente	JWT y Redis correctamente implementados
12 – 14	⚠️ Básico	Entiende teoría, pero necesita guía
< 12	❌ Insuficiente	No domina JWT, Redis o entorno
🧑‍💻 Autor

Mark Alexis Rodríguez Pérez
📍 Proyecto Día 2 — Backend Auth Microservice
🚀 Desarrollo con Django REST Framework, JWT, PostgreSQL y Redis.
