FROM nginx:latest

COPY nginx.conf /etc/nginx/nginx.conf
COPY server.key /etc/nginx/certs/server.key
COPY server.crt /etc/nginx/certs/server.crt

CMD ["nginx", "-g", "daemon off;"]