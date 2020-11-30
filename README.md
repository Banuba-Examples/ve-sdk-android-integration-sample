# Banuba Video Editor SDK. Integration sample for Android
Welcome to the [Banuba VE SDK](https://www.banuba.com/video-editor-sdk). Here you will find how to integrate VE into your product and take all  advantages of it.


## Requirements
- Java 1.8+
- Kotlin 1.4+
- Android Studio 4+
- Android OS 6.0 or higher with Camera 2 API
- OpenGL ES 3.0 (3.1 for Neural networks on GPU)  

## Dependencies
- [Koin](https://insert-koin.io/)
- [ExoPlayer](https://github.com/google/ExoPlayer)
- [Glide](https://github.com/bumptech/glide)
- [Kotlin Coroutines](https://github.com/Kotlin/kotlinx.coroutines)
- [ffmpeg](https://github.com/FFmpeg/FFmpeg/tree/n3.4.1)
- [AndroidX](https://developer.android.com/jetpack/androidx/versions) libraries
- [Banuba Face AR SDK](https://www.banuba.com/facear-sdk/face-filters). *Optional*

## Free Trial
We provide a 14-days free trial period. Here is how to get it:  
1. [Contact Us](https://www.banuba.com/video-editor-sdk#form)
1. Clone this repository
1. Request a [token](##Token) from our sales team
1. Put the token into the app
1. Start the sample
1. Follow the [integration guide](##Getting-Started) and customize the app to have your branded user experience.

## Token  
Banuba uses tokens for [Face AR SDK](https://www.banuba.com/facear-sdk/face-filters) to manage features and protect the technology. SDK usage requires up to date tokens (trial or commercial one). When tokens expires, the SDK features became not available.
The token should be put [here](app/src/main/res/values/strings.xml#L5).


## Getting Started
### Setup GitHub packages
GitHub packages are used to download the latest SDK modules. You would need them also to receive a new SDK versions.

Please add [Maven repository](build.gradle#L23) for GitHub
``` kotlin
allprojects {
    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/Banuba/banuba-ve-sdk")
            credentials {
                username = "Banuba"
                password = "put your new personal access token here"
            }
        }
        ...
    }
}
```  
### Add dependencies
Please specify list of dependencies as in [app/build.gradle](app/build.gradle#L38) file to integrate Banuba Video Editor SDK.

### Add Activity
Banuba Video Editor contains a specific flow and order of screens: camera, gallery, trimmer, editor and export. Each screen is implemented as a [Fragment](https://developer.android.com/jetpack/androidx/releases/fragment?authuser=1). All Fragmens are handled with [Activity](https://developer.android.com/jetpack/androidx/releases/activity?hl=en&authuser=1) - VideoCreationActivity. Add VideoCreationActivity to [AndroidManifest.xml](app/src/main/AndroidManifest.xml#L21)
``` xml
<activity android:name="com.banuba.sdk.ve.flow.VideoCreationActivity"
    android:screenOrientation="portrait"
    android:theme="@style/CustomIntegrationAppTheme"
    android:windowSoftInputMode="adjustResize"
    tools:replace="android:theme" />
```
It will allow to [launch Video Editor](app/src/main/java/com/banuba/example/integrationapp/MainActivity.kt#L24).  

[CustomIntegrationAppTheme](app/src/main/res/values/themes.xml#L14) theme overrides icons, colors, etc. for VE SDK screens. Use it to brand your app.

### Add config files  
Banuba VE SDK has several configuration files that allows to customize video editor for your needs. All config files should be placed into Android **assets** folder:  
- [camera.json](app/src/main/assets/camera.json) contains properties that you can customize on the camera screen i.e. the minimun and maximum video durations, turn flashlight feature on or off.
Usually, *minVideoDuration* and *maxVideoDuration* are most used properties.
- [music_editor.json](app/src/main/assets/music_editor.json) contains properties that you can customize on the audio editor screen i.e. how many timelines or tracks are allowed.
- [object_editor.json](app/src/main/assets/object_editor.json) contains properties that you can customize on the editor screen.
- [videoeditor.json](app/src/main/assets/videoeditor.json) contains properties that you can customize on editor, trimmer and gallery screens.  *Note*: please keep in mind that *minVideoDuration* and *maxVideoDuration* in this and [camera.json](app/src/main/assets/camera.json) should be the same.

### Configure DI  
VideoEditor behavior can be overridden. We use [Koin](https://insert-koin.io/) for this purpose.
Firstly, you need to create your own implementation of FlowEditorModule.  
``` kotlin
class VideoEditorKoinModule : FlowEditorModule() {

    override val effectPlayerManager: BeanDefinition<AREffectPlayerProvider> =
        single(override = true) {
            BanubaAREffectPlayerProvider(
                mediaSizeProvider = get(),
                token = androidContext().getString(R.string.video_editor_token)
            )
        }

    ...
}
```  
You will need to override several properties to customize video editor for your application.
Full example you can take a look [here](app/src/main/java/com/banuba/example/integrationapp/videoeditor/di/VideoEditorKoinModule.kt).

Secondly, you need to initialize Koin module in your [Application.onCreate](app/main/src/java/com/banuba/example/integrationapp/IntegrationApp.kt#L12) method.  
``` kotlin
startKoin {
    androidContext(this@IntegrationApp)        
    modules(VideoEditorKoinModule().module)
}
```

### Configure and start SDK in Android Java project
You can use Java in your Android project. In this case you can start Koin in this way
``` java
 startKoin(new GlobalContext(), koinApplication -> {
            androidContext(koinApplication, this);
            koinApplication.modules(new VideoeditorKoinModuleKotlin().getModule());
            return null;
        });
```
Full example of Java Application class you can find [here](app/src/main/java/com/banuba/example/integrationapp/videoeditor/IntegrationJavaApp.kt#17).


### Configure export flow  
Override [exportDir](app/src/main/java/com/banuba/example/integrationapp/videoeditor/di/VideoeditorKoinModule.kt#84) to specify where to store export targets - video, audio files and metadata.

Override [exportParamsProvider](app/src/main/java/com/banuba/example/integrationapp/videoeditor/di/VideoeditorKoinModule.kt#72) to specify export targets i.e. video and audio files.
Please see our [example](app/src/main/java/com/banuba/example/integrationapp/videoeditor/export/IntegrationAppExportParamsProvider.kt).

Override  [exportFlowManager](app/src/main/java/com/banuba/example/integrationapp/videoeditor/di/VideoEditorKoinModule.kt#56) in VideoEditorKoinModule to customize export flow. For instance, you can specify should export be performed in background or foreground. Please see our [example](app/src/main/java/com/banuba/example/integrationapp/videoeditor/export/IntegrationAppExportFlowManager.kt).

Override [exportResultHandler](app/src/main/java/com/banuba/example/integrationapp/videoeditor/di/VideoeditorKoinModule.kt#65) to handle an export result. Please see our [example](app/src/main/java/com/banuba/example/integrationapp/videoeditor/export/IntegrationAppExportResultHandler.kt)

### Configure watermark
One of the VE features is a watermark. You can add your branded image on top of the video, which user exports.

To utilize the watermark, add ``` WatermarkProvider``` interface to your app. 
Add watermark image in the method ```getWatermarkBitmap```. Finally, re-arrange the dependency ```watermarkProvider``` in [DI](app/src/main/java/com/banuba/example/integrationapp/videoeditor/di/VideoEditorKoinModule.kt#70). Check out [this example](app/src/main/java/com/banuba/example/integrationapp/videoeditor/impl/IntegrationAppWatermarkProvider.kt) if you have any troubles.

### Configure audio content

Video editor is able to work with audio files to create even more attractive video recordings. Video editor SDK does not provide audio files on its own but it has a convenient way to setup your internal or external audio files provider to apply audio content seamlessly for user.

Check out [step-by-step guide](mddocs/audio_content.md) to add your audio content into the SDK.

### Configure stickers content

Stickers are interactive objects (gif images) that can be added to the video recording to add more fun for users. 

By default [**Giphy API**](https://developers.giphy.com/docs/api/) is used to load stickers. All you need to start is just to pass your personal Giphy Api Key into **stickersApiKey** parameter in [videoeditor.json](app/src/main/assets/videoeditor.json) file.

### Add post processing effects
There are several effects in Banuba VE SDK: visual, time and mask. In order to add a visual effect you would need to add a class followed by type, name and the icon of the effect. [Example](app/src/main/java/com/banuba/example/integrationapp/videoeditor/data/VisualEffects.kt).

Same for [Time effects](app/src/main/java/com/banuba/example/integrationapp/videoeditor/data/TimeEffects.kt) and [Masks](app/src/main/java/com/banuba/example/integrationapp/videoeditor/data/MaskEffects.kt).

Finally, override the dependency [editorEffects](app/src/main/java/com/banuba/example/integrationapp/videoeditor/di/VideoEditorKoinModule.kt#74) and choose the effects you wannt to use.

### Configure record button

Record button is a main control on the camera screen that can be fully customized along with animations playing on tap. There are 3 steps to create record button:

1. Create custom view for the record button. [Example](app/src/main/java/com/banuba/example/integrationapp/videoeditor/widget/recordbutton/RecordButtonView.kt).

2. Implement ```CameraRecordingAnimationProvider``` interface. Here the view created in step 1 should be provided through method ```provideView()``` within this interface. [Example](app/src/main/java/com/banuba/example/integrationapp/videoeditor/impl/IntegrationAppRecordingAnimationProvider.kt). 

3. Provide ```CameraRecordingAnimationProvider``` implementation in [DI](app/src/main/java/com/banuba/example/integrationapp/videoeditor/di/VideoEditorKoinModule.kt#140).


### Configure screens  
The SDK allows to override icons, colors, typefaces and other things using Android theme and styles. Every SDK screen has its own set of styles.  
Below you can find how to customize each VE step to bring your experience:
1. [Camera screen](mddocs/camera_styles.md)
1. [Editor screen](mddocs/editor_styles.md)
1. [Gallery screen](mddocs/gallery_styles.md)
1. [Trimmer screen](mddocs/trimmer_styles.md)
1. [Music Editor screen](mddocs/music_editor_styles.md)
1. [Timeline Editor screen](mddocs/timeline_editor_styles.md)
1. [Cover screen](mddocs/cover_styles.md)
1. [Alert Dialogs](mddocs/alert_styles.md)


