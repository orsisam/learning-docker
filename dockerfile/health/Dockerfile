FROM golang:alpine

RUN apk --no-cache add curl
RUN mkdir app
COPY main.go app

EXPOSE 8080

HEALTHCHECK --interval=5s --start-period=5s CMD ["/bin/sh", "-c", "curl -f http://localhost:8080/health"]
CMD ["/bin/sh", "-c", "go run app/main.go"]
