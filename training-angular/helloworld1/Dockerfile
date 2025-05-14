# Etapa 1: Construcción de la aplicación Angular
FROM node:lts as build-step

# Crear el directorio de la app
RUN mkdir -p /app
WORKDIR /app

# Copiar solo los archivos de dependencias
COPY package.json package-lock.json /app/

# Instalar dependencias y Angular CLI globalmente
RUN npm install
RUN npm install -g @angular/cli

# Copiar el resto del proyecto
COPY . /app

# Asegurarse de que el binario ng tenga permisos de ejecución
RUN chmod +x ./node_modules/.bin/ng

# Ejecutar la construcción de la app Angular
RUN npm run build --prod

# Etapa 2: Servir con Nginx
FROM nginx:1.17.1-alpine

# Copiar los archivos construidos en la etapa anterior
COPY --from=build-step /app/dist/helloworld1/browser /usr/share/nginx/html
