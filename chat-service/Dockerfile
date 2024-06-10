FROM node:14.15.0

WORKDIR /opt/app

COPY package.json ./

RUN npm install

COPY . .

RUN npm run build

CMD ["node", "-r", "module-alias/register", "dist/index.js"]
