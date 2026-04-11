# Opracować pliki Dockerfile dla każdego mikroserwisu jaki został zaplanowany w ramach architektury z punktu 1. <br/> Ocena zawartości poszczególnych plików Dockerfile opierać się będzie na zgodności z dobrymi praktykami stosowanymi w procesie budowania obrazów. Proszę przypomnieć sobie zalecenia zebrane w dokumentacji środowiska Docker: https://docs.docker.com/build/building/bestpractices/
## Frontend (nginx)
### Etap 1 Budowanie (Build Stage)
```
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
```
### Etap 2: Produkcja (Production Stage)
```
FROM nginx:1.25-alpine
# Dobre praktyki: Kopiujemy tylko niezbędne pliki z etapu budowania
COPY --from=build /app/dist /usr/share/nginx/html
# Zmiana uprawnień dla bezpieczeństwa (non-root)
RUN touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid /var/cache/nginx /var/log/nginx
USER nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
## Backend (python API)

### Etap 1: Pobieranie zależności
```
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt
```
### Etap 2: Finalny obraz
```
FROM python:3.11-slim
WORKDIR /app
# Kopiujemy zainstalowane paczki z poprzedniego etapu
COPY --from=builder /root/.local /home/appuser/.local
COPY . .

# Dobre praktyki: Uruchamianie jako użytkownik o niskich uprawnieniach
RUN useradd -m appuser
USER appuser
ENV PATH=/home/appuser/.local/bin:$PATH

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Baza danych (postgre sql)
```
# Wybieramy stabilną, lekką wersję Alpine, aby zminimalizować podatności (Zadanie 4)
FROM postgres:16-alpine

# Dobre praktyki: Opisanie obrazu (ułatwia identyfikację w DockerHub)
LABEL maintainer="Twoje Imie <twoj-email@przyklad.pl>"
LABEL description="Baza danych PostgreSQL dla aplikacji TaskLite z inicjalizacją schematu."

# (Opcjonalnie) Kopiujemy skrypty SQL, które wykonają się automatycznie przy starcie
# Każdy plik .sql wrzucony do tego katalogu zostanie uruchomiony przy pierwszym starcie kontenera
# COPY ./init-scripts/ /docker-entrypoint-initdb.d/

# Dobre praktyki: Nie musimy definiować USER, bo obraz postgres ma własne mechanizmy bezpieczeństwa,
# ale warto upewnić się, że port jest udokumentowany
EXPOSE 5432
```
