# Escenario 5: Escaneo de Seguridad y Análisis Estático

Este escenario simula un pipeline de CI/CD en Jenkins para una API desarrollada con .NET Core 9.
Incluye etapas de análisis estático con SonarQube y escaneo de vulnerabilidades con Trivy.

## 🛠 Requisitos

### Jenkins Plugins Recomendados
- Docker Pipeline
- Pipeline
- Git Plugin
- Blue Ocean (Opcional)
- Pipeline Utility Steps
- SonarQube Scanner for Jenkins
- Quality Gates

### Docker
Asegúrate de tener Docker instalado y disponible en el host de Jenkins.

### SonarQube
1. Instala SonarQube localmente o en un servidor.
2. Crea un nuevo proyecto en SonarQube.
3. Genera un token y configúralo en Jenkins como `SONAR_TOKEN`.

### Trivy
Instala Trivy en el host de Jenkins con:

```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin v0.61.1
```

- Probar instalación:
```bash
trivy --version
```
```bash
trivy image nombre-imagen-analizar
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
EXPOSE 80
ENTRYPOINT ["dotnet", "ApiScanTest.dll"]
```
---

## 🔐 Jenkinsfile

El pipeline incluye:
1. **Clonar Código**: Desde GitHub.
2. **Build y Test**: Ejecuta `dotnet restore`, `build` y `test`.
3. **Escaneo SonarQube**: Instala `dotnet-sonarscanner`, realiza análisis estático.
4. **Espera de Calidad**: Aguarda el resultado del análisis de Sonar.
5. **Build Docker**: Construye la imagen con la API.
6. **Escaneo con Trivy**: Evalúa vulnerabilidades `CRITICAL` y `HIGH`.
7. **Despliegue**: Corre el contenedor Docker y expone el API.

Token de SonarQube debe estar configurado como variable segura en Jenkins con el nombre `SONAR_TOKEN`.

```groovy
withSonarQubeEnv('MySonarQubeServer') {
    sh 'sonar-scanner -Dsonar.projectKey=ApiScanTest -Dsonar.sources=. -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN'
}
```

---

## 🔁 Exposición del Servicio

La API .NET Core (`ApiScanTest`) se expone externamente a través del puerto **5006**. Esto se logra mediante el mapeo de puertos en la etapa de despliegue del `Jenkinsfile`, donde se configura:

```bash
docker run -d -p 5006:80 --name api-scan-test api-scan-test
```


Esto indica que el contenedor escucha en el puerto **80** interno, pero se accede desde fuera del contenedor por el **5006**, facilitando las pruebas con herramientas como Postman.

```
http://localhost:5006/swagger
```


---

## 🧪 Resultado esperado
- El pipeline se detendrá si:
  - Fallan los tests.
  - El análisis de SonarQube devuelve una nota baja.
  - Trivy encuentra CVEs `CRITICAL` o `HIGH`.

---

**Autor** 
Franklin Cappa Ticona  
DevSecOps - DBCODE Consulting
