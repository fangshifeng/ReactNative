# 《ReactNative系列讲义》高级篇---08.热更新分支逻辑实现

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介
前面我们已经把热更新的主体逻辑讲完了，并且主节点的代码实现也一一讲解了。本篇文章我们来详细讲解分支逻辑的技术实现，这部分的内容偏JAVA基础，基本属于辅助性功能，但是实现的好坏也关系到效率的高低，接下来我们一起来看看主节点2.5.6.8的分支内容的实现过程。

#### 二、代码讲解

##### 1. 构造方法及相关对象的实现

```
// 自定义Native module的名称
private static final String REACT_CLASS = "VersionUpdateCheck";
// SP中存储bundle版本号的key值
private static final String BUNDLE_VERSION = "CurrentBundleVersion";
// 上下文对象
private ReactApplicationContext mReactContext;
// SP对象
private SharedPreferences mSP;
// 自建文件操作工具类对象
private FileUtils mFileUtils;
// android存储根目录
private File mCacheRootDir;

public VersionUpdateCheckModule(ReactApplicationContext reactContext) {
   super(reactContext);
   mReactContext = reactContext;
   mSP = reactContext.getSharedPreferences("react_bundle", Context.MODE_PRIVATE);
   reactContext.addLifecycleEventListener(this);
   mFileUtils = FileUtils.getInstance(reactContext);
   mCacheRootDir = getReactApplicationContext().getCacheDir();
}
```

##### 2. Bmob SDK的嵌入与应用
建立Bean文件AppVersions，继承BmobObject类

```
public class AppVersions extends BmobObject {
    private String app_version; // bundle版本
    private String update_content; // 更新内容
    private BmobFile app_source; // 要下载的bundle文件
    private Boolean immediately; // bundle是否立即生效

    public String getAppVersion() {
        return app_version;
    }

    public String getUpdateContent() {
        return update_content;
    }

    public BmobFile getAppSource() {
        return app_source;
    }

    public Boolean getImmediately() {
        return immediately;
    }

    @Override
    public String toString() {
        return "app_version = " + getAppVersion() +
                " --> " + "update_content = " + getUpdateContent() +
                " --> " + "getAppSource = " + getAppSource() +
                " --> " + "getImmediately = " + getImmediately();
    }
}
```
建议Bomb云后台的详细使用方法请查看Bmob官方文档

##### 3. FileUtils
文件操作工具类，包含I/O流，File操作

