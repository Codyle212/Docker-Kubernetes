FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

ENV MONGODB_USERNAME=admin
ENV MONGODB_PASSWORD=adminpass
ENV MONGODB_URL=cluster0.wxmn5be.mongodb.net
ENV MONGODB_NAME=goals-dev

CMD ["npm", "start"]