FROM node:20-alpine
WORKDIR /app
ARG SERVICE_NAME
COPY ${SERVICE_NAME}/package*.json ./
RUN npm ci --omit=dev
COPY ${SERVICE_NAME}/ ./
ENV NODE_ENV=production
CMD ["npm", "start"]
