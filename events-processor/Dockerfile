FROM rust:1.85 AS rust-build

RUN git clone --tags https://github.com/getlago/lago-expression/
WORKDIR /lago-expression/expression-go
RUN git checkout v0.2.0 && cargo build --release

FROM golang:1.25 AS go-build

WORKDIR /app

COPY . /app/

RUN go mod download
COPY --from=rust-build /lago-expression/target/release/libexpression_go.so /usr/lib/libexpression_go.so
RUN go build -o event_processors .

FROM debian:13-slim

RUN apt-get update && apt-get upgrade -y && apt-get install -y ca-certificates

WORKDIR /app
COPY --from=rust-build /lago-expression/target/release/libexpression_go.so /usr/lib/libexpression_go.so
COPY --from=go-build /app/event_processors /app/event_processors

# Modified by Rezliant AI, 2026-09-17 16:31:00 GMT, add non-root user for container security
RUN useradd --uid 1000 --user-group --system --create-home --no-log-init appuser && \
    chown -R appuser:appuser /app
USER appuser

# Original Code
# ENTRYPOINT ["./event_processors"]
ENTRYPOINT ["./event_processors"]