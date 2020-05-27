FROM postgres:11.6-alpine

COPY dbFiles/ /docker-entrypoint-initdb.d/

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr 
