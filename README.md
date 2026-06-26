# 🐶 Proyecto Tienda de Perritos - Innovatech Chile (POC)

Este repositorio contiene la **Prueba de Concepto (POC)** para la migración de la infraestructura de la empresa *Innovatech Chile* hacia la nube de AWS. El proyecto se enfoca en la transición de una arquitectura monolítica hacia una solución **contenedorizada y automatizada** mediante prácticas de DevOps modernas.

Este trabajo forma parte de la **Evaluación Parcial N°2** de la asignatura **ISY1101 – Introducción a Herramientas DevOps**.

---

## 🏗️ 1. Arquitectura de Infraestructura (AWS)

La solución se despliega sobre una topología de red robusta diseñada bajo el principio de **Defensa en Profundidad**:

* **VPC:** Segmento `10.0.0.0/16` para escalabilidad.
* **Subredes:**
    * **Capa de Presentación (Pública):** Aloja el Frontend (`ec2-web-frontend-innovatech`) accesible mediante un Internet Gateway (IGW).
    * **Capa de Lógica (Privada):** Aloja el Backend API (`ec2-backend-innovatech`) sin acceso directo desde internet.
    * **Capa de Datos (Privada):** Aloja el motor de Base de Datos (`ec2-db-innovatech`) en una subred aislada.
* **Seguridad (Security Groups):**
    * `sg-web`: Permite tráfico HTTP (80) desde cualquier origen.
    * `sg-app`: Permite tráfico TCP (3001) **únicamente** desde el grupo de seguridad del Frontend.
    * `sg-datos`: Permite tráfico TCP (3306) **únicamente** desde el grupo de seguridad del Backend.
* **Administración:** Se ha implementado una política de **Cero Acceso SSH** (Puerto 22 cerrado), utilizando **AWS Systems Manager (Session Manager)** para la gestión de instancias mediante túneles seguros.

---

## 🐋 2. Contenedorización (Docker)

Siguiendo los indicadores de evaluación (**IE1, IE2 e IE3**), se han diseñado contenedores optimizados para producción:

### ⚙️ Backend (Multi-stage & Security)
Se utiliza un **Dockerfile multi-stage** para separar el entorno de compilación del de ejecución, reduciendo el tamaño de la imagen final y eliminando dependencias de desarrollo.
* **Usuario No-Root:** La aplicación corre bajo el usuario `node` (no root) para mitigar riesgos de seguridad.
* **Limpieza de capas:** Se agrupan comandos `RUN` para minimizar el número de capas.

### 💾 Persistencia de Datos
La base de datos utiliza un **Named Volume** (`mysql_data`) mapeado a `/var/lib/mysql`.
* **Justificación técnica:** Se eligió un volumen nombrado frente a un *bind mount* para garantizar la portabilidad entre entornos y asegurar que la información crítica no se pierda durante los ciclos de despliegue automatizado de CI/CD.

---

## 🚀 3. Pipeline de Integración y Despliegue Continuo (CI/CD)

La automatización se gestiona mediante **GitHub Actions** (Indicador **IE4**), con un flujo activado por la rama `deploy`:

1.  **Build & Push:** Se construye la imagen Docker y se publica en **Docker Hub**.
2.  **Deployment via SSM:** GitHub Actions se autentica en AWS (usando credenciales temporales de Academy) y envía un comando a través de **AWS Systems Manager** a la instancia EC2 correspondiente.
3.  **Update:** La instancia EC2 descarga la nueva imagen, detiene el contenedor antiguo y levanta la versión actualizada de forma transparente.

---

## 🛠️ 4. Ejecución en Entorno Local

Para replicar el stack completo en un entorno de desarrollo, se utiliza **Docker Compose**:

### Requisitos
* Docker Engine
* Docker Compose V2

### Pasos
1.  Clonar el repositorio.
2.  Configurar las variables de entorno si es necesario.
3.  Ejecutar el comando:
    ```bash
    docker-compose up -d --build
    ```

### Endpoints Disponibles
* **Frontend:** `http://localhost:80`
* **Backend Health:** `http://localhost:3001/api/health`
* **CRUD Productos:** `http://localhost:3001/api/productos`

---

## 📝 5. Documentación de Entrega

* **Autores:** Miguel Medina & Ian Pereira.
* **Repositorios:**
    * Frontend: https://github.com/MdeMedina/frontend-tiendaperritos.git
    * Backend: https://github.com/MdeMedina/backend-tiendaperritos.git
    * Base de Datos: https://github.com/MdeMedina/db-tiendaperritos.git
* **Estado del Despliegue:** Operativo en instancias EC2 con conectividad integrada Front → Back → DB.

---
*Este proyecto cumple con todos los criterios de la rúbrica de evaluación parcial N°2 de Duoc UC (2025).*
