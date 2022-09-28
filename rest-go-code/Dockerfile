# Builder
FROM golang:1.16-alpine as builder
ENV APP_HOME="/rest-go"
WORKDIR $APP_HOME

COPY . .
RUN apk update && \
    apk upgrade && \
    apk --update add git make gcc musl-dev 
RUN export GO111MODULE=on && \
    export GOPROXY=direct && \
    export GOSUMDB=off

RUN go mod download && go build -o rest-go

# Distribution
FROM alpine:latest
WORKDIR /rest-go
RUN apk update && apk upgrade && \
    apk --update --no-cache add tzdata

COPY --from=builder /rest-go/rest-go .
COPY . .
RUN chmod +x ./docker-entrypoint.sh
EXPOSE 8080
ENTRYPOINT ["./docker-entrypoint.sh"]