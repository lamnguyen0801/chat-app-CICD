FROM node:18.20.8-alpine3.21 as builder

# Cài các gói cần cho node-gyp (bcrypt)
RUN apk add --no-cache make g++ python3

USER node
# Đặt USER lên trên để ko cần chown cho thư mục workdir (như trên thì thư mục workdir sẽ tự động có user là node)
WORKDIR /home/node/app

COPY --chown=node:node package.json yarn.lock ./
RUN yarn install --frozen-lockfile && yarn cache clean 
COPY --chown=node:node . .

EXPOSE 5000
CMD [ "yarn", "start" ]


# Use --chown option of Docker's COPY command instead of doing it manually to reduce build time.

#  # manually changing owner
#  COPY . $APP_HOME
#  RUN chown -r app:app $APP_HOME

#  # using --chown option
#  COPY --chown=app:app . $APP_HOME


# expose là container đang lắng nghe cổng 5000 nhưng chưa cho access từ bên ngoài (docker run -p 5000:5000 để expose thực sự)
# FE không cần expose vì chỉ là static files, BE là nơi xử lý logic, nên cần EXPOSE để các service khác có thể giao tiếp (vd DB hoặc để FE call API)

## có thể tùy chỉnh bắt đầu sử dụng USER bắt đầu từ đâu tùy tình huống
# Đặt USER ở sau phần hệ thống (setup, quyền), nhưng trước phần chạy app
# => Mục tiêu là: dùng root cho setup, và non-root để chạy app an toàn