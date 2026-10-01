FROM golang:alpine AS builder

WORKDIR /app/
COPY main.go .
RUN go build -o main main.go


FROM alpine:latest
WORKDIR /app/
COPY --from=builder /app/main ./
EXPOSE 8080
CMD ["./main"]
