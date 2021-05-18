FROM nginx:stable
ARG WORKDIR=/app
ARG PORT=80

WORKDIR $WORKDIR

RUN echo " \n\
server { \n\
    listen       $PORT; \n\
    server_name  _; \n\
    location / { \n\
      root $WORKDIR; \n\
    } \n\
    location = /basic_status { \n\
      stub_status; \n\
    } \n\
}" > /etc/nginx/conf.d/default.conf

RUN echo " \n\
<p>kubernetes-monitoring</p> \n\
" > index.html

EXPOSE $PORT