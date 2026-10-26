# android-pipeline
 
An Android sample app wired to an Azure Pipelines CI/CD definition. On every push to `main` the pipeline builds a debug APK with Gradle, signs it, publishes it as a build artifact and sends it to a distribution group.
 
The app itself is the MVP to-do sample from the Android Architecture Blueprints (`com.example.android.architecture.blueprints.todomvp`). The point of the repo is the pipeline, not the app.
 
## Repository layout
 
| Path | Purpose |
|------|---------|
| `azure-pipelines.yml` | Azure Pipelines definition |
| `app/` | Android application module |
| `build.gradle`, `settings.gradle`, `gradle.properties` | Root Gradle configuration |
| `gradlew`, `gradlew.bat`, `gradle/wrapper` | Gradle wrapper |
| `.vscode/` | Editor settings |
| `LICENSE`, `LICENSE-CODE` | Documentation licence (CC BY 4.0) and code licence (MIT) |
 
## Pipeline
 
Defined in `azure-pipelines.yml`, triggered by commits to `main`.
 
| Step | Task | What it does |
|------|------|--------------|
| 1. Build | `Gradle@2` | Runs `clean assembleDebug` with the wrapper on JDK 11, heap set to 3 GB, and publishes JUnit results from `**/TEST-*.xml` |
| 2. Sign | `AndroidSigning@3` | Signs every APK with `android-keystore.jks` and zipaligns it |
| 3. Stage | `CopyFiles@2` | Copies APKs to the artifact staging directory |
| 4. Publish | `PublishBuildArtifacts@1` | Publishes the staged files as a build artifact |
| 5. Distribute | `AppCenterDistribute@3` | Uploads `app/build/outputs/apk/debug/*.apk` to the `agangwar/android-release` app using the `orgappcenter` service connection, with `CHANGELOG` as release notes |
 
The job runs on a self-hosted agent: pool `test`, agent name `mywindowsagent`.
 
### Required pipeline setup
 
Signing credentials are not stored in the repo. Create these as secret pipeline variables:
 
| Variable | Used as |
|----------|---------|
| `keystorepassword` | Keystore password |
| `keystorealias` | Key alias |
| `keypassword` | Key password |
 
The keystore file `android-keystore.jks` is not committed either. Provide it to the agent, for example as an Azure DevOps secure file, and make sure the signing step can find it.
 
You also need:
 
- A self-hosted agent named `mywindowsagent` in the `test` pool, with JDK 11 and the Android SDK
- A service connection named `orgappcenter`
## Build locally
 
```bash
./gradlew clean assembleDebug
```
 
The APK is written to `app/build/outputs/apk/debug/`.
 
## Important: App Center has been retired
 
The distribution step relies on Visual Studio App Center, which Microsoft retired on March 31, 2025 (only the Analytics and Diagnostics features were kept for a while longer). Step 5 will fail. Microsoft's migration guidance points to Azure Pipelines for builds and other tools for distribution, for example Firebase App Distribution or Google Play internal testing. Replace the `AppCenterDistribute@3` task with one of those.
 
## Other notes
 
- The Gradle setup uses the old Android Support Library (`com.android.support:*`) and `AndroidJUnitRunner` from `android.support.test`. Building with a current Android Gradle Plugin means migrating to AndroidX.
- The pipeline builds and distributes a debug build. A release build needs `assembleRelease` and a matching change to the APK path in the distribute step.
## Contributing
 
This project welcomes contributions and suggestions. Most contributions require agreeing to a Contributor License Agreement (CLA). When you open a pull request, a bot will tell you whether one is needed. The project follows the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
 
## Legal notices
 
Microsoft and any contributors grant a licence to the documentation and other content in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode) (see `LICENSE`), and a licence to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT) (see `LICENSE-CODE`).
 
Microsoft, Windows, Microsoft Azure and other Microsoft products and services referenced here may be trademarks of Microsoft. The licences do not grant any right to use Microsoft names, logos or trademarks. See Microsoft's [trademark guidelines](http://go.microsoft.com/fwlink/?LinkID=254653) and [privacy statement](https://privacy.microsoft.com/en-us/).
