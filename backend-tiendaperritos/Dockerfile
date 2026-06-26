# ==========================================
# Etapa 1: Builder (Construcción y dependencias)
# ==========================================
FROM node:18-alpine AS builder

WORKDIR /app

# Copiamos solo los archivos de dependencias primero para aprovechar el caché de capas
COPY package*.json ./

# Instalamos las dependencias
RUN npm install --production

# Copiamos el resto del código
COPY . .

# ==========================================
# Etapa 2: Producción (Imagen final limpia y segura)
# ==========================================
FROM node:18-alpine

WORKDIR /app

# Definimos el entorno de producción
ENV NODE_ENV=production

# Usamos el usuario 'node' (no root) que viene por defecto en la imagen alpine
# por motivos de seguridad (Principio de Mínimo Privilegio)
USER node

# Copiamos las dependencias instaladas y el código desde la etapa 'builder'
# y le asignamos los permisos al usuario 'node'
COPY --from=builder --chown=node:node /app/node_modules ./node_modules
COPY --from=builder --chown=node:node /app/package*.json ./
COPY --from=builder --chown=node:node /app/server.js ./

# Exponemos el puerto de la API
EXPOSE 3001

# Ejecutamos directamente con node (mejor práctica en prod que usar npm start)
CMD ["node", "server.js"]
