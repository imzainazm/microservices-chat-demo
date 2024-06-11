FROM node:20.14.0

WORKDIR /opt/app

COPY package.json ./

RUN npm install

COPY . .

RUN npm run build

CMD npm start
