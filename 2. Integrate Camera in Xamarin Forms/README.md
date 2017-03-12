# Integrating Camera in Xamarin Forms

## 1. Installing the Nuget Package

* Firstly search for and install ```Xam.Plugin.Media``` in all your projects.

## 2. Permissions

For Android, since API 23, much like on iOS, if you want to access the camera you have to request permissions during runtime.

* Expand your Android project and open 'MainActivity.cs'
* Add the following lines of code

```
public override void OnRequestPermissionsResult(int requestCode, string[] permissions, Android.Content.PM.Permission[] grantResults)
{
	PermissionsImplementation.Current.OnRequestPermissionsResult(requestCode, permissions, grantResults);
}
```

* If your application is targetting Android N (API 24) then you must add the following lines to your manifest file

```
<provider android:name="android.support.v4.content.FileProvider" 
                android:authorities="YOUR_APP_PACKAGE_NAME.fileprovider" 
                android:exported="false" 
                android:grantUriPermissions="true">
            <meta-data android:name="android.support.FILE_PROVIDER_PATHS" 
                android:resource="@xml/file_paths"></meta-data>
</provider>
```

* Inside the resouce file within your Android project, create a new folder called 'xml' and inside that folder, create a new .xml file called 'file_paths.xml'
* Add the following lines of code to that file.

```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="Android/data/YOUR_APP_PACKAGE_NAME/files/Pictures" />
    <external-path name="my_movies" path="Android/data/YOUR_APP_PACKAGE_NAME/files/Movies" />
</paths>
```

For iOS, open info.plst and add the following lines to that file.

```
<key>NSCameraUsageDescription</key>
<string>This app needs access to the camera to take photos.</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>This app needs access to photos.</string>
<key>NSMicrophoneUsageDescription</key>
<string>This app needs access to microphone.</string>
```

## 3. Taking photos

* First we'll modify the layout of our main page.
* Add the following lines of code

```
<StackLayout BackgroundColor="White">
	<Button Text="Take Picture" TextColor="White" BackgroundColor="Green" Clicked="TakePicture_Clicked" />
	<Image x:Name="image" VerticalOptions="Start" />
</StackLayout>
```

* Navigate to your portable project and inside MainPage.xaml.cs, after creating our button to trigger the camera, we need to now add the 'TakePicture_Clicked' event method that's called when out button is tapped. Add the following method under the constructor method.

```
private async void TakePicture_Clicked(object sender, System.EventArgs e) 
{

}
```

* Now that we have our 'TakePicture_Clicked' event handler, we now need to add code that's going to run whenever the user clicks on our button. First thing we need to do before trying to take a photo is to check and see if 1. the user's device has a camera and it's available to take photos and 2. the device supports taking photos. To do this, add the following lines of code in our 'TakePicture_Clicked' event handler.

```
if (!CrossMedia.Current.IsCameraAvailable || !CrossMedia.Current.IsTakePhotoSupported)
{
	await DisplayAlert("No Camera", ":( No camera avaialble.", "OK");
	return;
}
```

* To take a photo add the following lines of code

```
 var file = await CrossMedia.Current.TakePhotoAsync(new Plugin.Media.Abstractions.StoreCameraMediaOptions
 {
	DefaultCamera = Plugin.Media.Abstractions.CameraDevice.Front,
	Directory = "Moodify",
	Name = $"{DateTime.UtcNow}.jpg",
	CompressionQuality = 92
});

if (file == null)
	return;
```

* To diplay the image add the following lines of code.

```
image.Source = ImageSource.FromStream(() =>
{
	var stream = file.GetStream();
	file.Dispose();
	return stream;
});
```