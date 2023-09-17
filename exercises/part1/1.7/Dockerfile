FROM ubuntu:20.04

RUN apt update && apt install -y curl

WORKDIR /user/src/app

COPY curl_web.sh . 

RUN chmod +x ./curl_web.sh

CMD ./curl_web.sh