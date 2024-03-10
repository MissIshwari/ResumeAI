FROM node:18 as rm-backend

WORKDIR /app

COPY . .

RUN npm install --force
RUN npm run build

FROM nginx:alpine
COPY --from=rm-backend /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx","-g","daemon off;"]
