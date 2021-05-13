# C# Appcenter Xamarin UI Test

## Config Test

How to use App center xamarin ui test

1. Login at https://appcenter.ms/
2. Settup in app https://appcenter.ms/ -> overview
3. Create new Test     
    3.1 Create group devices
    3.2 Config test
4. Using NodeJS
    4.1 Run "npm install -g appcenter-cli"
    
    4.2 Login App center "appcenter login" -> verify code

    4.3 Upload and schedule tests

        - appcenter test run appium --app "USERNAME/TESTNAME" --devices "USERNAME/DEVICESETNAME" --app-path pathToFile.apk --test-series "master" --locale "en_US" --build-dir target/upload
        
        - Example: appcenter test run uitest --app "swd_mobile/siteTRAX-Air-2" --devices "swd_mobile/samsungtest" --app-path "C:\Users\vu.hoangtuan\Desktop\XamarinUITest\com.ucg.sitetraxair.apk"  --test-series "master" --locale "en_US" --build-dir "C:\Users\vu.hoangtuan\Desktop\XamarinUITest" --uitest-tools-dir "C:\NCC Files\Projects\UCG_NetWork\Hung\siteTRAX.Xamarin\UCG.SiteTRAXUITest\bin\Debug"
        
5. Access to Test -> Test run to view test
