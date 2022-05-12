# syntax=docker/dockerfile:1
FROM  python:3.10.4-alpine3.15 as build
RUN pip install --no-cache-dir mkdocs==1.3.0 mkdocs-material==8.2.14 

WORKDIR /docs
COPY ./ ./

RUN python -m mkdocs build

FROM nginx:1.21.6-alpine
COPY --from=build /docs/site /usr/share/nginx/html

EXPOSE 80
