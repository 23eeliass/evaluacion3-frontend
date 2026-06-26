# Innovatech - Frontend Despacho (Plataforma DevOps)

Este repositorio almacena el código fuente, la configuración de empaquetamiento y la lógica de automatización para el despliegue continuo (CI/CD) del módulo Frontend del sistema **Innovatech** (`front_despacho`).

## 🛠️ Stack Tecnológico Utilizado
* **Core:** React / Vite (Producción compilada en directorio `dist`)
* **Contenerización:** Docker (Estrategia Avanzada Multi-stage Build)
* **Web Server:** Nginx Alpine optimizado para arquitecturas SPA
* **Orquestador:** Amazon ECS (Elastic Container Service) sobre AWS Fargate (Serverless)
* **Pipeline CI/CD:** GitHub Actions Declarativo

---

## 📦 Estrategia de Contenerización (Dockerfile)
La construcción de la imagen de producción se realiza mediante un proceso multi-etapa aislada. Esto permite compilar la aplicación utilizando Node.js, y transferir únicamente los artefactos finales listos hacia un servidor Nginx liviano, reduciendo la superficie de ataque y el peso del contenedor:

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
# Inyección inline de configuración para el correcto ruteo del Frontend
RUN echo 'server { \
    listen 80; \
    location / { \
        root /usr/share/nginx/html; \
        index index.html index.htm; \
        try_files $uri $uri/ /index.html; \
    } \
}' > /etc/nginx/conf.d/default.conf

COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]





🚀 Pipeline de Integración y Despliegue Continuo (CI/CD)
El archivo de flujo de trabajo .github/workflows/deploy.yml está diseñado para reaccionar ante cambios en la rama principal (push), automatizando las siguientes fases de ingeniería:

Checkout: Descarga e inspección del código en el runner virtual de GitHub.

Configure AWS Credentials: Inyección segura de credenciales dinámicas de AWS Academy.

Login to Amazon ECR: Autenticación en el registro elástico de contenedores privado de AWS.

Build & Push: Compilación de la imagen Docker bajo la etiqueta latest y subida inmediata al repositorio ECR.

Deploy ECS Task: Creación de una nueva revisión del Task Definition en Amazon ECS, provocando que el servicio realice un rolling update controlado de la tarea sobre AWS Fargate de manera automatizada.



🔒 Gestión de Variables y Secretos Seguros
El pipeline requiere que se configuren de manera obligatoria las siguientes variables cifradas dentro del repositorio (Settings > Secrets and variables > Actions), mitigando riesgos de fugas de credenciales en los archivos de texto:

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

AWS_SESSION_TOKEN



🌐 Validación de Infraestructura y Acceso
Networking: El contenedor se ejecuta exponiendo el puerto estándar web HTTP (80).

Acceso del Cliente: Al operar en un entorno elástico dinámico (AWS Academy), la aplicación se encuentra expuesta para verificaciones y pruebas funcionales directamente mediante la IP Pública provista por la tarea activa de Fargate dentro del clúster elástico corporativo.