FROM wordpress:latest
ARG USER_ID
ARG GROUP_ID
RUN sed -i -e "s/33:33/${USER_ID}:${GROUP_ID}/g" /etc/passwd \
  && sed -i -e "s/33:/${GROUP_ID}:/g" /etc/group \
  && chown -R www-data:www-data /var/www
