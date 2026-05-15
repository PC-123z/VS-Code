# syntax=docker/dockerfile:1.7
# Builds Paperclip from GitHub source. Hermes is invoked via
# `docker exec hermes-gateway hermes` — no local install needed.

ARG PAPERCLIP_REF=master

FROM node:lts-trixie-slim AS base
ARG USER_UID=1000
ARG USER_GID=1000
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
       ca-certificates gosu curl git wget ripgrep python3 \
  && rm -rf /var/lib/apt/lists/* \
  && corepack enable \
  && usermod  -u $USER_UID --non-unique node \
  && groupmod -g $USER_GID --non-unique node \
  && usermod  -g $USER_GID -d /paperclip node

FROM base AS source
ARG PAPERCLIP_REF
WORKDIR /src
RUN git clone --depth=1 --branch "$PAPERCLIP_REF" \
      https://github.com/paperclipai/paperclip.git .

FROM source AS deps
WORKDIR /src
RUN pnpm install --frozen-lockfile

FROM deps AS build
WORKDIR /src
RUN pnpm --filter @paperclipai/ui         build \
 && pnpm --filter @paperclipai/plugin-sdk build \
 && pnpm --filter @paperclipai/server     build \
 && test -f server/dist/index.js || (echo "ERROR: server build missing" && exit 1)

FROM base AS production
ARG USER_UID=1000
ARG USER_GID=1000
WORKDIR /app

COPY --chown=node:node --from=build /src /app

# Install global agent CLIs + docker CLI (for `docker exec hermes-gateway hermes`)
RUN npm install --global --omit=dev \
      @anthropic-ai/claude-code@latest \
      @openai/codex@latest \
      opencode-ai \
  && apt-get update \
  && apt-get install -y --no-install-recommends openssh-client jq docker.io \
  && rm -rf /var/lib/apt/lists/* \
  && mkdir -p /paperclip \
  && chown node:node /paperclip

COPY --from=build /src/scripts/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

# Bake in hermes config so the adapter can auto-detect the Ollama model
COPY hermes-config.yaml /paperclip/.hermes/config.yaml
RUN chown node:node /paperclip/.hermes/config.yaml

ENV NODE_ENV=production \
    HOME=/paperclip \
    HOST=0.0.0.0 \
    PORT=3100 \
    SERVE_UI=true \
    PAPERCLIP_HOME=/paperclip \
    PAPERCLIP_INSTANCE_ID=default \
    USER_UID=${USER_UID} \
    USER_GID=${USER_GID} \
    PAPERCLIP_CONFIG=/paperclip/instances/default/config.json \
    PAPERCLIP_DEPLOYMENT_MODE=authenticated \
    PAPERCLIP_DEPLOYMENT_EXPOSURE=private \
    OPENCODE_ALLOW_ALL_MODELS=true

VOLUME ["/paperclip"]
EXPOSE 3100

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["node", "--import", "./server/node_modules/tsx/dist/loader.mjs", "server/dist/index.js"]
