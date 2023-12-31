FROM node:18-slim as builder

ENV FRONTEND_DIR="/frontend"
WORKDIR ${FRONTEND_DIR}

# 复制前端代码并安装依赖
COPY ./frontend/package.json .
RUN yarn install --network-timeout 1000000
COPY ./frontend/ .

# 构建前端
RUN yarn run build

FROM node:18-slim as app

# 安装 nginx
RUN apt-get update  \
    && npm config set update-notifier false \
    && apt-get install -y nginx git gnupg wget chromium

ENV WORKDIR="/app" \
    FRONTEND_DIR="/frontend" \
    SERVER_DIR="/server"

# 设置工作目录
WORKDIR ${WORKDIR}

RUN mkdir ${FRONTEND_DIR} && mkdir ${SERVER_DIR}

# 复制构建完成的前端文件到/app/frontend
COPY --from=builder ${FRONTEND_DIR}/dist/ ${WORKDIR}/${FRONTEND_DIR}/

# 安装后端依赖
COPY ./server/package.json ./server/
RUN cd server && yarn install --network-timeout 1000000
COPY ./server/ ./server/

# 复制 nginx 配置文件
COPY ./deploy/nginx.conf /etc/nginx/sites-enabled/default
EXPOSE 80

ENTRYPOINT service nginx start && cd server && yarn run start
