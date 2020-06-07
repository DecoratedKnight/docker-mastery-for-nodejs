FROM node:8-slim as base
ENV NODE_ENV=production
EXPOSE 80
WORKDIR /app
COPY package*.json ./
RUN npm ci \
    && npm cache clean --force
ENV PATH /app/node_modules/.bin:$PATH
CMD ["node", "server.js"]


FROM base as dev
ENV NODE_ENV=development

# NOTE: these apt dependencies are only needed
# for testing. they shouldn't be in production
RUN apt-get update -qq && apt-get install -qy \ 
    ca-certificates \
    bzip2 \
    curl \
    libfontconfig \
    --no-install-recommends
RUN npm install --only-dev \
    && npm cache clean --force
CMD [ "nodemon", "server.js" ]


FROM dev as test
COPY . .
# RUN npm audit //ここで診断にひっかかると次のステップに行かない様にできる。いったん面倒なのでコメントアウト

FROM test as pre-prod
RUN rm -rf ./tests && rm -rf ./node_modules

FROM base as prod
COPY --from=pre-prod /app /app
HEALTHCHECK CMD curl http://127.0.0.1/ || exit 1