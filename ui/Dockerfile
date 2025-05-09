##############################################
## Build Stage
##############################################
FROM node:23-alpine AS builder

# PROXY_HOST is for backwards compatibility for older versions of Unitycatalog UI
# Not used in production build
ARG PROXY_HOST=localhost:8080
ENV PROXY_HOST=${PROXY_HOST}

RUN apk --no-cache add \
    sed 

WORKDIR /usr/src/app

# Copy package.json and lockfile to the working directory
COPY ./package.json ./
COPY ./yarn.lock ./

# Install dependencies
RUN yarn --frozen-lockfile

# Copy the rest of our files
COPY . .

# Backwards compatibility for older versions of Unitycatalog UI
RUN sed -i 's/localhost:8080/'$PROXY_HOST'/g' ./package.json

RUN yarn build

##############################################
## Deploy Stage - Local/Development 
##############################################
FROM node:23-alpine AS development

WORKDIR /usr/src/app

COPY --from=builder /usr/src/app/ .

EXPOSE 3000

CMD [ "yarn", "start" ]

##############################################
## Deploy Stage - Production 
##############################################
FROM nginx:1.28-alpine AS production

# Copy the build folder from the previous stage with the bundled app
COPY --from=builder /usr/src/app/build /usr/share/nginx/html

# Copy gzip conf
COPY ./nginx/gzip.conf /etc/nginx/conf.d/gzip.conf

# Overwrite the default nginx site conf with ours
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf

# Expose port of our nginx server serving the built html/js files
EXPOSE 3000
