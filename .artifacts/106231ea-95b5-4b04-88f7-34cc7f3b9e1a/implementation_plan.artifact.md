# Implement MainActivity and Jetpack Compose Entry Point

Add a `MainActivity.kt` file with a Jetpack Compose UI, enable Compose in `build.gradle.kts`, add cached Compose dependencies (`activity-compose`, `compose.ui`, `compose.material3`), and update `AndroidManifest.xml` to launch `MainActivity`.

## User Review Required

> [!IMPORTANT]
> This plan configures Jetpack Compose (`compileSdk 36`, Compose UI `1.10.4`, Activity Compose `1.13.0`) and creates `MainActivity.kt`.

## Proposed Changes

### Configuration and Dependencies

#### [MODIFY] [libs.versions.toml](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/gradle/libs.versions.toml)
- Add Compose version (`compose = "1.10.4"`, `activityCompose = "1.13.0"`) and libraries (`androidx-activity-compose`, `androidx-compose-ui`, `androidx-compose-material3`).

#### [MODIFY] [build.gradle.kts](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/build.gradle.kts)
- Enable `buildFeatures { compose = true }`.
- Add Compose dependencies.

### Application Entry Point

#### [MODIFY] [AndroidManifest.xml](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/AndroidManifest.xml)
- Register `MainActivity` with a MAIN action and LAUNCHER category.

#### [NEW] [MainActivity.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/composearchlab/MainActivity.kt)
- Create `MainActivity` extending `ComponentActivity` hosting a simple Jetpack Compose screen.

## Verification Plan

### Automated Tests
- Run gradle build (`app:assembleDebug`) to verify compilation and resource/dependency resolution.
