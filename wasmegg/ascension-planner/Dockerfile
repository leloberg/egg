# Builds the Ascension Planner and serves it with nginx at /ascension-planner/.
# The build context must be the repository root, because the planner depends on
# the shared `lib` and `ui` workspaces:
#   docker build -f wasmegg/ascension-planner/Dockerfile .

FROM node:22-bookworm-slim AS build
RUN corepack enable
WORKDIR /repo
COPY . .
RUN pnpm install --frozen-lockfile --filter ascension-planner...
# Generate the protobuf bindings (what `make` in lib/ does).
RUN cd lib \
 && pnpm pbjs -p protobuf -t static-module -w es6 --force-number -o proto/index.js ../protobuf/*.proto \
 && node proto/patch-missioninfo.js \
 && pnpm pbts -o proto/index.d.ts proto/index.js \
 && pnpm pbjs -t static-module -w es6 --force-number -o sandbox/schema.js sandbox/schema.proto \
 && pnpm pbts -o sandbox/schema.d.ts sandbox/schema.js
RUN cd wasmegg/ascension-planner && pnpm fastbuild

FROM nginx:1.27-alpine
COPY wasmegg/ascension-planner/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /repo/wasmegg/ascension-planner/dist /usr/share/nginx/html/ascension-planner
EXPOSE 80