```
import android.util.Log;

import com.facebook.react.bridge.ReactApplicationContext;

import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.Enumeration;
import java.util.zip.ZipEntry;
import java.util.zip.ZipFile;

/**
 * Created by fangshifeng on 2018/1/19.
 */

public class FileUtils {
    private static FileUtils mInstance;
    private ReactApplicationContext mReactContext;

    private FileUtils(ReactApplicationContext reactContext) {
        mReactContext = reactContext;
    }

    public static FileUtils getInstance(ReactApplicationContext reactContext) {
        if(mInstance == null) {
            synchronized (FileUtils.class) {
                if(mInstance == null) {
                    mInstance = new FileUtils(reactContext);
                }
            }
        }

        return mInstance;
    }

    /**
     * 解压缩
     *
     * @param dirPath 压缩文件存放目录 FileObject
     * @param zipFile 压缩文件 FileObject
     *
     * @return true 解压缩成功；false 解压缩失败
     */
    public boolean unZipFile(File dirPath, File zipFile) {
        if(!dirPath.exists()) {
            dirPath.mkdirs();
        } else {
            deleteDir(dirPath);
        }

        try {
            // 1. 创建压缩文件通道
            ZipFile zf = new ZipFile(zipFile);
            for (Enumeration<?> entries = zf.entries(); entries.hasMoreElements();) {
                ZipEntry entry = (ZipEntry) entries.nextElement();

                // 排除一些垃圾文件
                if(entry.getName().contains("__") || entry.getName().contains("._")) {
                    continue;
                }

                if(entry.isDirectory()) {
                    File folderFile = new File(dirPath, entry.getName());
                    folderFile.mkdirs();
                } else {
                    // 2. 创建文件输入流通道（从外部存储介质中读取文件到内存中）
                    InputStream is = zf.getInputStream(entry);

                    // 3. 创建解压缩之后的文件路径及名称（创建输出文件通道，指定接收字节流数据的载体）
                    File bundleFileDir = new File(dirPath, entry.getName());
                    bundleFileDir.createNewFile();

                    // 4. 创建输出流通道并将文件通道包裹其中（将字节流数据从内存中写入外部存储介质中）
                    OutputStream out = new FileOutputStream(bundleFileDir);

                    // 5. 流通道创建完成！开始输送
                    byte buffer[] = new byte[1024];
                    int realLength;
                    while ((realLength = is.read(buffer)) > 0) {
                        out.write(buffer, 0, realLength);
                    }

                    // 6. 关闭所有通道
                    is.close();
                    out.close();
                }
            }

            return true;
        } catch (IOException e) {
            e.printStackTrace();
        }

        return false;
    }

    /**
     * 递归删除目录下的所有文件及子目录下所有的文件
     *
     * @param dir
     * @return
     */
    public boolean deleteDir(File dir) {
        if(dir.isDirectory()) {
            String[] children = dir.list();
            // 递归删除目录中的子目录下
            for(int i = 0; i < children.length; i ++) {
                boolean success = deleteDir(new File(dir, children[i]));
                if(!success) {
                    return false;
                }
            }
        }

        // 目录此时为空，可以删除
        return dir.delete();
    }

    /**
     * 浏览指定目录
     *
     * @param dir
     */
    public void scanFileDir(File dir) {
        String absolutePath = dir.getAbsolutePath();

        File dirFile = new File(absolutePath);
        if(dir.isDirectory()) {

            File[] children = dirFile.listFiles();
            for(int i = 0; i < children.length; i ++) {
                Log.d("msg", "children = " + children[i].getName());
            }
        }
    }

    /**
     * 读取字节流到字符串
     *
     * @param is 字节流通道
     * @return
     */
    public String readInputStreamToString(InputStream is) {
        String result = "";

        try {
            // 1. 创建缓冲输入流，提高读取效率
            BufferedInputStream bis = new BufferedInputStream(is);
            // 2. 获取文件总长度
            int size = bis.available();
            // 3. 创建字节流数据存储数组
            byte[] buffer = new byte[size];
            // 4. 一次性读取所有字节存储到buffer数组中
            bis.read(buffer);
            // 5. 关闭字节流通道
            bis.close();
            // 6. 字节转换成字符串
            result = new String(buffer, "UTF-8");
        } catch (IOException e) {
            e.printStackTrace();
        }

        return result;
    }

    /**
     * 将文件转换成字符串
     *
     * @param path 文件路径
     * @return
     */
    public String readFileToString(String path) {
        FileReader reader;
        String results = "";
        try {
            reader = new FileReader(path);
            int result = reader.read();
            StringBuilder sb = new StringBuilder();
            while(result != -1) {
                sb.append((char) result);
                result = reader.read();
            }
            reader.close();
            results = sb.toString();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

        return results;
    }
}
```

* 关于文件操作工具类中某些地方并不是很严谨，为了功能的实现，只是个参照而已。大家可以按照自己的思路和习惯去创建文件操作工具类。

##### 4. Constants
定义全局的常量

```
/**
 * 定义常量
 *
 * Created by fangshifeng on 2018/1/19.
 */

public class Constants {
    // 定义存放Bundle差量压缩包文件的文件夹名称 (/data/user/0/com.snailapp/cache/react_native)
    public static final String NAME_REACT_NATIVE_FOLDER = "react_native";
    // 定义Bundle差量压缩包名称 （/data/user/0/com.snailapp/cache/react_native/bundle-patch.zip）
    public static final String NAME_JS_BUNDLE_ZIP_FILE = "bundle-patch.zip";
    // 定义存放Bundle差量包压缩文件解压后文件的文件夹名称（/data/user/0/com.snailapp/cache/react_native/JSBundleFile）
    public static final String NAME_JS_BUNDLE_FILE_FOLDER = "JSBundleFile";
    // Assets文件夹中bundle文件名称
    public static final String NAME_JS_BUNDLE_JS_FILE = "index.android.bundle";
    // bundle差量包名称（/data/user/0/com.snailapp/cache/react_native/JSBundleFile/patches.pat）
    public static final String NAME_BUNDLE_PATCH_FILE = "patches.pat";
}
```

* Constants文件中定义的常量都是在差量更新方式下创建的常量，全量更新方式不需要这么多的常量。这里我不在做修改，大家只需要自己定义一个存放解压缩文件存放的路径即可（如果我没记错的话-_-!!!）。

#### 三、总结
到此，Anroid平台RN热更新功能70%的技术点已经讲完。通过这几篇文章，我们完整的实现了热更新的全量更新。以上功能其实可以应用到实际项目中，但是，热更新到这里还远远没有完，还有很多的地方可以升级，它还可以变得更完美。不过，首先我们要先做到完成。下一篇中，我们会讲解热更新的差量更新。


