FROM golang:1.24 AS base
WORKDIR /app
COPY go.mod .
RUN go mod download
COPY . .
RUN go build -o test .

FROM gcr.io/distroless/base:latest
COPY --from=base /app/test .
EXPOSE 8080
CMD [ "./test" ]
