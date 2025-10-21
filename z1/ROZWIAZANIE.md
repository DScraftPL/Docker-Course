1.

```bash
# 1. Pobierz Ubuntu
docker pull ubuntu

# 2. Uruchom kontener
docker run -dit --name pingwin ubuntu

# 3. Zatrzymaj kontener
docker stop pingwin

# 4. Pokaż działające kontenery
docker ps

# 5. Pokaż wszystkie kontenery
docker ps -a
```

2. 

```bash
# Uruchom kontener o nazwie "kontenerek" z obrazu Alpine, który pinguje google.com
docker run -d --name kontenerek alpine ping google.com

# Sprawdź czy kontener działa
docker ps

# Zobacz logi na żywo (CTRL+C aby wyjść)
docker logs -f kontenerek
```

3. 
```bash
# Krok 1: Utwórz i uruchom kontener o nazwie "aktualizator" z obrazu Ubuntu
docker run -it --name aktualizator ubuntu bash

# Krok 2: Wewnątrz kontenera wykonaj aktualizację systemu
apt update && apt upgrade -y

# Krok 3: Wyjdź z kontenera
exit

# Krok 4: Zapisz kontener jako nowy obraz
docker commit aktualizator ubuntu:zaktualizowany

# Sprawdź czy nowy obraz został utworzony
docker images

# Uruchom kontener z nowego obrazu aby przetestować
docker run -it ubuntu:zaktualizowany bash
```
