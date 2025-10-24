🧭 Proyecto de Microservicios — Sistema Backend Distribuido
🚀 Descripción General

Este proyecto forma parte del desarrollo de un sistema backend basado en microservicios, en el cual cada servicio es independiente, mantiene su propia base de datos y se comunica con otros por medio de APIs REST.
El objetivo es construir una infraestructura modular, escalable y mantenible utilizando Django REST Framework, PostgreSQL, Redis y Docker.

👉 Propósito del sistema:
Gestión de autenticación y registro de usuarios dentro de un entorno distribuido de microservicios.

🧩 Microservicios Implementados
Servicio	Descripción	Puerto
Auth-Service	Maneja registro, login y autenticación JWT con PostgreSQL y Redis.	8000
(Próximos servicios)	Student-Service, Incident-Service, Reports-Service	—
⚙️ Tecnologías utilizadas
Componente	Función
Python 3.11	Lenguaje principal del backend
Django REST Framework (DRF)	Creación de APIs REST
PostgreSQL	Base de datos relacional
Redis	Cacheo, sesiones y optimización
Docker / Docker Compose	Contenedorización y orquestación
JWT (SimpleJWT)	Autenticación segura entre servicios
🧱 Estructura del Proyecto
proyecto-microservicios/
│
├── auth-service/               # Microservicio de autenticación (Día 2)
│   ├── users/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── README.md               # Documentación específica del microservicio
│
├── docker-compose.yml          # Orquestador de todos los servicios
├── .env.example                # Variables de entorno (opcional)
└── README.md                   # Documentación general (este archivo)

🐳 Ejecución con Docker

1️⃣ Construir e iniciar los servicios:

docker-compose up --build


2️⃣ Verificar los contenedores activos:

docker ps


3️⃣ Acceder al servicio de autenticación:

http://localhost:8000

🔑 Endpoints principales (Auth-Service)
Método	Ruta	Descripción
POST	/api/register/	Crear usuario
POST	/api/token/	Obtener tokens JWT
POST	/api/token/refresh/	Renovar token
GET	/api/me/ (opcional)	Ver datos del usuario autenticado
🧪 Pruebas con Postman

1️⃣ Registro de usuario

POST /api/register/
{
  "email": "usuario@demo.com",
  "password": "123456"
}


2️⃣ Login y obtención de tokens

POST /api/token/
{
  "email": "usuario@demo.com",
  "password": "123456"
}


3️⃣ Renovación de token

POST /api/token/refresh/
{
  "refresh": "TU_REFRESH_TOKEN_AQUI"
}