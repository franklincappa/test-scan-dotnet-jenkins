# Escenario 5: Escaneo de Seguridad y Análisis Estático

Este escenario simula un pipeline de CI/CD en Jenkins para una API desarrollada con .NET Core 9.
Incluye etapas de análisis estático con SonarQube y escaneo de vulnerabilidades con Trivy.

## 🛠 Requisitos

### Jenkins Plugins Recomendados
- Docker Pipeline
- Pipeline
- Git Plugin
- Blue Ocean
- Pipeline Utility Steps
- SonarQube Scanner for Jenkins

### Docker
Asegúrate de tener Docker instalado y disponible en el host de Jenkins.

### SonarQube
1. Instala SonarQube localmente o en un servidor.
2. Crea un nuevo proyecto en SonarQube.
3. Genera un token y configúralo en Jenkins como `SONAR_TOKEN`.

### Trivy
Instala Trivy en el host de Jenkins con:

```bash
apt install wget -y
wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.1_Linux-64bit.deb
dpkg -i trivy_0.50.1_Linux-64bit.deb
```

## 🧪 API de Prueba

El proyecto `ApiScanTest` es una API básica de .NET Core 9.

### Cómo correrla localmente
```bash
dotnet build
dotnet run --project ApiScanTest/ApiScanTest.csproj
```
La API se expondrá en: `https://localhost:5001`

### Swagger habilitado
Accede a: [https://localhost:5001/swagger](https://localhost:5001/swagger)

## 📦 Dockerfile
El `Dockerfile` construye y ejecuta la API expuesta en el puerto 80.

```
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore "./ApiScanTest.csproj"
RUN dotnet build "./ApiScanTest.csproj" -c Release -o /app/build
RUN dotnet publish "./ApiScanTest.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "ApiScanTest.dll"]
```
---

## 🔐 Jenkinsfile

El pipeline incluye:
1. Clonado del código.
2. Análisis estático con SonarQube.
3. Construcción de imagen Docker.
4. Escaneo de la imagen con Trivy.
5. Despliegue si todo pasa sin CVEs críticos.

Token de SonarQube debe estar configurado como variable segura en Jenkins con el nombre `SONAR_TOKEN`.

```groovy
withSonarQubeEnv('MySonarQubeServer') {
    sh 'sonar-scanner -Dsonar.projectKey=ApiScanTest -Dsonar.sources=. -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_AUTH_TOKEN'
}
```


## 📤 Comando para Compilar y Probar Localmente

```bash
dotnet build
dotnet run --project ApiScanTest.csproj
```

Luego prueba en Postman:
```
http://localhost:5006/weatherforecast
```

---

## 🔁 Exposición del Servicio

La API .NET Core (`ApiScanTest`) se expone externamente a través del puerto **5006**. Esto se logra mediante el mapeo de puertos en la etapa de despliegue del `Jenkinsfile`, donde se configura:

```bash
docker run -d -p 5006:8080 --name api-scan-test api-scan-test
```

Esto indica que el contenedor escucha en el puerto **8080** interno, pero se accede desde fuera del contenedor por el **5006**, facilitando las pruebas con herramientas como Postman.

---

**Autor** 
Franklin Cappa Ticona  
DevSecOps - DBCODE Consulting
