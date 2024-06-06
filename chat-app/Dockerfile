FROM node:16.0.0

WORKDIR /opt/app

COPY package.json yarn.lock ./

RUN yarn install

COPY . .

CMD ["yarn", "watch"]
