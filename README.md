# Save-Image-Using-MediaStoreAPI-with-required-Permissions-Android-Kotlin

#Permissions
```
<uses-permission
        android:name="android.permission.WRITE_EXTERNAL_STORAGE"
        tools:ignore="ScopedStorage" />
```
#OnButtonClicked
```
 binding.btSaveImage.setOnClickListener {
            checkWriteStoragePermission()
        }
```
#checkWriteStoragePermission
```
    private fun checkWriteStoragePermission() {
        if (ContextCompat.checkSelfPermission(this,Manifest.permission.WRITE_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED ||
            Build.VERSION.SDK_INT>= Build.VERSION_CODES.Q) {
            saveToPictures()
        } else requestStoragePermissionLauncher.launch(Manifest.permission.WRITE_EXTERNAL_STORAGE)
    }
```
#requestStoragePermissionLauncher
```
 private val requestStoragePermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted: Boolean ->
        if (isGranted) {
            saveToPictures()
        } else {
            Utils.toast(this, "Storage permission denied")
        }
    }
```
#saveToPictures
```
    private fun saveToPictures() {
        if (savePhotoToExternalStorage(viewModel.imageName,viewModel.imageBitmap)) 
            Utils.toasty(this, "Saved to Gallery")
        else {
            Utils.toast(this, "Unable to save image")
        }
    }
```
#savePhotoToExternalStorage
```
  private fun savePhotoToExternalStorage(displayName: String, bmp: Bitmap): Boolean {
        val imageCollection = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            MediaStore.Images.Media.getContentUri(MediaStore.VOLUME_EXTERNAL_PRIMARY)
        } else {
            MediaStore.Images.Media.EXTERNAL_CONTENT_URI
        }
        val contentValues = ContentValues().apply {
            put(MediaStore.Images.Media.DISPLAY_NAME, "$displayName.jpg")
            put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
            put(MediaStore.Images.Media.WIDTH, bmp.width)
            put(MediaStore.Images.Media.HEIGHT, bmp.height)
        }
        return try {
            contentResolver.insert(imageCollection, contentValues)?.also { uri ->
                Log.e("file path", File(uri.path.toString()).absolutePath)
                contentResolver.openOutputStream(uri).use { outputStream ->
                    if (!bmp.compress(Bitmap.CompressFormat.JPEG, 100, outputStream)) {
                        throw IOException("Couldn't save bitmap")
                    }
                }
            } ?: throw IOException("Couldn't create MediaStore entry")
            true
        } catch (e: IOException) {
            e.printStackTrace()
            false
        }
    }

```
