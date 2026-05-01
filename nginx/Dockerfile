#Start with base Nginx image, use alpine (lightweight)
FROM nginx:alpine

#Copy SSL certificates to container
COPY ssl/localhost.crt /etc/nginx/certs/localhost.crt
COPY ssl/localhost.key /etc/nginx/certs/localhost.key

#Copy nginx config file to specified location inside container
COPY nginx.conf /etc/nginx/conf.d/default.conf

#Expose port 80 for HTTP traffic
EXPOSE 80

#Expose port 443 for HTTPS traffic
EXPOSE 443
