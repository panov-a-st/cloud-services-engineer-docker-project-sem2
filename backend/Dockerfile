# Сборка
FROM golang:1.24.4-alpine3.22 AS builder

# Меняем workdir
WORKDIR /app

# Копируем и устанавливаем зависимости
COPY go.mod go.sum ./
RUN go mod download

# Копируем другие файлы, выполняем сборку
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags="-w -s" -o /go/bin/app ./cmd/api

# Используем alpine для запуска
FROM alpine:3.22.0

# Добавим сертификаты
RUN apk --no-cache add ca-certificates

# Создадим пользователя
RUN adduser -D -g '' user

WORKDIR /app

COPY --from=builder /go/bin/app /app/momo-store
RUN chmod 500 /app/momo-store && chown -R user:user /app

# Переключимся на пользователя, используем порт 8081
USER user
EXPOSE 8081

# Проверка приложения
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=5 \
CMD wget -q --spider http://localhost:8081/health || exit 1

# Запуск
CMD ["/app/momo-store"]