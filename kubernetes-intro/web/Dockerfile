FROM httpd:2.4
RUN useradd --user-group --system --create-home --no-log-init --uid 1001 otuslab02
RUN mkdir /app && chown otuslab02:otuslab02 /app && chown -R otuslab02:otuslab02 /usr/local/apache2/
RUN sed -i 's/User daemon/User otuslab02/g' /usr/local/apache2/conf/httpd.conf && \
    sed -i 's/Group daemon/Group otuslab02/g' /usr/local/apache2/conf/httpd.conf && \
    sed -i 's/Listen 80/Listen 8000/g' /usr/local/apache2/conf/httpd.conf
RUN echo '<Directory "/app">' >> /usr/local/apache2/conf/httpd.conf && \
    echo '    Options Indexes FollowSymLinks' >> /usr/local/apache2/conf/httpd.conf && \
    echo '    AllowOverride None' >> /usr/local/apache2/conf/httpd.conf && \
    echo '    Require all granted' >> /usr/local/apache2/conf/httpd.conf && \
    echo '</Directory>' >> /usr/local/apache2/conf/httpd.conf
RUN sed -i 's/DocumentRoot \"\/usr\/local\/apache2\/htdocs\"/DocumentRoot \"\/app\"/g' /usr/local/apache2/conf/httpd.conf
EXPOSE 8000
USER otuslab02