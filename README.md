# Terra Samsung Library (BETA)

Library built on Android Studio Artic Fox with Kotlin 1.6 (jvmTarget 1.8). This package targets a minimum SDK version of 26 (Android OREO). 

This library is intended for Android Developers wishing to connect their users to Terra through Samsung (Samsung Health). To use this package, please make sure the users have downloaded [Health Platform](https://play.google.com/store/apps/details?id=com.samsung.android.service.health&hl=en_GB&gl=US) on their devices and linked their Samsung Health Account to Health Platform. This can be done by going on Samsung Health -> Profile -> Settings -> Connected Services -> Health Platform and giving Health Platform access to their data. 

## Demo Application
You can find a DEMO application [here](https://github.com/tryterra/TerraSamsungDemo). Please note it is meant for demo purposes only, and you may reuse any code you wish on this demo. 

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
private val terra: Terra = Terra("YOUR_DEV_ID", "YOUR_X_API_KEY", this, store, "A REFERENCE ID")
```

The intiantiation or this class will throw an error if there is a connection error to our server, or if Health Platform service is not available on the device. Please catch this error and handle it properly on your application (Example application shows this).

The reference ID is not a must pass, however if you wish to pass it, you can use it to associate users from your backend to the Terra user ID. You can access either of these properties as:

```kotlin
val user_id = terra.userId
val reference_id = terra.referenceId
```

Upon initializaion, the Terra class will automatically connect to Terra and start pushing Body, Daily and Sleep data every 8 hours, while activity data every 20 minutes (the default according to our [documentation](https://docs.tryterra.co/integrations). 

If you wish to customize this timer, you can set it by passing the parameters `bodyTimer`, `nutritionTimer`, `dailyTimer`, `sleepTimer`, and `activityTimer`in milliseconds to the Terra class upon initialization. **Make sure these values are divisible by 60000 (1 minute)**. Please refer to the demo application for example. 

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

### Nutrition Data

```kotlin
terra.getNutrition(startDate: Date, endDate: Date)
```


The following function will push athlete data to your webhook URL. (No parameters required)

### Athlete Data

```kotlin
terra.getAthlete()
```

## Deauthenticate

To deauthenticate users, you may run the following function:

```kotlin
terra.disconnect()
```

## Terra Endpoints

This package also allows you to call the "authenticateUser" and "deauthenticateUser" endpoint along with the data request endpoints provided by [Terra API](https://docs.tryterra.co). 

### Authenticate User and Deauthenticate User

The `TerraAuth` class deals with these two endpoints. First you have to initiate one:

```kotlin
val terraAuthClient: TerraAuth = TerraAuth("YOUR X API KEY", "YOUR DEV ID")
```

and then call the following function:
```kotlin
terraAuthClient.authUser(RESOURCE){it ->
    //Do something with the response
        Log.i(TAG, it.toString())
    }
```
Call back returns an `AuthenticateUser` datamodel with the following fields:

```kotlin
data class AuthenticateUser(
    var status: String,
    var auth_url: String,
    var user_id: String,
)
```

Similarly for deauthentication:

```kotlin
terraAuthClient.deauthenticateUser(USER_ID)
```

### Data Endpoints

To use the Terra API Data endpoints, you may do so with the `TerraClient` class:

```kotlin
val terraClient = TerraClient("USER ID", "YOUR X API KEY", "YOUR DEV ID")
```

Using this class, you can then request for data. The following example requests Body Data from our API with startDate and endDate parameters set as UNIX Timestamps in seconds. You may set "toWebhook" to false if you wish for the callback function to return the data payload.

```kotlin
    terra.getBody(
        startDate = Date.from(Instant.ofEpochMilli(startTime)).toInstant().epochSecond,
        endDate = Date.from(Instant.ofEpochMilli(endTime)).toInstant().epochSecond,
        toWebhook = false
    ){bodyData ->
        // Do something with body data
        Log.i(TAG, bodyData.toString())
    }
```

The other data functions would be : `getDaily`, `getActivity`, `getSleep`, `getNutrition`. These functions uses the same argument as the `getBody` function and all of them returns a callback with their own datamodels. The `toWebhook` argument defaults to `true` if not provided.

The data models correspond to having fields (exactly) the same as we the ones given by https://docs.tryterra.co/data-models

Finally theres also `getAthlete` for which only accepts `toWebhook` argument.
