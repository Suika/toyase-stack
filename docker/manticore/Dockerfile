FROM manticoresearch/manticore:latest

RUN apt-get update && apt-get -y install mariadb-client && rm -rf /var/lib/apt/lists/*
COPY sphinx.sh /etc/sphinxsearch/sphinx.conf
VOLUME /etc/sphinxsearch

ENV DB_HOST=4chan-scrape-db \
    DB_ASAGI_NAME=asagi \
    DB_PORT=3306 \
    DB_SPHINX_USER=sphinx \
    DB_SPHINX_PASS=pass \
