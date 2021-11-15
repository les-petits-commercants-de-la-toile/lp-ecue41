FROM mysql:5.7
ARG USER_ID
ARG GROUP_ID
RUN sed -i -e "s/999:999/${USER_ID}:${GROUP_ID}/g" /etc/passwd \
  && sed -i -e "s/999:/${GROUP_ID}:/g" /etc/group \
  && chown -R mysql:mysql /var/lib/mysql
