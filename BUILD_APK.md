# 📱 Guía: Convertir a APK con Cordova

## 🔧 Requisitos Previos

1. **Node.js y NPM** (desde https://nodejs.org)
   ```bash
   node --version
   npm --version
   ```

2. **Java Development Kit (JDK) 17+**
   ```bash
   java -version
   ```

3. **Android SDK** (desde Android Studio)
   - Descargar: https://developer.android.com/studio

4. **Variables de entorno** (Windows/Mac/Linux)
   ```bash
   ANDROID_SDK_ROOT = ruta/al/Android/sdk
   ANDROID_HOME = ruta/al/Android/sdk
   JAVA_HOME = ruta/al/jdk
   ```

---

## 🚀 Pasos para Generar APK

### 1️⃣ Instalar Cordova globalmente
```bash
npm install -g cordova
```

### 2️⃣ Clonar y navegar al repositorio
```bash
git clone https://github.com/chr9stian-droid/call-blacklist-app.git
cd call-blacklist-app
```

### 3️⃣ Instalar dependencias locales
```bash
npm install
```

### 4️⃣ Preparar proyecto Cordova
```bash
cordova prepare android
```

### 5️⃣ Compilar APK (Debug - más rápido)
```bash
cordova build android
```

**Output (archivo APK):**
```
platforms/android/app/build/outputs/apk/debug/app-debug.apk
```

---

## 📦 Generar APK Firmado (Release)

Para publicar en Google Play Store, necesitas un APK firmado:

### 1. Crear keystore
```bash
keytool -genkey -v -keystore callblock-key.keystore \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias callblock-key
```

### 2. Configurar firma en Cordova
```bash
cordova build android --release -- --keystore=callblock-key.keystore \
  --storePassword=TuContraseña --alias=callblock-key --password=TuContraseña
```

**Output:**
```
platforms/android/app/build/outputs/apk/release/app-release.apk
```

---

## 🎯 Próximos Pasos

### Para Development (Testing)
- Usa el **APK Debug** generado
- Instálalo en tu emulador o dispositivo Android

### Para Producción (Google Play Store)
- Usa el **APK Release** firmado
- Sigue: https://developer.android.com/studio/publish

### Para Instalar en tu Dispositivo
```bash
cordova run android
```
(requiere dispositivo conectado o emulador activo)

---

## ⚠️ Solución de Problemas

### Error: "ANDROID_SDK_ROOT not found"
```bash
# Windows (PowerShell)
$env:ANDROID_SDK_ROOT = "C:\Users\TuUsuario\AppData\Local\Android\sdk"

# Mac/Linux
export ANDROID_SDK_ROOT=~/Android/sdk
```

### Error: "Gradle build failed"
```bash
cordova clean
rm -rf platforms/
cordova platform add android
cordova build android
```

### APK no instala en dispositivo
```bash
adb install -r platforms/android/app/build/outputs/apk/debug/app-debug.apk
```

---

## 📊 Información del APK

| Propiedad | Valor |
|-----------|-------|
| **Nombre** | CallBlock |
| **ID** | com.callblock.app |
| **Versión** | 1.0.0 |
| **Tamaño** | ~15-20 MB |
| **Android mínimo** | 5.0 (API 21) |

---

## 🔗 Recursos Útiles

- [Documentación Cordova](https://cordova.apache.org/docs/es/latest/)
- [Android Studio](https://developer.android.com/studio)
- [Google Play Console](https://play.google.com/console)
- [Guía de Firma de APK](https://developer.android.com/studio/publish/app-signing)

---

**¿Necesitas ayuda?** Revisa los logs de error con:
```bash
cordova build android --verbose
```
