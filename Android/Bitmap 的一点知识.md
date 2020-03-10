# Bitmap 的一点知识


---

## 1. 从相册加载图片

```
/**
 * 打开手机相册
 */
private void selectFromGalley() {
    Intent intent = new Intent();
    intent.setType("image/*");
    intent.setAction(Intent.ACTION_GET_CONTENT);
    intent.addCategory(Intent.CATEGORY_OPENABLE);
    startActivityForResult(intent, REQUEST_CODE_PICK_FROM_GALLEY);
}
```

```
在onActivityResult中接收这张图片
if (resultCode == Activity.RESULT_OK) {
    Uri uri = data.getData();
    if (uri != null) {
        ProcessResult(uri);
    }
}
@TargetApi(Build.VERSION_CODES.KITKAT)
private void ProcessResult(Uri destUrl) {
    String pathName = FileHelper.stripFileProtocol(destUrl.toString());
    showBitmapInfos(pathName);
    Bitmap bitmap = BitmapFactory.decodeFile(pathName);
    if (bitmap != null) {
        mImageView.setImageBitmap(bitmap);
        float count = bitmap.getByteCount() / M_RATE;
        float all = bitmap.getAllocationByteCount() / M_RATE;
        String result = "这张图片占用内存大小:\n" +
            "bitmap.getByteCount()== " + count + "M\n" +
            "bitmap.getAllocationByteCount()= " + all + "M";
        info.setText(result);
        Log.e(TAG, result);
            bitmap = null;
    } else {
        T.showLToast(mContext, "fail");
    }
}
/**
 * 获取Bitmap的信息
 * @param pathName
 */
private void showBitmapInfos(String pathName) {
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(pathName, options);
    int width = options.outWidth;
    int height = options.outHeight;
    Log.e(TAG, "showBitmapInfos: \n" +
        "width=: " + width + "\n" +
        "height=: " + height);
    options.inJustDecodeBounds = false;
}
```

## 2. 从相机拍摄图片
```
private void openCamera() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    //创建一个临时文件夹存储拍摄的照片
    File file = FileHelper.createFileByType(mContext, destType, "test");
    imageUrl = Uri.fromFile(file);
    takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, imageUrl);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takePictureIntent, REQUEST_CODE_TAKE_PIC_CAMERA);
    }
}
```

将拍摄的图片添加到手机相册中
```
private void insertToGallery(Uri imageUrl) {
    Uri galleryUri = Uri.fromFile(new File(FileHelper.getPicutresPath(destType)));
    boolean result = FileHelper.copyResultToGalley(mContext, imageUrl, galleryUri);
    if (result) {
        Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
        mediaScanIntent.setData(galleryUri);
        sendBroadcast(mediaScanIntent);
    }
}
```

## 3. 图片的压缩

### 3.1 压缩图片方案一（Compress）
```
private Bitmap getCompressedBitmap(Bitmap bitmap) {
    try {
        //创建一个用于存储压缩后Bitmap的文件
        File compressedFile = FileHelper.createFileByType(mContext, destType, "compressed");
        Uri uri = Uri.fromFile(compressedFile);
        OutputStream os = getContentResolver().openOutputStream(uri);
        Bitmap.CompressFormat format = destType == FileHelper.JPEG ?
            Bitmap.CompressFormat.JPEG : Bitmap.CompressFormat.PNG;
        boolean success = bitmap.compress(format, compressRate, os);
        if (success) {
            T.showLToast(mContext, "success");
        }

        final String pathName = FileHelper.stripFileProtocol(uri.toString());
        showBitmapInfos(pathName);
        bitmap = BitmapFactory.decodeFile(pathName);
        os.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return bitmap;
}
```

### 3.2 压缩图片方案二 （Crop）
```
private void CropTheImage(Uri imageUrl) {
    Intent cropIntent = new Intent("com.android.camera.action.CROP");
    cropIntent.setDataAndType(imageUrl, "image/*");
    cropIntent.putExtra("cropWidth", "true");
    cropIntent.putExtra("outputX", cropTargetWidth);
    cropIntent.putExtra("outputY", cropTargetHeight);
    File copyFile = FileHelper.createFileByType(mContext, destType, String.valueOf(System.currentTimeMillis()));
    copyUrl = Uri.fromFile(copyFile);
    cropIntent.putExtra("output", copyUrl);
    startActivityForResult(cropIntent, REQUEST_CODE_CROP_PIC);
}
```

### 3.3 图片压缩方案三 （Sample ）
```
private Bitmap getRealCompressedBitmap(String pathName, int reqWidth, int reqHeight) {
    Bitmap bitmap;
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(pathName, options);
    int width = options.outWidth / 2;
    int height = options.outHeight / 2;
    int inSampleSize = 1;

    while (width / inSampleSize >= reqWidth && height / inSampleSize >= reqHeight) {
        inSampleSize = inSampleSize * 2;
    }

    options.inSampleSize = inSampleSize;
    options.inJustDecodeBounds = false;
    bitmap = BitmapFactory.decodeFile(pathName, options);
    showBitmapInfos(pathName);
    return bitmap;
}
```
### 3.4 三种方案对比
上面提到的三种压缩方案，通过对比可以发现，第一种方案适用于进行纯粹的文件压缩，而不适用进行图像处理压缩；第二种方案压缩方案适用于进行图像编辑时的压缩，就像手机自带相册的编辑功能，可以随着裁剪区域的大小进行最终的压缩；第三种方案相对来说，适应性较强，各种场景都会符合。




