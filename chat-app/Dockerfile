FROM node:16.16.0 AS build

WORKDIR /app

COPY package.json tsconfig.json ./

RUN yarn install

COPY . .

RUN yarn build

FROM nginx:1.21.4-alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
