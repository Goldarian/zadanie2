Autor: Kacper Staniek 6.3 Ti Zadanie 2

Plik docker-build.yml w folderze .github/workflows:
```
name: Docker Build

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Pobranie repozytorium
      uses: actions/checkout@v2
      # Pobieranie kod źródłowy repozytorium
    
    - name: Konfiguracja QEMU
      uses: docker/setup-qemu-action@v1
      # Konfiguracja QEMU dla wielu architektur
    
    - name: Konfiguracja Docker Buildx
      uses: docker/setup-buildx-action@v1
      # Konfiguracja Docker Buildx
    
    - name: Logowanie do DockerHub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
      # Logowanie do DockerHub
    
    - name: Cache Docker
      uses: actions/cache@v2
      with:
        path: /tmp/.buildx-cache
        key: ${{ runner.os }}-docker-${{ github.sha }}
        restore-keys: |
          ${{ runner.os }}-docker-
      # Przechowywanie buforowanych warstw obrazu Docker w celu przyspieszenia procesu budowy
    
    - name: Budowa obrazu dla architektury x86_64 i wysłanie go do DockerHub
      uses: docker/build-push-action@v2
      with:
        context: .
        file: Dockerfile
        platforms: linux/amd64
        push: true
        tags: goldarian/zadanie2:latest
        cache-from: type=local,src=/tmp/.buildx-cache
        cache-to: type=local,dest=/tmp/.buildx-cache
      # Budowa obrazu dla architektury x86_64 i wysłanie go do DockerHub
      
    - name: Budowa obrazu dla architektury arm64 i wysłanie go do DockerHub
      uses: docker/build-push-action@v2
      with:
        context: .
        file: Dockerfile
        platforms: linux/arm64
        push: true
        tags: goldarian/zadanie2:latest-arm64
        cache-from: type=local,src=/tmp/.buildx-cache
        cache-to: type=local,dest=/tmp/.buildx-cache
      # Budowa obrazu dla architektury arm64 i wysłanie go do DockerHub
```
Plik Dockerfile:
```
FROM node:alpine
ENV NODE_OPTIONS=--openssl-legacy-provider
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx
EXPOSE 80
COPY --from=0 /app/build /usr/share/nginx/html
```
Aplikacja została zmodyfikowana tak, aby wyświetlała imię i nazwisko:

![image](https://github.com/Goldarian/zadanie2/assets/77618644/6e89e75b-f0f3-4e9b-94a5-1b4d29977506)

Plik App.js:
```
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Kacper Staniek
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
```

Łańcuch Github Actions został uruchomiony:

![image](https://github.com/Goldarian/zadanie2/assets/77618644/6f59f397-9d7e-4148-8644-b895ca7a6ac4)

Nie mogłem przetestować obrazu pod kątem CVE, ponieważ pojawiał się błąd:

![image](https://github.com/Goldarian/zadanie2/assets/77618644/58f7691a-1403-437b-82f0-754e3a850a57)
