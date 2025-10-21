```docker
version: '3.8'

services:
  # Baza danych PostgreSQL
  blog_database:
    image: postgres:15-alpine
    container_name: blog_database
    environment:
      POSTGRES_DB: blogdb
      POSTGRES_USER: bloguser
      POSTGRES_PASSWORD: secretpassword123
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - blog_network
    restart: always

  # Aplikacja backendowa
  blog_backend:
    image: node:18-alpine
    container_name: blog_backend
    environment:
      DATABASE_URL: postgresql://bloguser:secretpassword123@blog_database:5432/blogdb
      NODE_ENV: production
    ports:
      - "3000:3000"
    volumes:
      - ./backend:/app
    working_dir: /app
    command: npm start
    depends_on:
      - blog_database
    networks:
      - blog_network
    restart: always

  # Nginx reverse proxy
  blog_nginx:
    image: nginx:alpine
    container_name: blog_nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - blog_backend
    networks:
      - blog_network
    restart: always

# Definicja sieci
networks:
  blog_network:
    driver: bridge

# Definicja wolumenów
volumes:
  db_data:
```