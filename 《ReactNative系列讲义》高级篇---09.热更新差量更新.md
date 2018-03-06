# 《ReactNative系列讲义》高级篇---09.热更新差量更新

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介
通过前面几篇文章我们已经实现了全量热更新，这仅仅是实现了热更新的第一步，全量更新的bundle包会稍显大一些，差量更新就是给bundle包做瘦身。
大致思路如下：
发布APP版本前，保留发布版APP的bundle包；等再次更新的时候，手动打bundle包，将生成的bundle包和保存的原始版本做差量动作，生成差量包；将生成的bundle差量包上传至服务器，修改bundle版本号；用户端APP下载差量包，将差量包与APP本地正使用的bundle包进行合并，生成新的bundle；RN重新加载指定路径的bundle包。
这里讨论的差量指的是代码文件的差量，即index.android.bundle文件的差量计算。图片目前还是全量更新，如果做差量，需要修改RN源码，如果APP RN的版本需要升级，升级完成之后仍然需要再次修改RN源码，后续文章我们会讲到。


#### 二、代码实现
##### 1. 生成差量包

* 需将新旧bundle包转换成String类型的数据

```
/**
 * 将文件转换成字符串
 * FileReader
 * BufferedReader
 *
 * @param path 文件路径
 * @return
 */
public String readFileToString(String path) {
   String results = "";
   try {
       FileReader reader = new FileReader(path);
       BufferedReader bufferedReader = new BufferedReader(reader);
       int result = bufferedReader.read();
       StringBuilder sb = new StringBuilder();
       while(result != -1) {
           sb.append((char) result);
           result = bufferedReader.read();
       }
       bufferedReader.close();
       reader.close();
       results = sb.toString();
   } catch (FileNotFoundException e) {
       e.printStackTrace();
   } catch (IOException e) {
       e.printStackTrace();
   }

   return results;
}
```

* 使用Google diff_match_patch算法库，从官网下载最新的代码文件，从中选择java版本

```
/**
 * 生成差异补丁包
 *
 * @param oldSource 
 * @param newSource 
 * @return
 */
public String createPatch (String oldSource, String newSource) {
   diff_match_patch dmp = new diff_match_patch();
   // 对比新旧资源，获取差异
   LinkedList<diff_match_patch.Diff> diffs = dmp.diff_main(oldSource, newSource);
   // 生成差异补丁包
   LinkedList<diff_match_patch.Patch> patches = dmp.patch_make(diffs);
   // 解析补丁包
   String patchesSource = dmp.patch_toText(patches);

   return patchesSource;
}
```

* 需将生成的差异数据写到文件中

```
/**
 * 将字符串写入文件
 * FileWriter
 * BufferedWriter
 *
 * @param source
 * @param path
 */
public void writeStringToFile(String source, File path) {
   try {
       FileWriter fileWriter = new FileWriter(path);
       BufferedWriter bufferedWriter = new BufferedWriter(fileWriter);
       bufferedWriter.write(source);
       bufferedWriter.close();
       fileWriter.close();
   } catch (IOException e) {
       e.printStackTrace();
   }
}
```

* 编写调用方法

```
/**
 * 生成文件差量包
 */
private void createPatchFile () {
   File oldBundleFile = new File("本地文件路径");
   String oldBundleString = mFileUtils.readFileToString(oldBundleFile.getPath());

   File newBundleFile = new File("本地文件路径");
   String newBundleString = mFileUtils.readFileToString(newBundleFile.getPath());

   String patch = mFileUtils.createPatch(oldBundleString, newBundleString);
   File patchFile = new File("本地文件路径");

   mFileUtils.writeStringToFile(patch, patchFile);
}
```

* 以上功能的实现可能需要你创建Java工程

##### 2. 在Android工程中创建Google diff_match_patch算法工具类

* 算法库需要先行倒入

