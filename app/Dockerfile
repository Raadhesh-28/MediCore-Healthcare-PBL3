
FROM nginx:alpine

# Update installed Alpine packages to patched versions
RUN apk upgrade --no-cache

# Copy MediCore application
COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
