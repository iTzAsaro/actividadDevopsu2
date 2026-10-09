# Guía Paso a Paso: Configuración de Pipeline CI/CD DevSecOps con Snyk, JaCoCo y SonarCloud
**Asignatura:** DOY0101 - Ingeniería DevOps / Seguridad y Calidad en Desarrollo de Software  
**Proyecto:** Spring Boot (`bdget-jacoco`) con Maven y GitHub Actions

---

## 🎯 Objetivo de la Actividad
Implementar un pipeline de Integración Continua (CI) en GitHub Actions que integre análisis de seguridad y pruebas de calidad automatizadas en 4 fases secuenciales:

1. **Snyk Security Scan (SCA):** Análisis de dependencias de código abierto y generación de reporte de vulnerabilidades.
2. **Tests & Cobertura con JaCoCo:** Ejecución de pruebas unitarias y generación de métricas de cobertura de código.
3. **SonarCloud Analysis (SAST):** Análisis estático de código, detección de code smells, vulnerabilidades y recepción del reporte JaCoCo.
4. **Build Final:** Empaquetado del archivo binario ejecutable (`.jar`).

Al finalizar, el pipeline debe quedar 100% en verde y generar **3 artefactos** descargables:
- `snyk-report` (`snyk-report.json`)
- `jacoco-report` (`target/site/jacoco/`)
- `app-jar` (`target/*.jar`)

---

