FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

FROM nginx:alpine

COPY nginx.conf /etc/nginx/conf.d/default.conf

RUN mkdir -p /usr/share/nginx/html/react-basics-notes-todo

COPY --from=build /app/dist/ /usr/share/nginx/html/react-basics-notes-todo/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]