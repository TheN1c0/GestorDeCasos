FROM node:20-alpine

WORKDIR /app

# Instala dependencias primero (mejor cache)
COPY package*.json ./
RUN npm ci

# Copia el resto
COPY . .

# Prisma (client)
RUN npx prisma generate

EXPOSE 3000

# En docker-compose sobreescribimos el command
CMD ["npm", "run", "start:dev"]
