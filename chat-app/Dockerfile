FROM node:16

WORKDIR /app

COPY package.json yarn.lock ./

RUN yarn install

COPY . .

EXPOSE 7001

CMD ["yarn", "watch"]
