FROM node:14.18.1-alpine
WORKDIR /app
COPY sys-stats .
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]