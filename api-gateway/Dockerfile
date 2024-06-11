FROM node:14.21.3

WORKDIR /opt/app

COPY package.json yarn.lock ./

RUN npm install && npm install --save-dev @types/node@latest

COPY . .

RUN npm run build

CMD npm start
