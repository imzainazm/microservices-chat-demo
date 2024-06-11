FROM node:14.15.0

WORKDIR /app

COPY package.json ./

RUN npm install

COPY . .

RUN npm run clean && tsc

CMD npm start
