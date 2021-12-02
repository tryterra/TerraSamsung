# Terra Samsung Library

Library built on Android Studio Artic Fox with Kotlin 1.6 (jvmTarget 1.8). This package targets a minimum Sdk version of 26. 

This library is intended for Android Developers wishing to connect their users to Terra through Samsung (Samsung Health). To use this package, please make sure the users have downloaded [Health Platform](https://play.google.com/store/apps/details?id=com.samsung.android.service.health&hl=en_GB&gl=US) on their devices and linked their Samsung Health Account to Health Platform. This can be done by going on Samsung Health -> Profile -> Settings -> Connected Services -> Health Platform and giving Health Platform access to their data. 

## Setup
Please download the `.aar` file from this repository and include it within the `app/libs` folder in your project structure. You may now add it as a dependency in your gradle. In your project level gradle (`build.gradle(:Project)`), edit the `repositories` to include:

```gradle
flatDir{
  dirs 'libs'
}
```

For example:
```gradle
repositories {
    google()
    mavenCentral()
    flatDir {
        dirs 'libs'
    }
}
```

Then you must include the library and Health Platform API dependency in your app level gradle (`build.gradle(:app)`) by adding the following line under `dependencies`:

```gradle
implementation files('libs/TerraSamsung-alpha.aar')
implementation 'com.google.android.libraries.healthdata:health-data-api:1.0.0-alpha01'

```
For example:
```gradle
dependencies {
    implementation 'androidx.core:core-ktx:1.7.0'
    implementation 'com.google.android.libraries.healthdata:health-data-api:1.0.0-alpha01'
    implementation 'androidx.appcompat:appcompat:1.4.0'
    implementation files('libs/TerraSamsung-alpha.aar')
}
```

You may now import classes from the library as: `import com.example.terra.*`

## Permissions 

Users will be prompted to give permissions to all the health data provided by Samsung Health. If the user does not give access to a certain datatype, the library will automatically request for access again when the data that requires that datatype is requested.

## Usage 

The main class you will need within this package is the `Terra` (`com.example.terra.Terra`) and `HealthStoreManager` (`com.example.terra.HealthStoreManager`) classes and the `HealthDataClient` (`com.google.android.libraries.healthdata.HealthDataClient`) and `HealthDataService` (`com.google.android.libraries.healthdata.HealthDataService`) classes from the Health Platform API. 

To get started, initialie a `HealthDataClient`:

```kotlin
private val healthDataStore: HealthDataClient by lazy {HealthDataService.getClient(this)}
```


