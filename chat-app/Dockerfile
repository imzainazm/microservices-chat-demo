FROM node:16.16.0 AS build

WORKDIR /app

COPY package.json tsconfig.json ./

RUN yarn install

COPY . .

RUN yarn build

FROM nginx:1.21.4-alpine

COPY --from=build /app/dist /usr/share/nginx/html

COPY 50x.html /usr/share/nginx/html/50x.html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
