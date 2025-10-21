# DOCKER

Kontenery współdzielą system operacyjny, który odpowiada za zachowanie separacji między kontenerami. 
Wirutalizacja lekka - CPU i RAM są współdzielone z maszyną gospodarza, mają różne nazwy, adresy IP i przestrzeń dyskową.

Klient - Serwer. Serwer do daemon Dockera, zajmuje się budowaniem, uruchamianiem kontenerów. Komunikujemy się poprzez klienta z nim.

Aby uruchomić kontener, potrzebny jest obraz kontenera, "Płyta startowa" pozwalająca uruchomić kontener.
Obraz kontenera jest zbiorem warstw tylko do odczytu. Pozwala to cachować warstwy, oraz kontenery mogą je współdzielić.

Kontenery można pobierać z repozytorium kontenerów [Docker Hub](https://hub.docker.com/). Jeżeli polecenie nie wykryje obrazu lokalnie, pobierze go z internetu.

Wybrane popularne obrazy: 
- [Nginx](https://hub.docker.com/_/nginx), 
- [Alpine](https://hub.docker.com/_/alpine), 
- [Postgres](https://hub.docker.com/_/postgres), 
- [Mongo](https://hub.docker.com/_/mongo),
- [Oracle Express DB](https://container-registry.oracle.com/ords/f?p=113:4:12025339938025:::4:P4_REPOSITORY,AI_REPOSITORY,AI_REPOSITORY_NAME,P4_REPOSITORY_NAME,P4_EULA_ID,P4_BUSINESS_AREA_ID:803,803,Oracle%20Database%20Express%20Edition,Oracle%20Database%20Express%20Edition,1,0&cs=3A8vzUW1Iz8YE2n8y0d3P7xBqWd-mqjs7iWl91dJ1kKE4R8xcfbvr45eZKywdFRpfsl24U_-xEMdGEZyeQTKNoA),
- [Scrath](https://hub.docker.com/_/scratch),

## Tworzenie i uruchamianie obrazu (z1)

`docker init` tworzy pliki compose.yaml + Dockerfile 

Dockerfile to plik określający, jak Docker ma zbudować obraz. Kolejność poleceń ma znaczenie, ponieważ wpływa na cache budowania. Pisanie pliku jest deklaratywne!

```Dockerfile
# Use an official base image
FROM node:18-alpine AS base

# Set working directory
WORKDIR /app

# Install dependencies separately for better caching
# Copy package files first
COPY package*.json ./

# Install production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application source code
COPY . .

# Create a non-root user for security
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

# Change ownership of app directory
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Expose the application port
EXPOSE 3000

# Set environment variables
ENV NODE_ENV=production \
    PORT=3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js || exit 1

# Define the command to run the application
CMD ["node", "server.js"]
```

Wybrane polecenia:
- `FROM <image>` - źródłowy obraz
- `RUN <command>` - uruchamia polecenie w nowej warstwie.
- `WORKDIR <dir>` - ustawia roboczy katalog dla poleceń
- `CMD <command>` - definiuje domyślny program, który ma zostać uruchomiony, po uruchomieniu kontenera
- `COPY <src> <desc>` - kopiuje pliki z src do dest
- `ENV <var>=<value>`- ustawia zmienną środowiskową systemu Linux
- `EXPOSE <port>` - określa, na którym porcie kontener ma nasłuchiwać na połączenia
- `ARG <var>=<value>` - definiuje zmienną, którą można podać w linii poleceń z flagą `--build-arg <var>=<value>`

`docker build -t <nazwa>:<tag> <dir>` - pozwala zbudować kontener, tworząc obraz. Nie uruchamia kontenera. 
tag to identyfikator kontenera, odróżniający np. wersję, ścieżka jest potrzebna w poleceniu aby określić kontekst budowania. 

`docker run <nazwa>:<tag>` - uruchamia wskazany obraz, tworząc kontener. 

Flaga `-p host:kontener`

`docker run ubuntu /bin/bash` - brak dostępu do kontenera
`docker run -it ubuntu /bin/bash` - dostęp do kontenera, 
Ctrl+P + Ctrl+Q zamyka terminal, nie wyłącza kontenera.
Ctrl+D wyłącza kontener.

## Wieloetapowe budowanie obrazu (z2)

2 przypadki użycia:

1. Kompilacja
  - Kontener budujący, zawierający wszystkie zależności i dane, przekazuje program.
  - Kontener uruchamiający, który zawiera przebudowany program.
2. Testowanie
  - Kontener budujący, przekazuje program
  - Kontener testujący, który sprawdza czy program poprawnie działa
  - Kontener uruchamiający, wdrażany w środowisku docelowym 

To rozwiązanie pozwala pozbyć się nie potrzebnych plików programu (np. kodu źródłowego), redukując miejsce na dysku.

## Wybrane polecenia Docker

- `docker ps` - wypisuje kontenery,
  - `-a` - wszystkie
  - `-l` - ostatni kontener
- `docker rm` - usuwa kontenery
- `docker images` - wypisuje obrazy kontenerów
- `docker rmi` - usuwa obrazy kontenerów
- `docker pull` - pobiera obraz kontenera
- `docker stop` - zatrzymuje kontener
- `docker exec` - pozwala uruchomić polecenie w działającym kontenerze
  - `-d` - w tle
- `docker restart` - restart kontenera
- `docker (un)pause` - pauzowanie, odpauzowanie kontenera
- `docker attach` - przyłącza konsolę do kontenera 

## Docker Compose

Zamiast tworzyć każdy indywidualny kontener, można stworzyć przepis, który zarządza innymi przepisami. Docker Compose jest mechanizmem deklaratywnym, pisanym w formacie YAML.

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - app-network
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  db_data:

networks:
  app-network:
    driver: bridge
```

Wybrane polecenia:
- `docker-compose up`

## Źródła

[https://www.geeksforgeeks.org/devops/docker-instruction-commands/](https://www.geeksforgeeks.org/devops/docker-instruction-commands/)
[https://www.frontstack.pl/blog/docker-komendy](https://www.frontstack.pl/blog/docker-komendy)
[https://docs.docker.com/engine/containers/run/](https://docs.docker.com/engine/containers/run/)
[https://docs.docker.com/reference/cli/docker/init/](https://docs.docker.com/reference/cli/docker/init/)
