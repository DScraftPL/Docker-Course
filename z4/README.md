1. Baza danych (PostgreSQL)

- Użyj obrazu postgres:15-alpine
- Nazwa kontenera: blog_database
- Zmienne środowiskowe:
  - POSTGRES_DB: blogdb
  - POSTGRES_USER: bloguser
  - POSTGRES_PASSWORD: secretpassword123
- Port: 5432 (mapowany na host)
- Volume: dane bazy powinny być persystowane w wolumenie nazwanym db_data

2. Aplikacja backendowa (Node.js)

- Użyj obrazu node:18-alpine
- Nazwa kontenera: blog_backend
- Port: 3000 (mapowany na host jako 3000)
- Zmienne środowiskowe:
  - DATABASE_URL: postgresql://bloguser:secretpassword123@blog_database:5432/blogdb
  - NODE_ENV: production
- Zależność: backend powinien startować dopiero po uruchomieniu bazy danych
- Volume: zamontuj katalog ./backend do /app w kontenerze

3. Nginx (reverse proxy)

- Użyj obrazu nginx:alpine
- Nazwa kontenera: blog_nginx
- Port: 80 (mapowany na host jako 8080)
- Volume: zamontuj plik konfiguracyjny ./nginx.conf do /etc/nginx/nginx.conf
- Zależność: nginx powinien startować po backendzie