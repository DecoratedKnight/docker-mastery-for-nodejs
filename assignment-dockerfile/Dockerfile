# fill in the blanks to dockerize this node app
FROM node:10.15-alpine

EXPOSE 3000

RUN apk add --no-cache tini

WORKDIR /node

COPY package*.json ./

RUN chown -R node:node .

USER node

RUN npm i && npm cache clean --force

COPY --chown=node:node . .

ENTRYPOINT ["/sbin/tini", "--"]

CMD ["node", "app.js"]
