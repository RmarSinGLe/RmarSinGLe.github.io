---
title: "unityAssetBundle笔记"
math: true
date: 2025-03-22 10:30:00 +0800
categories: [Unity]
tags: [打包,热更]
---
## 什么是AssetBundle（AB包）

AB包就是特定于平台的资产压缩包，有点类似压缩文件。这些资产包括但不限于∶模型、贴图、预设体、音效、材质球等等。但是它不支持将C#代码一并打包，这也是要学习Lua的原因

## AB包的作用

AB包相对于Resources加载，它可以更好的管理资源。项目的Resources在出包的时候就已经无法变动了，且只读无法修改。在Resources文件下的文件，无论是否被工程使用到它都会被打包。而AB包存储位置可自定义，压缩方式自定义且后期可以动态更新。而且AB包可以减少包体大小。这是因为AB包本身就有压缩资源，如果我们将AB包放置到后端服务器上然后让玩家在包体安装好后再去下载AB包。那么初始包的包体大小就会减少了。除此之外，AB包也支持热更新。

![Pasted image 20250425173657.png](assets/img/Pasted image 20250425173657.png)

## 如何生成AB包

生成AB包资源文件有两种方法：我们使用Unity编译器开发自定义打包工具或者是使用Unity官方提供的打包工具Asset Bundle Browser来进行打包。

> 在开始前，我们首先要让资源和AB包关联。首先我们选中任意一个预制体，然后在Inspector窗口下选中AssetBundle选项并且new一个名字
> 
> 这是用来表明这个预制体会打包到model这个包中
> 
> 此时大家可以点击Windows→AssetBundleBrowser来查看当前项目中AB包的信息。大家可以在它的Configure页签下看到一个model的选项，再次点击这个选项就可以看到我们之前设置的模型了
> 另一个页签Build就是我们用来打包的。BuildTarget是用来指定我们要打包到哪里的平台
> 
> OutputPath是我们打包成功后输出的路径。Clear Folders表示是否要请空OutputPath路径下的文件夹。Copy toStreamingAssets表示我们是否要将其复制到StreamingAssets文件夹中AdvancedSetting下Comperssion是压缩类型，LZMA压缩的大小最小但是每次加载要将全部的资源都解压出来，LZ4压缩的大小较大但是我们可以要一个资源解压一个。ExcludeType Information在资源包中不包含资源的类型信息。ForceRebuild重新打包时需要重新构建包，但是它和ClearFolders不同，它不会删除不再存在的包。lgnoreType Tree Changes增量构建检查时，忽略类型数的更改。Append Hash将文件哈希值附加到资源包名上。StrictMode严格模式，如果打包时报错了，则打包直接失败无法成功。Dry RunBuild运行时构建。我们设置完后点击Build按钮就可以进行打包

## 使用代码加载AB包文件

### 同步和异步加载AB包

同步加载
```
using System.Collections;  
using System.Collections.Generic;  
using System.IO;  
using UnityEngine;  
  
namespace MyGame  
{  
    public class ABTest : MonoBehaviour  
    {  
        void Start()  
        {  
            // 加载AB包  
            AssetBundle ab = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "model"));  
            GameObject obj = ab.LoadAsset<GameObject>("Cube");  
            GameObject.Instantiate(obj);            
        }  
    }  
}
```

因为我们之前设定文件在项目打包后是不能打进包中的，所以我们才用streamingAssets文件存储。实际在项目中，AB包资源会存放在服务器中然后我们从服务器中下载

我们不能重复加载AB包比如下面的这段代码就会报错：TheAssetBundle 'StreamingAssets' can't be loaded because anotherAssetBundle with the same files is already loaded.
```
using System.Collections;  
using System.Collections.Generic;  
using System.IO;  
using UnityEngine;  
  
namespace MyGame  
{  
    public class ABTest : MonoBehaviour  
    {  
        void Start()  
        {  
            // 加载AB包  
            AssetBundle ab = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "model"));  
            GameObject obj = ab.LoadAsset<GameObject>("Cube");  
            GameObject.Instantiate(obj);  
            AssetBundle ab2 = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "model"));  
        }  
    }  
}
```


