FROM node:10

WORKDIR /usr/src

COPY ["package.json", "package-lock.json", "/usr/src/"]
# COPY package.json ./

RUN npm install

COPY . ./

EXPOSE 3000

CMD ["npm","start"]
