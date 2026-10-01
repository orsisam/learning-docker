FROM golang:alpine

RUN mkdir /app

RUN addgroup -S orsigroup
RUN adduser -S -D -h /app orsi orsigroup
RUN chown -R orsi:orsigroup /app

USER orsi

COPY main.go /app

EXPOSE 8080

CMD ["go", "run", "/app/main.go"]