## 📋 Requisitos Previos
- Cuenta en [GitHub](https://github.com/).
- Cuenta en [Snyk.io](https://snyk.io/) (puedes iniciar sesión con tu cuenta de GitHub).
- Cuenta en [SonarCloud.io](https://sonarcloud.io/) (puedes iniciar sesión con tu cuenta de GitHub).
- Git instalado en tu máquina local.

---

## 🚀 Paso a Paso de Configuración

### Paso 1: Crear tu Repositorio en GitHub y Subir el Proyecto
1. Ingresa a GitHub y crea un nuevo repositorio público (por ejemplo, `actividadDevopsu2` o `bdget-jacoco`). Déjalo vacío (sin README ni .gitignore).
2. Abre una terminal dentro de la carpeta local de tu proyecto y vincula tu repositorio remoto:
   ```bash
   # Reemplaza con la URL de tu repositorio personal
   git remote set-url origin https://github.com/<TU_USUARIO>/<TU_REPOSITORIO>.git
   git branch -M main
   git push -u origin main
   ```

---

### Paso 2: Obtener y Configurar el Token de Snyk (`SNYK_TOKEN`)
1. Ve a [https://app.snyk.io/account](https://app.snyk.io/account) (o ve a tu avatar ➔ **Account Settings**).
2. En la sección **Auth Token**, haz clic en **Click to show** y copia el token (o genera uno nuevo).
3. Ve a tu repositorio en GitHub:
   - Clic en **Settings** ➔ **Secrets and variables** (menú izquierdo) ➔ **Actions**.
   - Haz clic en el botón verde **New repository secret**.
   - **Name:** `SNYK_TOKEN`
   - **Secret:** *(Pega el token copiado de Snyk)*
   - Haz clic en **Add secret**.

---

### Paso 3: Configurar SonarCloud

#### 3.1 Generar el Token de SonarCloud (`SONAR_TOKEN`)
1. Ve a [https://sonarcloud.io/account/security/](https://sonarcloud.io/account/security/).
2. En el campo **Generate New Token**, escribe un nombre descriptivo (ej. `github-actions`) y haz clic en **Generate**.
3. Copia el token generado inmediatamente.
4. Ve a tu repositorio en GitHub (**Settings** ➔ **Secrets and variables** ➔ **Actions** ➔ **New repository secret**):
   - **Name:** `SONAR_TOKEN`
   - **Secret:** *(Pega el token generado en SonarCloud)*
   - Haz clic en **Add secret**.

#### 3.2 Importar el Proyecto en SonarCloud
1. En SonarCloud, haz clic en el icono **+** (arriba a la derecha) y selecciona **Analyze new project**.
2. Selecciona tu organización de GitHub y busca el repositorio que acabas de subir.
3. Haz clic en **Set Up**.
4. Anota los siguientes dos valores clave:
   - **Organization Key:** Suele ser tu usuario de GitHub o nombre de organización (ej. `itzasaro`).
   - **Project Key:** Suele tener el formato `<Organización>_<NombreRepositorio>` (ej. `iTzAsaro_actividadDevopsu2`).

#### 3.3 ⚠️ PASO CRÍTICO: Desactivar "Automatic Analysis"
> [!IMPORTANT]
> SonarCloud activa por defecto el "Análisis Automático". Si no se desactiva, el pipeline de GitHub Actions fallará con el error:  
> `[ERROR] You are running CI analysis while Automatic Analysis is enabled.`

1. Entra a tu proyecto en SonarCloud.
2. En la barra superior o menú de navegación del proyecto, ve a:  
   **Administration** ➔ **Analysis Method**  
   *(O ingresa directamente a: `https://sonarcloud.io/project/configuration?id=<TU_PROJECT_KEY>`)*.
3. Ubica el interruptor **Automatic Analysis** y cámbialo a **OFF / Disabled**.
4. Confirma la acción si la plataforma te lo solicita.

---

### Paso 4: Ajustar los Archivos del Proyecto

#### 4.1 Verificar `pom.xml`
Asegúrate de que dentro de la sección `<build><plugins>` estén declarados tanto JaCoCo como el plugin de Sonar:

```xml
<!-- Plugin de Cobertura JaCoCo -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>

<!-- Plugin de SonarCloud Scanner -->
<plugin>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>5.0.0.4389</version>
</plugin>
```

#### 4.2 Ajustar `.github/workflows/main.yml`
En la tarea de SonarCloud, actualiza los parámetros `-Dsonar.projectKey` y `-Dsonar.organization` con tus valores obtenidos en el Paso 3.2:

```yaml
      # =========================
      # 📊 3. SONARCLOUD
      # =========================
      - name: Run SonarCloud
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: |
          mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
          -Dsonar.projectKey=<TU_PROJECT_KEY> \
          -Dsonar.organization=<TU_ORGANIZACION> \
          -Dsonar.host.url=https://sonarcloud.io \
          -Dsonar.token=$SONAR_TOKEN \
          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

---

### Paso 5: Subir Cambios y Validar la Ejecución
1. Guarda los cambios y haz push a tu rama principal:
   ```bash
   git add pom.xml .github/workflows/main.yml
   git commit -m "ci: configurar pipeline de seguridad con snyk y sonarcloud"
   git push origin main
   ```
2. Ve a la pestaña **Actions** de tu repositorio en GitHub.
3. Observa la ejecución en tiempo real. Al finalizar satisfactoriamente verás:
   - ✅ **Snyk Security Scan** (~45s)
   - ✅ **Run Tests with JaCoCo** (~40s)
   - ✅ **SonarCloud Analysis** (~45s)
   - ✅ **Build JAR** (~30s)
4. En el resumen de la ejecución (**Summary**), en la sección inferior **Artifacts**, descarga y verifica los 3 entregables:
   - `snyk-report`
   - `jacoco-report`
   - `app-jar`

---

## 🛠️ Preguntas Frecuentes y Resolución de Problemas

| Error común | Causa principal | Solución |
| :--- | :--- | :--- |
| **`snyk auth failed (timeout)`** | El secreto `SNYK_TOKEN` no está configurado o está vacío. | Verificar que el secreto en GitHub se llame exactamente `SNYK_TOKEN` y contenga el token copiado de Snyk. |
| **`No plugin found for prefix 'sonar'`** | Falta `sonar-maven-plugin` en el archivo `pom.xml`. | Agregar el plugin en el `pom.xml` o invocar el goal completo: `org.sonarsource.scanner.maven:sonar-maven-plugin:sonar`. |
| **`You are running CI analysis while Automatic Analysis is enabled`** | SonarCloud tiene el análisis automático activo. | Ir a SonarCloud ➔ Proyecto ➔ **Administration** ➔ **Analysis Method** y apagar **Automatic Analysis**. |
| **`Project not found / Unauthorized`** | El `projectKey` u `organization` no coinciden o `SONAR_TOKEN` no tiene permisos. | Verificar que las propiedades en `main.yml` coincidan exactamente con la clave de tu proyecto en SonarCloud. |
