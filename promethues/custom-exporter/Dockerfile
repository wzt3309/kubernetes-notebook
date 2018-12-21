
#build stage
FROM golang:alpine AS builder
WORKDIR /go/src/app
COPY . .
RUN go install -v ./...

#final stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY --from=builder /go/bin/app /app
ENTRYPOINT ./app
LABEL Name=prometheus-exporter-demo Version=0.0.1
EXPOSE 8080
