# Stage 1: Build dependencies
FROM node:21-bookworm-slim as builder

WORKDIR .

COPY yarn.lock package.json ./
COPY . .  

# Default to yarn v1.22.22
ENV YARN_VERSION 1.22.22
RUN yarn policies set-version $YARN_VERSION

RUN yarn install && yarn build:photos && yarn build:accounts && yarn build:auth && yarn build:cast

# Stage 2: Final image (without node_modules)
FROM caddy:latest as caddy
WORKDIR /app

# Copy only necessary files from the build stage
COPY --from=builder /apps/photos/out /app/photos
COPY --from=builder /apps/auth/out /app/auth
COPY --from=builder /apps/accounts/out /app/accounts
COPY --from=builder /apps/cast/out /app/cast

# Expose ports
ARG PHOTOS=3001
EXPOSE ${PHOTOS}

ARG ACCOUNTS=3002
EXPOSE ${ACCOUNTS}

ARG AUTH=3003
EXPOSE ${AUTH}

ARG CAST=3004
EXPOSE ${CAST}

COPY <<EOF /etc/caddy/Caddyfile 
:${PHOTOS} {
  root * /app/photos
    file_server
}
:${ACCOUNTS} {
  root * /app/accounts
    file_server
}
:${AUTH} {
  root * /app/auth
    file_server
}
:${CAST} {
  root * /app/cast
    file_server
}
EOF

RUN echo "serving on ports ${PHOTOS} ${ACCOUNTS} ${AUTH} ${CAST}"
CMD ["caddy", "start", "--config", "/etc/caddy/Caddyfile"]
