# Terra Samsung Library (ALPHA)

Library built on Android Studio Artic Fox with Kotlin 1.6 (jvmTarget 1.8). This package targets a minimum SDK version of 26 (Android OREO). 

This library is intended for Android Developers wishing to connect their users to Terra through Samsung (Samsung Health). To use this package, please make sure the users have downloaded [Health Platform](https://play.google.com/store/apps/details?id=com.samsung.android.service.health&hl=en_GB&gl=US) on their devices and linked their Samsung Health Account to Health Platform. This can be done by going on Samsung Health -> Profile -> Settings -> Connected Services -> Health Platform and giving Health Platform access to their data. 

## Setup
Please download the `.aar` file from this repository and include it within the `app/libs` folder in your project structure. You may now add it as a dependency in your gradle configuration files. In your project level gradle file (`build.gradle(:Project)`), edit the `repositories` to include:

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

Then you must include the library and Health Platform API dependency in your app level gradle file(`build.gradle(:app)`) by adding the following line under `dependencies`:

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

You may now import classes from the library as: `import co.tryterra.terra.*`

## Permissions 

Users will be prompted to give permissions to all the health data provided by Samsung Health. If the user does not give access to a certain datatype, the library will automatically request for access again when the data that requires that datatype is requested.

The requested permissions include: Location, Body Temperature, Respiratory Rate, Woman's Health, Heart Rate, Vo2Max, Activity, Personal Information, Sleep, Nutrition, Blood Pressure, Oxygen Saturation, and Blood Glucose.

## Usage 

The main classes you will need within this package is the `Terra` (`co.tryterra.terra.Terra`) and `HealthStoreManager` (`co.tryterra.terra.HealthStoreManager`) classes and the `HealthDataClient` (`com.google.android.libraries.healthdata.HealthDataClient`) and `HealthDataService` (`com.google.android.libraries.healthdata.HealthDataService`) classes from the Health Platform API. 

To get started, initialize a `HealthDataClient` from the Health Platform API package giving it a context:

```kotlin
private val healthDataStore: HealthDataClient by lazy {HealthDataService.getClient(this)}
```

You can then initiaite a `HealthStoreManager` by:

```kotlin
private val store: HealthStoreManager = HealthStoreManager(healthDataStore)
```
and finally a `Terra` class by:

```kotlin
private val terra: Terra = Terra("YOUR_DEV_ID", "YOUR_X_API_KEY", this, store)
```

Upon initializaion, the Terra class will automatically connect to Terra and start pushing Body, Daily and Sleep data every 8 hours, while activity data every 20 minutes (the default according to our [documentation](https://docs.tryterra.co/integrations). 

If you wish to customize this timer, you can set it by passing the parameter `bodySleepDailyTimer` and `activityTimer`in milliseconds to the Terra class upon initialization. 

## Manual Query

The class also allows for manual querying of data. 

The following functions will push data to your webhook URL from between the given startDate and endDate parameters:

### Body Data

```kotlin
terra.getBody(startDate: Date, endDate: Date)
```

### Sleep Data

```kotlin
terra.getSleep(startDate: Date, endDate: Date)
```

### Daily Data

```kotlin
terra.getDaily(startDate: Date, endDate: Date)
```

### Activity Data

```kotlin
terra.getActivity(startDate: Date, endDate: Date)
```

The following function will push athlete data to your webhook URL. (No parameters required)

### Athlete Data

```kotlin
terra.getAthlete()
```

