# 《ReactNative系列讲义》高级篇---07.热更新全量更新主体代码实现

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介
上一篇文章中我们已经大概的了解到了热更新的作用，实现思路与技术节点。既包含了全量更新也包含了差量更新，不过仅限于Android平台。我们分步前进，先做全量更新，在做增量更新；先做代码的增量更新，在做图片的增量更新。下面我们一起看看具体的实现代码如何编写。

#### 二、代码实现
##### 1. 服务器端的准备
我使用的是免费的云后台服务Bmob，先创建一个表，定义你需要的字段，操作说明可查看Bmob的官方网站。
![屏幕快照 2018-03-02 下午2.43.42](media/15199700699722/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-03-02%20%E4%B8%8B%E5%8D%882.43.42.png)

##### 2. 定义APP本地bundle版本
使用Android.BuildConfig功能定义bundle文件的版本号，在app/build.gradle文件里增加自定义的BuildConfig常量。

* defaultConfig里面添加buildConfigField选项，参数：字段类型，名称，内容
![屏幕快照 2018-03-02 下午2.57.12](media/15199700699722/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-03-02%20%E4%B8%8B%E5%8D%882.57.12.png)

Android的BuildConfig操作（参考：http://blog.csdn.net/aotian16/article/details/51776051）

##### 3. 创建android native module
创建android native module，名称定义为VersionUpdateCheckModule，继承ReactContextBaseJavaModule类。

* 自定义native module的方法可参照官网http://facebook.github.io/react-native/docs/native-modules-android.html

##### 4. 创建getConstants()方法，获取当前bundle版本号
* 将从服务器获取到的bundle版本号存入SharedPreferences

```
private static final String BUNDLE_VERSION = "CurrentBundleVersion";

// 构造方法中实现
SharedPreferences mSP = reactContext.getSharedPreferences("react_bundle", Context.MODE_PRIVATE);

@Nullable
@Override
public Map<String, Object> getConstants() {
   Map<String, Object> constants = MapBuilder.newHashMap();
   // 定义在app/build.gradle文件中的bundle基础版本号
   String bundleVersion = BuildConfig.BUNDLE_VERSION;
   // 获取保存在SP中的版本号，该版本号是从服务器端获取并存储的。
   String cacheBundleVersion = mSP.getString(BUNDLE_VERSION, "");
   // 如果为空，说明是第一次进入程序
   if(!TextUtils.isEmpty(cacheBundleVersion)) {
       bundleVersion = cacheBundleVersion;
   }
   // 将获取到的版本号存入Map中
   constants.put(BUNDLE_VERSION, bundleVersion);

   return constants;
} 
```

##### 5. 创建版本检测主方法伪代码

```
@ReactMethod
public void check() {
    // TODO 获取SP中存储的最新bundle文件版本号
    // TODO 获取服务器端bundle版本号
    // TODO 如果不需要更新，不做任何逻辑处理
    // TODO 需要更新版本，先下载bundle更新包
    // TODO 定义文件存放根目录
    // TODO 如果路经存在，删除该目录下所有文件；如果不存在，创建根目录
    // TODO 下载bundle压缩文件
    // TODO 解压缩bundle文件到文件根目录下
    // TODO 解压缩成功后保存从服务器端获取到的当前bundle版本号到SP中
}
```

##### 6. 创建版本检测主方法

```
@ReactMethod
public void check() {
    // 1. 获取SP中存储的最新bundle文件版本号
    Map<String, Object> maps = getConstants();
    String currentVersion = (String) maps.get(BUNDLE_VERSION);
    // 2. 获取服务器端bundle版本号（后续有讲解）；Bmob云后台提供的API
    BmobQuery<AppVersions> query = new BmobQuery<>();
    query.setLimit(1);
    query.addWhereGreaterThan("app_version", currentVersion);
    // 3. 需要更新版本，先下载bundle更新包；如果不需要更新，不做任何逻辑处理
    query.findObjects(new FindListener<AppVersions>() {
       @Override
       public void done(List<AppVersions> list, BmobException e) {
           if(e == null) {
               if(list != null && !list.isEmpty()) {
                   final AppVersions info = list.get(0);
                   Log.d("msg", "Bundle更新实体的详情 === " + info.toString());

                   // 4. 定义文件存放根目录
                   final File jsBundleFilesRootDir = new File(mCacheRootDir, Constants.NAME_REACT_NATIVE_FOLDER);

                   // 5. 路径不存在，创建该路径；路径存在，在解压前删除该目录下所有文件（后续有讲解）
                   if(!jsBundleFilesRootDir.exists()) {
                       jsBundleFilesRootDir.mkdirs();
                   } else {
                       mFileUtils.deleteDir(jsBundleFilesRootDir);
                   }

                   // 6. 定义Bundle压缩文件存放路径（后续有讲解）
                   final File jsBundleZipFile = new File(jsBundleFilesRootDir, Constants.NAME_JS_BUNDLE_ZIP_FILE);

                   // 7. 下载bundle压缩文件
                   BmobFile bundleFile = info.getAppSource();
                   bundleFile.download(jsBundleZipFile, new DownloadFileListener() {
                       @Override
                       public void done(String s, BmobException e) {
                           if(e == null) {
                               // 8. 解压JSBundle压缩文件到(react_native/JSBundleFile)文件夹下（后续有讲解）
                               File jsBundleFolder = new File(jsBundleFilesRootDir, Constants.NAME_JS_BUNDLE_FILE_FOLDER);
                               boolean result = mFileUtils.unZipFile(jsBundleFolder, jsBundleZipFile);

                               if(result) {
                                   // 9. 解压成功后保存当前bundle的版本
                                   mSP.edit().putString(BUNDLE_VERSION, info.getAppVersion()).apply();
                               } else {
                                   // 9.1 解压失败应该删除掉有问题的文件，防止RN加载错误的bundle文件
                                   File reactDir = new File(getReactApplicationContext().getCacheDir(), Constants.NAME_REACT_NATIVE_FOLDER);
                                   mFileUtils.deleteDir(reactDir);
                               }
                           } else {
                               e.printStackTrace();
                               Log.d("msg", "bundle-patch download failure");
                           }
                       }

                       @Override
                       public void onProgress(Integer integer, long l) {}
                   });
               }
           } else {
               e.printStackTrace();
               Log.d("msg", "获取版本信息失败！");
           }
       }
    });
}
```
以上是热更新程序的主题逻辑

* 2.5.6.8.有分支内容，也很重要，将在下一篇文章中详细讲解

##### 7. 修改ReactNative加载bundle文件路径
MainApplication文件中覆写getJSBundleFile()方法，指定自定义bundle文件读取路径。

```
private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
    @Nullable
    @Override
    protected String getJSBundleFile() {
         // 此处路径是差量更新方式bundle文件的路径，实际应用中改成你自己的File路径即可。
         File bundleFile = new File(getCacheDir() + "/" + Constants.NAME_REACT_NATIVE_FOLDER + "/" + Constants.NAME_JS_BUNDLE_FILE_FOLDER, Constants.NAME_JS_BUNDLE_JS_FILE);
         if(bundleFile.exists()){
           return bundleFile.getAbsolutePath();
         }
         return super.getJSBundleFile();
    }
};
```

#### 三、总结
到这里，RN Android平台热更新功能的技术主体已经讲解完毕。本篇文章讲解了主体流程，分支逻辑和技术细节方面将会在下一篇文章中详细讲解。


