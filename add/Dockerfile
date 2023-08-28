FROM golang:latest

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
ENTRYPOINT go run main.go
EXPOSE 3000