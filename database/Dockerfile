# Wykorzystujemy obraz Alpine ze względu na minimalny rozmiar i wysoką ocenę bezpieczeństwa [cite: 6, 10]
# Obraz ten wspiera natywnie architektury linux/amd64 oraz linux/arm64 
FROM postgres:16-alpine

# Dobre praktyki: Dodanie metadanych obrazu [cite: 7]
LABEL maintainer="Twoje Imie"
LABEL project="TaskLite"
LABEL version="1.0"

# Opcjonalnie: Jeśli masz skrypt inicjalizujący bazę (np. schema.sql), odkomentuj poniższą linię:
# COPY ./init.sql /docker-entrypoint-initdb.d/

# Dobre praktyki: Mechanizm Healthcheck pozwala Docker Compose stwierdzić, 
# czy baza jest już gotowa na przyjmowanie połączeń z Backend-u [cite: 13]
HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
  CMD pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB} || exit 1

# Baza PostgreSQL domyślnie używa portu 5432
EXPOSE 5432

# UWAGA: W obrazach bazodanowych nie zmieniamy użytkownika na USER 1000, 
# ponieważ PostgreSQL musi mieć uprawnienia roota podczas startu do zarządzania systemem plików, 
# a następnie samoczynnie przełącza się na bezpiecznego użytkownika 'postgres'.