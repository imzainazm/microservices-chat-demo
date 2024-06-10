FROM node:16.0.0

WORKDIR /opt/app

COPY package.json ./

RUN yarn install

COPY . .

CMD ["yarn", "watch"]
