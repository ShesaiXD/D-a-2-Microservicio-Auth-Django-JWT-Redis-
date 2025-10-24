# 🧭 Auth Service — Microservicio de Autenticación (Django + DRF + JWT + PostgreSQL + Redis)

## 🎯 Objetivo general
Construir un microservicio de autenticación completamente independiente, que maneje usuarios, login y tokens **JWT**, corriendo en su propio contenedor **Docker** y conectado a **PostgreSQL** y **Redis**.

---

## 🧩 Conceptos clave

- Autenticación basada en **JWT (JSON Web Tokens)**  
- Estructura de un servicio **Django aislado**  
- Configuración de **variables de entorno y dependencias**  
- **Cacheo y sesiones** con Redis  
- Comunicación segura entre servicios vía API  

---

## ⚙️ Tecnologías utilizadas

| Componente | Tecnología |
|-------------|-------------|
| Lenguaje | Python 3.11 |
| Framework backend | Django 5.0 |
| API REST | Django REST Framework 3.15 |
| Autenticación | SimpleJWT 5.3 |
| Base de datos | PostgreSQL |
| Cache y sesiones | Redis |
| Servidor WSGI | Gunicorn |
| Contenedores | Docker / Docker Compose |

---

## 🧱 Estructura del proyecto

auth-service/
│
├── auth_service/ # Configuración principal del proyecto Django
│ ├── settings.py
│ ├── urls.py
│ ├── wsgi.py
│
├── users/ # Aplicación encargada de los usuarios
│ ├── models.py # Modelo personalizado User
│ ├── serializers.py
│ ├── views.py
│ ├── urls.py
│
├── manage.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── README.md


---

## ⚙️ Configuración y despliegue con Docker

### 1️⃣ Construir e iniciar contenedores

```bash
docker-compose up -d --build

2️⃣ Verificar que estén activos
docker ps


Deberías ver:

CONTAINER ID   NAME           STATUS          PORTS
xxxxx          auth_service   Up 2 minutes    0.0.0.0:8000->8000/tcp
xxxxx          postgres       Up 2 minutes    5432/tcp
xxxxx          redis          Up 2 minutes    6379/tcp

3️⃣ Aplicar migraciones de Django
docker exec -it auth_service python manage.py migrate

4️⃣ Ver logs (opcional)
docker logs auth_service --follow

⚙️ Configuración de entorno (variables)

Variables definidas en docker-compose.yml o .env:

DEBUG=1
DB_HOST=postgres
DB_NAME=main_db
DB_USER=devuser
DB_PASS=devpass
REDIS_HOST=redis
REDIS_PORT=6379

🧩 Configuración del proyecto Django
settings.py

Apps instaladas:

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'users',
]


Base de datos:

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': os.environ.get('DB_HOST'),
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASS'),
        'PORT': 5432,
    }
}


Cache (Redis):

CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': f"redis://{os.environ.get('REDIS_HOST')}:{os.environ.get('REDIS_PORT')}/1",
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}


Middleware CORS:

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]


Autenticación JWT:

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    )
}


Usuario personalizado:

AUTH_USER_MODEL = 'users.User'

👤 Modelo de usuario personalizado

Archivo: users/models.py

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

🔐 Endpoints principales
Método	Endpoint	Descripción	Requiere JWT
POST	/api/register/	Registra un nuevo usuario	❌
POST	/api/token/	Genera tokens (login)	❌
POST	/api/token/refresh/	Renueva el token de acceso	❌
GET	/api/me/	Devuelve la información del usuario autenticado	✅
🧪 Ejemplos de uso (Postman o curl)
🧍 Registro
POST http://localhost:8000/api/register/
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "TestPass123!"
}


Respuesta

{
  "id": 1,
  "email": "user@example.com"
}

🔑 Login (obtener tokens)
POST http://localhost:8000/api/token/
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "TestPass123!"
}


Respuesta

{
  "refresh": "eyJhbGciOiJIUzI1NiIs...",
  "access": "eyJhbGciOiJIUzI1NiIs..."
}

♻️ Refresh token
POST http://localhost:8000/api/token/refresh/
Content-Type: application/json

{
  "refresh": "eyJhbGciOiJIUzI1NiIs..."
}


Respuesta

{
  "access": "eyJhbGciOiJIUzI1NiIs..."
}

👤 Usuario autenticado
GET http://localhost:8000/api/me/
Authorization: Bearer <access_token>


Respuesta

{
  "id": 1,
  "email": "user@example.com"
}
