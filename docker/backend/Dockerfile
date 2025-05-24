# Builder stage
FROM rust:1.75-slim as builder

WORKDIR /usr/src/app
RUN apt-get update && apt-get install -y pkg-config libssl-dev

# Create blank project
RUN cargo new investutil-back
WORKDIR /usr/src/app/investutil-back

# Copy manifests
COPY Cargo.toml Cargo.lock ./

# Cache dependencies
RUN cargo build --release
RUN rm src/*.rs

# Copy source code
COPY . .

# Build for release
RUN rm ./target/release/deps/investutil_back*
RUN cargo build --release

# Final stage
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y libssl-dev ca-certificates && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy the binary from builder
COPY --from=builder /usr/src/app/investutil-back/target/release/investutil-back .

# Copy migrations
COPY --from=builder /usr/src/app/investutil-back/migrations ./migrations

# Create config directory
RUN mkdir -p /app/config

# Create a non-root user
RUN useradd -m -u 1001 appuser
RUN chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8080

# Command to run
CMD ["./investutil-back"] 