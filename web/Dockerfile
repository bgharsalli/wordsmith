# BUILD stage
FROM golang:alpine AS builder
WORKDIR /usr/local/app

COPY dispatcher.go .

# Compilation classique pour Linux amd64
RUN GOOS=linux GOARCH=amd64 go build -o dispatcher dispatcher.go

# RUN stage
FROM alpine:latest
WORKDIR /usr/local/app

COPY --from=builder /usr/local/app/dispatcher ./
COPY static ./static/

EXPOSE 80
CMD ["/usr/local/app/dispatcher"]