```
import com.facebook.react.bridge.ReactApplicationContext;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.Writer;
import java.util.LinkedList;

/**
 * Created by fangshifeng on 2018/1/19.
 */

public class GoogleDiffMatchPatchUtils {
    private static GoogleDiffMatchPatchUtils mInstance;
    private diff_match_patch dmp = null;
    private ReactApplicationContext mReactContext;

    private GoogleDiffMatchPatchUtils(ReactApplicationContext reactContext) {
        mReactContext = reactContext;

        if(dmp == null) {
            dmp = new diff_match_patch();
        }
    }

    // 多线程安全单例模式（双重同步锁）
    public static GoogleDiffMatchPatchUtils getInstance(ReactApplicationContext reactContext) {
        if(mInstance == null) {
            synchronized (GoogleDiffMatchPatchUtils.class) {
                if(mInstance == null) {
                    mInstance = new GoogleDiffMatchPatchUtils(reactContext);
                }
            }
        }

        return mInstance;
    }

    /**
     * 文件合并
     *
     * @param assetsBundleFile
     * @param patchesFile
     */
    public void mergePatSourceWithAsset(String assetsBundleFile, String patchesFile, File newBundleFileDir) {
        // 转换pat
        LinkedList<diff_match_patch.Patch> patchesList = (LinkedList<diff_match_patch.Patch>) dmp.patch_fromText(patchesFile);
        // 合并，生成新的bundle
        Object[] bundleArray = dmp.patch_apply(patchesList, assetsBundleFile);

        try {
            Writer writer = new FileWriter(newBundleFileDir);
            String newBundleFile = (String) bundleArray[0];
            writer.write(newBundleFile);
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

##### 3. 修改热更新主流程

* 修改构造方法

```
private GoogleDiffMatchPatchUtils mPatchUtils;

public VersionUpdateCheckModule(ReactApplicationContext reactContext) {
   super(reactContext);
   // 实例化算法工具类
   mPatchUtils = GoogleDiffMatchPatchUtils.getInstance(reactContext);
}
```

* 修改check方法
在第9步之后添加相关的文件操作

```
try {
    // 10. 获取本地index.android.bundle文件
    String assetsBundleFile = mFileUtils.readInputStreamToString(mReactContext.getAssets().open(Constants.NAME_JS_BUNDLE_JS_FILE));
    // 11. 获取解压后的bundle差量包
    String patchesFile = mFileUtils.readFileToString(jsBundleFolder.getPath() + "/" + Constants.NAME_BUNDLE_PATCH_FILE);

    if(!TextUtils.isEmpty(assetsBundleFile) && !TextUtils.isEmpty(patchesFile)) {
        // 12. 将本地bundle文件和差量包进行合并
        File newBundleFile = new File(jsBundleFolder, Constants.NAME_JS_BUNDLE_JS_FILE);

        if(!newBundleFile.exists()) {
            newBundleFile.createNewFile();
        }

        mPatchUtils.mergePatSourceWithAsset(assetsBundleFile, patchesFile, newBundleFile);

        // 13.删除.pat文件
        new File(jsBundleFolder + "/" + Constants.NAME_BUNDLE_PATCH_FILE).delete();
    }
} catch (IOException error) {
    error.printStackTrace();
}
```

##### 4. 修改JSBundle加载路径

* 将MainApplication文件中加载JSbundle文件的路径修改为合并之后新bundle包路径

```
@Override
protected String getJSBundleFile() {
    File bundleFile = new File(getCacheDir() + "/" + Constants.NAME_REACT_NATIVE_FOLDER + "/" + Constants.NAME_JS_BUNDLE_FILE_FOLDER, Constants.NAME_JS_BUNDLE_JS_FILE);
    if(bundleFile.exists()){
        return bundleFile.getAbsolutePath();
    }
    return super.getJSBundleFile();
}
```

##### 5. 手动打包命令

```
bundle文件打包命令详解
react-native bundle 
--entry-file index.android.js 
--bundle-output ./android/app/src/main/assets/index.android.bundle
--platform android 
--assets-dest ./bundle 
--dev false

参数含义
--entry-file（入口文件）：android平台是index.android.js；iOS平台是index.ios.js
--bundle-output（bundle文件输出路径）： 项目根目录/android/app/src/main/assets/index.android.bundle
--platform（平台）：android
--assets-dest（图片资源输出目录）： 项目根目录/bundle
--dev（是否是开发版本）
```

#### 三、总结
如果没有疏漏的话，Android平台RN热更新差量更新（代码部分，即index.android.bundle文件）算是全部实现了。后续还有一些地方可以优化升级：版本升级功能可放入子线程；图片更新做成差量。

#### 四、文章参考
* https://www.jianshu.com/p/2cb3eb9604ca
* http://blog.csdn.net/u013718120/article/details/55096393
* http://blog.csdn.net/it_talk/article/details/54346566





