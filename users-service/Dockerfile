FROM node:14

WORKDIR /opt/app

COPY package.json yarn.lock ./

RUN yarn install

COPY . .

CMD ["yarn", "watch"]
