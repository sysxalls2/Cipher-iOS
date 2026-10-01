# CIPHER — Notes d’environnement & build (Phase 1)

**Date :** 2026-10-01  

## Constat initial
- Pas de `.git` à la racine ni dans `Signal-Android-main` (sources type archive)
- `git`, `java`, `adb`, Android SDK absents du PATH au démarrage de l’audit
- `winget` disponible

## Actions effectuées
- [x] Création dossier `docs/` + audits Phase 0
- [x] Installation **Eclipse Temurin JDK 21** via winget (`EclipseAdoptium.Temurin.21.JDK` 21.0.12.101)
  - `JAVA_HOME` : `C:\Program Files\Eclipse Adoptium\jdk-21.0.12.101-hotspot`
- [x] Installation **Git for Windows** 2.55.0.5 via winget
- [x] Installation **Android Platform-Tools** 37.0.1 via winget (`adb`)
- [ ] Android cmdline-tools + packages SDK (platforms, build-tools, NDK 28.0.13004108, CMake) — en cours / à finaliser
- [ ] `local.properties` (`sdk.dir`)
- [ ] Premier `assemblePlayProdDebug` réussi

## Actions restantes pour compiler
1. Finaliser SDK sous `%LOCALAPPDATA%\Android\Sdk` :
   ```bat
   sdkmanager "platforms;android-36" "platforms;android-37" "build-tools;36.0.0" "ndk;28.0.13004108" "cmake;3.22.1" "platform-tools"
   ```
2. Créer `Signal-Android-main\local.properties` :
   ```
   sdk.dir=C:\\Users\\<user>\\AppData\\Local\\Android\\Sdk
   ```
3. Depuis `Signal-Android-main` :
   ```bat
   set JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-21.0.12.101-hotspot
   gradlew.bat :Signal-Android:assemblePlayProdDebug
   ```

## Note
La compilation complète n’est **pas encore validée** tant que NDK/platforms ne sont pas installés.  
Aucun code applicatif source n’a été modifié pour le branding ou la privacy (Phase 0 docs only).
