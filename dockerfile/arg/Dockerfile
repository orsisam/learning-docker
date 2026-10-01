FROM golang:alpine

ARG app=main

RUN mkdir app
COPY main.go app
RUN mv app/main.go app/${app}.go

EXPOSE 8080

ENV app=${app}
CMD ["/bin/sh", "-c", "go run app/${app}.go"]

