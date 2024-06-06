FROM node:14.15.0

WORKDIR /opt/app

COPY package.json yarn.lock ./

RUN yarn install

COPY . .

CMD yarn db:migrate && yarn watch
