FROM golang:latest AS builder

RUN apt-get update && apt-get install -y ca-certificates openssl

ARG cert_location=/usr/local/share/ca-certificates

# Get certificate from "github.com"
RUN openssl s_client -showcerts -connect github.com:443 </dev/null 2>/dev/null|openssl x509 -outform PEM > ${cert_location}/github.crt
# Get certificate from "proxy.golang.org"
RUN openssl s_client -showcerts -connect proxy.golang.org:443 </dev/null 2>/dev/null|openssl x509 -outform PEM >  ${cert_location}/proxy.golang.crt
# Update certificates
RUN update-ca-certificates

# create appuser.
RUN adduser --disabled-password -gid 0 elf

WORKDIR /go/src/add
COPY go.mod go.sum ./
RUN go mod download && \
    go mod verify
# copy the source code as the last step
COPY . .

# build binary
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -a -installsuffix cgo -o /go/bin/api ./


# build a small image
FROM alpine:3.17.3
LABEL language="golang"
# import the user and group files from the builder
COPY --from=builder /etc/passwd /etc/passwd
# copy the static executable
COPY --from=builder --chown=elf:1000 /go/bin/api /api
# use a non-root user
USER elf
# run app
ENTRYPOINT ["./api"]
# ENTRYPOINT go run main.go
EXPOSE 8080