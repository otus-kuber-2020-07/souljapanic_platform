FROM nginx:alpine

ARG UID_USER
ARG APP_DIR

ENV UID_USER=${UID_USER:-'1001'} \
    APP_DIR=${APP_DIR:-'/app'}

RUN apk --no-cache add shadow \
    && usermod -u $UID_USER nginx

COPY ./default.conf /etc/nginx/conf.d

RUN sed -i 's/^user/#user/' /etc/nginx/nginx.conf \
    && sed -i 's/\/var\/run\/nginx\.pid/\/tmp\/nginx\.pid/' /etc/nginx/nginx.conf \
    && mkdir $APP_DIR \
    && chown -R nginx:nginx /var/cache/nginx /var/log/nginx $APP_DIR

USER nginx

EXPOSE 8000
