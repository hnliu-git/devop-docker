FROM ubuntu:18.04

WORKDIR /app

RUN apt-get update && apt-get install git-core curl xsel --yes

# Install node and npm
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash && apt-get update

RUN apt install -y nodejs

RUN node -v && npm -v

# Clone the git project
RUN cd /app && git clone https://github.com/docker-hy/material-applications.git

EXPOSE 5000

# Enter the path
RUN cd /app/material-applications/example-frontend/ \
       && npm install \
       && npm run build \
       && npm install -g serve

ENTRYPOINT ["serve", "-s", "-l", "5000"]

CMD ["/app/material-applications/example-frontend/build/"]
