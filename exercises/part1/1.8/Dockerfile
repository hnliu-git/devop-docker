FROM ubuntu:18.04

RUN apt-get update \
    &&  apt-get install --yes curl

CMD echo "Input website:" \
    && read website \
    && echo "Searching.." \
    && sleep 1 \
    && curl http://$website
