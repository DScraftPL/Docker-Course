```dockerfile
FROM ubuntu:latest

# Aktualizacja systemu i instalacja nginx
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && 

# Skopiowanie plików konfiguracyjnych
COPY nginx.conf /etc/nginx/nginx.conf
COPY index.html /var/www/html/index.html

# Ustawienie uprawnień
RUN chmod 644 /var/www/html/index.html

# Eksponowanie portu 80
EXPOSE 80

# Uruchomienie nginx w trybie foreground
CMD ["nginx", "-g", "daemon off;"]
```