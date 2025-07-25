### build stage
FROM node:18.20.8-alpine3.21 as builder
WORKDIR /app

COPY package.json yarn.lock ./
# lock version và clean cache khi yarn install tạo ra
RUN yarn install --frozen-lockfile && yarn cache clean 

COPY . .
RUN yarn build

### run stage
FROM nginx:1.29.0-alpine3.22

WORKDIR /run
COPY --from=builder /app/build /run
COPY nginx.conf /etc/nginx/nginx.conf


# COPY package.json yarn.lock ./
# RUN yarn install --frozen-lockfile && yarn cache clean

# COPY . .
# RUN yarn build

# => sử dụng cách này thì code thôi đủ nhưng vẫn cache lại được layer COPY và RUN đầu chỉ cần chạy mới 2 layer sau
# nếu chỉ có 2 layer dưới thì dù 2 file package.json và yarn.lock có thay đổi hay không mà code có thay đổi thì vẫn bị thay đổi cả context -> thay đổi layer nên ko có cache -> chạy tất cả layer mà ko cache