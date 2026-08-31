# Serves a locally built docs site together with a working search,
# mirroring production docs.cfengine.com: nginx serves the static site and
# proxies /docs/search to the search server (see nginx.conf), the same way
# Apache does in production. The search server itself is the server.js
# from NorthernTechHQ/infra.
#
# Expected mounts (see serve_site in build-locally.sh):
#   /site               built _site (nginx root)
#   /docs/<branch>      the same _site, where server.js looks for
#                       <branch>/assets/searchIndex under DOCS_PATH=/docs
#   /search/server.js   search server from the infra repo
FROM nginx:alpine

# flexsearch must match the version the search index was built with
# (nt-docs scripts/search/index)
ARG FLEXSEARCH_VERSION=0.7.31
RUN apk add --no-cache nodejs npm \
    && npm install --global "flexsearch@${FLEXSEARCH_VERSION}"

COPY nginx.conf /etc/nginx/conf.d/default.conf

# server.js resolves flexsearch from the global node_modules via NODE_PATH
CMD ["sh", "-c", "NODE_PATH=$(npm root -g) node /search/server.js & exec nginx -g 'daemon off;'"]
