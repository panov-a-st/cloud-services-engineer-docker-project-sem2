# Сборка
FROM node:18-alpine AS builder

# Создаем пользователя
RUN addgroup -S user && adduser -S user -G user

# Используем /app
WORKDIR /app

# Копируем базовые файлы зависимостей, включая lock и устанавливаем зависимости
COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund

# Копируем остальное и собираем
COPY . .
ENV NODE_OPTIONS=--openssl-legacy-provider
RUN npm run build && chown -R user:user /app/dist

# Запустим на образе alpine
FROM nginx:1.26-alpine

# Создаем пользователя
RUN addgroup -S user && adduser -S user -G user

# Используем следующую директорию, докер должен сам создать ее
WORKDIR /var/www/html/momo-store

# Копирование приложения и конфига nginx. Docker создаст сам необходимый каталог
COPY --from=builder --chown=user:user /app/dist/ /var/www/html/momo-store
COPY --chown=user:user nginx.conf /etc/nginx/conf.d/default.conf

# Установим права доступа
RUN chown -R user:user /var/cache/nginx \
    && chown -R user:user /var/log/nginx \
    && chown -R user:user /etc/nginx/conf.d \
    && chmod -R 755 /var/www/html/momo-store \
    && touch /var/run/nginx.pid && chown user:user /var/run/nginx.pid

# Используем нового пользователя
USER user

# Порт 80
EXPOSE 80

# Проверка приложения и запуск nginx
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=5 \
CMD curl -f http://localhost/momo-store/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
