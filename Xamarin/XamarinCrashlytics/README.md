# C# Xamarin Crashlytics

## Create project on https://console.firebase.google.com/

- Create firebase project if not exist
- Create 2 project with Android project and IOS project (Note: the package name and bundle ID have to correct)
- We need save 2 files google-services.json to add it in Android project(Build action: GoogleServicesJson) and GoogleService-Info.plist  (Build action: Bundle resource) to add it in IOS project.

## Config on Android app

- From nuget package install some dictionary: Xamarin.Android.Crashlytics, Xamarin.Android.Crashlytics.Answers, Xamarin.Android.Crashlytics.Beta, Xamarin.Android.Crashlytics.Core, Xamarin.Android.Fabric.
- From values -> String.xml add (<string name="com.crashlytics.android.build_id">1</string>)
- MainActivity -> OnCreate add "Fabric.Fabric.With(this, new Crashlytics.Crashlytics());
                                                  Crashlytics.Crashlytics.HandleManagedExceptions();"

## Config on IOS app

- From nuget package install some dictionary: Xamarin.Build.Download, Xamarin.Firebase.iOS.Core, Xamarin.Firebase.iOS.Analytics, Xamarin.Firebase.iOS.Crashlytics, Xamarin.Firebase.iOS.Installations, Xamarin.Firebase.iOS.InstanceID
- Add this to IOS project file 
```
<PropertyGroup>
        <FirebaseCrashlyticsUploadSymbolsEnabled>True<FirebaseCrashlyticsUploadSymbolsEnabled>
</PropertyGroup>
```
- From IOS project Option: Uncheck box (Enable build device-specific)
- Also from IOS project Option add "--dsym=true" to "target argument"