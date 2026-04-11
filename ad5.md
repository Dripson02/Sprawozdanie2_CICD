# Opracować testową (deweloperską) wersję pliku docker-compose.yaml, na podstawie której możliwe będzie uruchomienie pierwszej, testowej wersji aplikacji i tym samym potwierdzenie poprawności funkcjonalnej zaprojektowanej aplikacji/systemu. <br/> Ocena zawartości pliku docker-compose.yaml opierać się będzie głównie na pierwszych 7 punktach z bloga na temat dobrych praktyk przy opracowywaniu aplikacji w środowisku Docker Compose

## Services
```
  db:
    build: ./database
    container_name: tasklite-db
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend-net
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
```
## Backend
```
 build: ./backend
    container_name: tasklite-api
    env_file:
      - .env
    depends_on:
      - db
    networks:
      - backend-net
      - frontend-net
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
```
## Frontend
```
  build: ./frontend
    container_name: tasklite-ui
    ports:
      - "8080:80"
    depends_on:
      - backend
    networks:
      - frontend-net
    deploy:
      resources:
        limits:
          cpus: "0.25"
          memory: 128M
```
## Networks
```
  frontend-net:
  backend-net:
```
## Volumes
```
  db_data:
```
## Plik .env
```
POSTGRES_USER=user
POSTGRES_PASSWORD=password123
POSTGRES_DB=tasklite_db
DATABASE_URL=postgresql://user:password123@db:5432/tasklite_db
```

## Analiza zgodności pliku docker-compose.yaml z 7 dobrymi praktykami (wg. W. Freitasa):
1. **Use Versioning Properly (Prawidłowe wersjonowanie):** Na samej górze pliku zadeklarowano najnowszą, stabilną wersję schematu Compose, co zapewnia pełną kompatybilność z nowymi funkcjami platformy Docker.
2. **Keep Services Modular and Purpose-Driven (Modułowość i celowość usług):** Architektura została podzielona na trzy niezależne kontenery: db (tylko baza danych), backend (tylko logika API) oraz frontend (tylko serwowanie UI). Każdy serwis realizuje wyłącznie jedno przypisane mu zadanie.
3. **Use Named Volumes for Data Persistence (Nazwane wolumeny dla trwałości danych):** W celu zabezpieczenia danych przed utratą po restarcie kontenera, na samym dole pliku zdefiniowano nazwany wolumen db_data, który następnie został przypięty do ścieżki /var/lib/postgresql/data w serwisie db.
4. **Leverage Environment Variables (Wykorzystanie zmiennych środowiskowych):** Usunięto zaszyte na sztywno (hardcoded) hasła i dane logowania z pliku konfiguracyjnego. Zamiast tego użyto dyrektywy env_file: - .env, która bezpiecznie wstrzykuje sekrety do kontenerów bazy danych oraz backendu.
5. **Define Networks Explicitly (Jawne definiowanie sieci):** Stworzono dwie odseparowane sieci: frontend-net oraz backend-net. Serwis db nie ma dostępu do frontend-net, co zwiększa bezpieczeństwo systemu poprzez ścisłą kontrolę komunikacji między węzłami.
6. **Use Dependencies Carefully (Ostrożne używanie zależności):** Wykorzystano klucz depends_on, aby zdefiniować kolejność uruchamiania kontenerów (Frontend czeka na Backend, a Backend na Bazę Danych). Minimalizuje to błędy połączeń w pierwszych sekundach startu systemu.
7. **Apply Resource Limits (Limity zasobów CPU i Pamięci):** W każdym serwisie zastosowano blok deploy -> resources -> limits, przypisując konkretne maksymalne wartości zasobów (np. 512MB RAM i 0.5 rdzenia dla bazy). Zapobiega to sytuacji, w której jeden wadliwy kontener zużywa wszystkie zasoby maszyny hosta, powodując awarię całego systemu.
