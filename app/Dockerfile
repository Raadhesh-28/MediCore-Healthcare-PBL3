
FROM raadhesh/healthcare-app:secure-v2

# Copy the latest MediCore application
COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