异步加载
```
using System.Collections;  
using System.Collections.Generic;  
using System.IO;  
using UnityEngine;  
  
namespace MyGame  
{  
    public class ABTest : MonoBehaviour  
    {  
        void Start()  
        {  
            StartCoroutine(LoadABAsync("model", "Cube"));  
        }  
  
        // 异步加载  
        private IEnumerator LoadABAsync(string abName, string resName)  
        {  
            // 加载ab包  
            AssetBundleCreateRequest abcr = AssetBundle.LoadFromFileAsync(Path.Combine(Application.streamingAssetsPath, abName));  
            yield return abcr; // 等待ab包加载完  
            // 异步加载资源  
            AssetBundleRequest abr = abcr.assetBundle.LoadAssetAsync<GameObject>(resName);  
            yield return abr;  
            GameObject.Instantiate(abr.asset as GameObject);  
            yield return null;  
        }  
    }  
}
```
我们可以调用AssetBundle.UnloadAllAssetBundles来卸载全部加载过的AB包。但是如果我们只想卸载个别的AB包，那么可以按照下面的代码执行：
```
using System.Collections;  
using System.Collections.Generic;  
using System.IO;  
using UnityEngine;  
  
namespace MyGame  
{  
    public class ABTest : MonoBehaviour  
    {  
        void Start()  
        {  
            // 加载AB包  
            AssetBundle ab = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "model"));  
            ab.Unload(false);  
        }  
    }  
}
```
AssetBundle的官网的介绍：[AssetBundle - Unity 脚本 API](https://docs.unity3d.com/cn/2022.2/ScriptReference/AssetBundle.html)

### 代码加载AB包中的依赖

 我创建了一个新的材质命名为CubeMat，然后我让它被打包在mat包中。那么在我们加载model包之前，我们就要先让mat包被加载或者同时加载。而包之间的依赖，我们可以通过主包来获得。具体代码如下：
```
using System.Collections;  
using System.Collections.Generic;  
using System.IO;  
using UnityEngine;   
namespace MyGame  
{  
     public class ABTest : MonoBehaviour  
     {  
         void Start()  
         {  
             LoadAB();  
         }  
  
         private void LoadAB()  
         {  
             // 加载主包  
             AssetBundle mainAB = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "StandaloneWindows"));  
             // 在主包中默认就有这个存在  
             AssetBundleManifest manifest = mainAB.LoadAsset<AssetBundleManifest>("AssetBundleManifest");  
             // 获取所有的依赖  
             string[] strs = manifest.GetAllDependencies("model");  
             foreach(string str in strs)  
             {  
                 AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, str));  
             }  
             AssetBundle ab = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "model"));  
             GameObject obj = ab.LoadAsset<GameObject>("Cube");  
             GameObject.Instantiate(obj);  
         }  
    }  
}
```

## AB包管理器

视频中使用了单例模式，但是我感觉更本没有这个必要的直接静态类应该也可以的。这里我就贴一个简单的单例实现就是了。我看视频中在调用的时候出现了一个DontDestroyOnLoad，我想这个单例大概是继承了Test : MonoBehaviour。所以单例模式如下：
```
using System.Collections;  
using System.Collections.Generic;  
using UnityEngine;  
  
namespace Common  
{  
    public class MonoSingle<T> : MonoBehaviour where T : MonoSingle<T>  
    {  
        private static T instance;  
        private static Object lockflag = new Object();  
        public static T Instance  
        {  
            get  
            {  
                if (instance == null)  
                {  
                    lock (lockflag)  
                    {  
                        if (instance == null)  
                        {  
                            instance = FindObjectOfType<T>();  
                            if (instance == null)  
                            {  
                                new GameObject(typeof(T).Name).AddComponent<T>();  
                            }  
                            instance.Init();  
                        }  
                    }  
                }  
                return instance;  
            }  
        }  
        void Awake()  
        {  
            if (instance == null)  
            {  
                instance = this.transform.GetComponent<T>();  
                instance.Init();  
            }  
            else Destroy(this.gameObject);  
        }  
  
        public virtual void Init()  
        {  
  
        }  
    }  
}
```
具体的AB包管理器代码如下：
```
using Common;  
using System.Collections;  
using System.Collections.Generic;  
using System.IO;  
using UnityEngine;  
  
namespace MyGame  
{  
    public class ABManager : MonoSingle<ABManager>  
    {  
        // 存储已经加载过的AB包  
        private Dictionary<string, AssetBundle> loadedAB = new Dictionary<string, AssetBundle>();  
  
        private string pathUrl = Application.streamingAssetsPath;  
  
        private string MainPackName  
        {  
            get  
            {  
#if UNITY_IOS  
    return "IOS";  
#elif UNITY_ANDROID  
    return "android";  
#else  
                return "StandaloneWindows";  
#endif  
            }  
        }  
  
        private AssetBundle mainPackage = null;  
        private AssetBundleManifest manifest = null;  
  
        // 加载依赖包  
        private void LoadDependencies(string abName)  
        {  
            if (loadedAB.ContainsKey(abName))  
            {  
                return;  
            }  
            if (mainPackage == null)  
            {  
                // 加载主包  
                mainPackage = AssetBundle.LoadFromFile(Path.Combine(pathUrl, MainPackName));  
                // 在主包中默认就有这个存在  
                manifest = mainPackage.LoadAsset<AssetBundleManifest>("AssetBundleManifest");  
            }  
  
            // 获取所有的依赖  
            string[] strs = manifest.GetAllDependencies(abName);  
            foreach (string str in strs)  
            {  
                if (!loadedAB.ContainsKey(str))  
                {  
                    loadedAB.Add(str, AssetBundle.LoadFromFile(Path.Combine(pathUrl, str)));  
                }  
            }  
        }      
  
        //同步加载  
        public Object loadRes(string abName, string resName)  
        {  
            if (loadedAB.ContainsKey(abName))  
            {  
                return loadedAB[abName].LoadAsset(resName);  
            }  
            LoadDependencies(abName);  
            loadedAB.Add(abName, AssetBundle.LoadFromFile(Path.Combine(pathUrl, abName)));  
            return loadedAB[abName].LoadAsset(resName);  
        }  
  
        public Object loadRes(string abName, string resName, System.Type type)  
        {  
            if (loadedAB.ContainsKey(abName))  
            {  
                return loadedAB[abName].LoadAsset(resName, type);  
            }  
            LoadDependencies(abName);  
            loadedAB.Add(abName, AssetBundle.LoadFromFile(Path.Combine(pathUrl, abName)));  
            return loadedAB[abName].LoadAsset(resName, type);  
        }  
  
        public T loadRes<T>(string abName, string resName, System.Type type) where T : UnityEngine.Object  
        {  
            if (loadedAB.ContainsKey(abName))  
            {  
                return loadedAB[abName].LoadAsset<T>(resName);  
            }  
            LoadDependencies(abName);  
            loadedAB.Add(abName, AssetBundle.LoadFromFile(Path.Combine(pathUrl, abName)));  
            return loadedAB[abName].LoadAsset<T>(resName);  
        }  
  
        // 卸载包  
        public void UnLoadAB(string abName, bool unLoadAllLoadedObj = false)  
        {  
            if (loadedAB.ContainsKey(abName))  
            {  
                loadedAB[abName].Unload(unLoadAllLoadedObj);  
                loadedAB.Remove(abName);  
            }  
        }  
        public void UnLoadAllAB(bool unLoadAllLoadedObj = false)  
        {  
            loadedAB.Clear();  
            AssetBundle.UnloadAllAssetBundles(unLoadAllLoadedObj);  
            mainPackage = null;  
            manifest = null;  
        }  
    }  
}
```