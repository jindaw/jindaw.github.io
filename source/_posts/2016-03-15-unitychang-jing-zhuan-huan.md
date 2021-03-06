---
layout: post
title: "Unity场景转换"
date: 2016-03-15 17:11:55 +0800
comments: true
categories: Unity
---
#Unity 场景切换
Unity场景切换只有一种方法，就是使用application，但是在使用的时候可以有两种方法。

[TOC]

## 第一种方法原生application实现
###可用api接口介绍
1. loadedLevel: 返回最后加载关卡的索引
2. loadedLevelName: 返回最后加载的关卡名称
3. isLoadingLevel: 是否正在加载关卡
4. levelCount: 可用关卡总数
5. LoadLevel: 加载关卡，释放当前scene
6. LoadLevelAsync: 异步加载一个scene,可用loadingScene过渡
7. LoadLevelAdditive: 累加一个关卡，不销毁当前scene
8. LoadLevelAdditiveAsync: 异步加载一个scene,不释放当前scene

###实际中使用示例
* 使用Application.LoadLevel(0||name)加载索引为0或者场景名的场景，其中的场景名需要在Unity中使用File->Build Settings.....添加场景，添加后的场景顺序就是索引。
* 添加游戏Scene时，当前已经加载的对象都会被销毁，如果想不被销毁可使用Object.DontDestroyOnLoad方法保存对象。
例如：
		public class NewBehaviourScript2 : MonoBehaviour
        {
        public Texture BackImage = null;
        private AsyncOperation async = null;

        void Start ()
        {
            //此物体在下一个场景中不会被销毁
            DontDestroyOnLoad(this);
            //开始加载场景
            StartCoroutine("LoadScene");
        }
        //异步加载
        IEnumerator LoadScene()
        {
            async = Application.LoadLevelAsync("Next");
            yield return async;
            Debug.Log("Complete!");
        }

        void OnGUI()
        {
            //切换场景中的背景，可以是图片或者动画，或者其它
            GUI.DrawTexture(new Rect(0, 0, Screen.width, Screen.height), BackImage);
            //加载过程中显示进度，也可以是进度条
            if (async != null && async.isDone == false)
            {
                GUIStyle style = new GUIStyle();
                style.fontSize = 48;
                GUI.Label(new Rect(0, 200, Screen.width, 20), async.progress.ToString("F2"), style);
            }

            //在加载结束后，弹出是否下个场景，模拟手游中"触摸任意位置进入游戏"
            if (async != null && async.isDone == true)
            {
                if (GUI.Button(new Rect(100, 100, 100, 100), new GUIContent("跳起进入下一个场景")))
                {
                    Destroy(this);
                }
            }
        }
    	}


## 第二种方法使用ScenceManager插件实现封装加效果

###SceneManger提供Scene和Level两种场景，一个是大场景转换，一个是关卡之间的场景转换。我们通过创建一个SceneConfiguration来编辑游戏中所有Scene的类别和关系。

###实现
（1）SMSceneManager
一旦Scene Configuration创建完成之后，即可以在第一个“Screen场景”中创建出单例类SMGameEnvironment实例，其
其构造方法中完成对SMSceneManager与SMLevelProgress实例的创建：
（注意一定要在Screen场景中实例化SMGameEnvironment，如果是Level场景，则有可能对各个Level之间的关系有错误）
SMSceneManager提供切换场景的接口（包括加载场景，加载关卡，加载第一个关卡）
SMLevelProgress用以保存Level之间的关系（包括当前Level，上一Level，当前Level状态）

（2）SMTransition
SMTransition及其子类，提供了很多方便的切换场景（包括Screen和Level）动画效果，包括 淡入淡出，闪烁，卡通等等

（这些动画效果都作为Prefab保存在SceneManager/Resources/Transitions/下）
SMTransition作为基类，提供了是否异步加载场景，实际调用Unity3d API切换场景方法，但主要提供了一个动画的模板方法 DoTransition()
例如 SMFadeTransition 类中，通过传入参数elapsedTime与配置淡入淡出参数duration计算得到当前进度，正交化进度，得到当前遮盖的alpha值，并在OnGUI绘制

