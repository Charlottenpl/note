## Android 裁切图片功能

1. 动态申请权限

   ---

   申请文件的读写权限

   ----

2. Android 7以上适配FileProvider

   Android 7 以后，Android对文件的保护变得十分严格，应用只有在调用自己的私有目录时才不需要权限，其他目录都需要动态申请权限。并且uri的创建不能再使用`uri.fromFile();`的方法，需要使用FileProvider的静态方法进行创建。

   `Uri out = FileProvider.getUriForFile()。`

   

3. 从系统图库选择图片

   ```java
   Intent intent = new Intent(Intent.ACTION_PICK,null);
   intent.setDataAndType(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,"image/*");
   activity.startActivityForResult(intent, BasicSDK.REQUEST_CUT_IMAGE_CAPTURE);
   ```

4. 调用系统CROP方法进行裁切

   android系统有图片裁切的方法，需要通过intent进行调用。

   ```java
   Intent intent = new Intent("com.android.camera.action.CROP");
   intent.setDataAndType(uri, "image/*");//设置要处理的图片资源
   intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
   intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION);// 这两句是在7.0以上版本
   intent.putExtra("crop", "true");
   intent.putExtra("aspectX", ox);//裁切框的比例
   intent.putExtra("aspectY", oy);
   intent.putExtra("outputX", x);//输出图片的宽高像素数
   intent.putExtra("outputY", y);
   intent.putExtra("scale", true);//裁剪时是否保留宽高比
   ```

5. 两种方法返回数据

   > 两种方法的区别：通过return-data返回数据的话，返回的数据有大小限制，貌似是不大于1M的bitmap，小尺寸图片的话用return-data就可以，转换成jpg大概180宽高最大，超过之后就会被压缩到180左右，而使用MediaStore.EXTRA_OUTPUT返回数据的话不会有这个限制，适合对大图片进行裁切的时候使用，保存的是全尺寸图像。

   1. 通过return-data返回数据

      通过return-data返回数据时需要将return-data设置为true，之后裁切后的数据会通过intent返回到回调函数中。

      ``` java
      intent.putExtra("return-data", true);
      intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());//输出格式
      intent.putExtra("noFaceDetection", true); // 取消人脸识别
      intent = Intent.createChooser(intent, "裁剪图片");
      activity.startActivityForResult(intent, BasicSDK.REQUEST_CUT_IMAGE);
      ```

   2. 通过MediaStore.EXTRA_OUTPUT返回数据

      将return-data设置为false之后，裁切到的图片会保存到自定义的地址。

      ``` java
      intent.putExtra("return-data", false);
      Uri out = FileProvider.getUriForFile(activity,"com.basicsdk.sdkdemo.provider", new File(getSave_file()));//这里不能用这种方法创建uri，否则会报错，不理解
      String save_file = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES).getPath()+"/save_"+UUID.randomUUID().toString()+".jpg";//自定义存储地址，这里需要将文件的保存地址定义到公共目录中，不能放到应用的私有目录，否则会保存不了，因为一般情况下只有应用本身能访问自己的私有目录。
      setSave_file(save_file);//忽略
      intent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(new File(save_file)));//存储地址
      intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
      intent.putExtra("noFaceDetection", true); // 取消人脸识别
      intent = Intent.createChooser(intent, "裁剪图片");
      activity.startActivityForResult(intent, BasicSDK.REQUEST_CUT_IMAGE);
      ```

      

6. 图片回显

   同样不能使用`uri.fromFile();`的方法获取图片，需要改成这种`Uri out = FileProvider.getUriForFile()`

   ``` java
   Uri out =FileProvider.getUriForFile(getApplicationContext(),"com.basicsdk.sdkdemo.provider", new File(photoPath));
   imageView.setImageURI(out);
   ```

   

