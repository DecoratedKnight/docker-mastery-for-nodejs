FROM node:8

ENV CHARCOAL_FACTOR=0.1

RUN apt-get update && apt-get install -y graphicsmagick

WORKDIR /app

COPY package*.json ./

RUN npm i && npm cache clean --force

COPY . .

# RUN mkdir in && mkdir out

# VOLUME [ "./in", "./out" ]

CMD [ "node", "index.js" ]