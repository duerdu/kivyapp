# A simple python kivy-app instructions 
[Kivy](https://kivy.org) - Open source Python library for rapid development of applications. Kivy runs on Linux, Windows, OS X, Android, iOS, and Raspberry Pi. 

记录用python Kivy开发一个小应用的例子，by Kivy-2.1.0, macOS。

## 安装配置
[官方doc](https://kivy.org/doc/stable/gettingstarted/installation.html)
- 开发包
    pip到指定venv
``` 
    python -m virtualenv kivy_venv
    source kivy_venv/bin/activate
    python -m pip install "kivy[base]" kivy_examples 
``` 
> 有个[kivy-designer](https://github.com/kivy/kivy-designer)，但不更新了。

- 打包包(macOS)
    下载[Kivy.dmg](https://kivy.org/downloads/2.1.0/)，安装后生成一个空应用套件/Applications/Kivy.app，打包就从复制该Kivy.app开始。
> Win没试

- 中文支持
    用中文字体如SimSun.ttf替换以下文件（命同名），若开发和打包环境不一样都替换一下。
    ```kivy_venv/lib/python3.8/site-packages/kivy/data/fonts/Roboto-Regular.ttf ```

## 填坑过程
开发时按照[quickstart](https://kivy.org/doc/stable/guide/basic.html#quickstart)问题不大，主要是打包时有些调整。

1. 文件目录，用于生成python模块，以直接pip进要打包的kivy-venv环境
```
├── MyApp
│   ├── __init__.py （空文件）
│   ├── setup.py （模块配置）
│   ├── MyApp（安装后的package包名）
│   │   ├── main.py (名称须就是main)
│   │   ├── my.kv (以main.py中"MyApp().run())"之"App"前的部分命名）
│   │   ├── mod1.py (待引用子模块)
│   │   ├── __init__.py （空文件）
│   │   └── data（字体图片等用到文件）
│   │       ├── SimSun.ttf
│   │       ├── bg.jpg
│   │       ├
```

2. setup.py
生成python模块的设置，解决打包安装后，双击图标不运行的问题。
``` 
...
    package_data = {
        # 没打进包的文件可以在此检查检查
        # If any package contains *.kv files, include them:
        '': ['*.kv'],
        # Include any *.ttf files and bg.jpg in 'data' subdirectory
        'MyApp': ['data/*.ttf','data/bg.jpg'],
    },
    packages=find_packages(),
    # 下面是使安装后/Applications/MyApp.app可以双击打开的前提
    # "RunApp"是main.py中包含"MykApp().run()"的启动类，可以是只有这一句的class
    entry_points={'console_scripts': ['MyApp=MyApp.main:RunApp']},
...    
```

2. 代码结构
    - py文件，逻辑写python
        * 主文件以main.py命名
        * 双击安装后的MyApp.app可执行
        有一个包含"MyApp().run()"的class，且类名写进setup.py的entry_points
        > 双击不管用时可加上：environ['KIVY_NO_CONSOLELOG'] = '1'，没试
        * 窗口默认可随意拉伸，往小拽走样，设个窗口最小值
        ``` 
        from kivy.core.window import Window
        Window.size = (700, 500)
        Window.minimum_width, Window.minimum_height = Window.size 
        ```
        > 另有一个：Config.set('graphics', 'resizable', 0)，没试
        * 打包后相对路径可能有问题，试试统一整成绝对路径
        ``` 
        from os import path
        abspath=path.dirname(path.realpath(__file__))
        BgDir=f"{abspath}/data/bg.jpg"
        ```
        * 读取GUI的kv文件传入的变量
        ``` 
        #用控件的id获取/修改其属性值
        self.ids['LabelId'].text
        ```
        * import同目录下的py模块
        ``` 
        #开发时'mod1'可以'.mod1'不行，打包后反之...
        from .mod1 import modxxx
        ```
    
    - kv文件，GUI写Kv language
        * 命名：默认导入main.py中"MyApp().run()"的"App"之前部分命名的kv文件
        * 读取py文件中的变量/函数：root.xxx，可带参传入
        ``` 
        root.abspath
        root.bt_deact(id='ButtonID')  
        ```
        * 灵活绘制：canvas.before等可以在控件上绘制各种形状
        * kivy-examples，用pip安装时的例子
        ``` 
        kivy_venv/share/kivy-examples/demo/kivycatalog
        kivy_venv/share/kivy-examples/demo/showcase
        ``` 

3. 撸码
    - 开发时，用虚拟的kivy python环境执行
    ``` 
    pushd xxx/kivy_venv/bin
    source activate
    ...
    deactivate
    popd  
    ```
    或者，alias后直接kivy开始
    ``` alias kivy="source xxx/kivy_venv/bin/activate " ```

4. 打包
    - 下载kivy-sdk-packager，and Use Kivy.app 
    Packaging using the Kivy SDK is recommended for general use.
    使用Kivy.dmg安装后生成的/Applications/Kivy.app
    ``` 
    git clone https://github.com/kivy/kivy-sdk-packager
    cd kivy-sdk-packager/osx
    cp -R /Applications/Kivy.app MyApp.app
    ```   
    > cp时要用大写的R   

    - 安装开发的python模块
    使用setup.py的配置，安装到复制的MyApp.app的python环境
    ``` 
    # 使用MyApp的python环境
    pushd MyApp.app/Contents/Resources/venv/bin
    source activate
    popd
    # 安装开发的python模块，MyApp为包含setup.py的目录
    python -m pip install xxx/MyApp
    ... 
    deactivate
    ```  
    
    - 打包设置
    
        * 先瘦身：
        ``` ./cleanup-app.sh MyApp.app ```
    
        * 创建双击图标打开的入口：
        ``` 
        pushd MyApp.app/Contents/Resources/
        ln -s ./venv/bin/MyApp MyApp
        popd
        ```
        > **此处需要编辑 MyApp.app/Contents/Resources/script,**
        > **将其中的"yourapp"替换为ln过来的自己的应用名MyApp**
        > 正常的话，此时双击本MyApp.App，就能打开应用窗口
        > 窗口没有log，在此venv环境中直接"python xxx/main.py"也可以运行

        * Relocate the hard-coded links created by pip
        ``` ./relocate.sh MyApp.app ```
        * package it
        在当前目录生成MyApp.dmg，拿去安装试试。
        ``` ./create-osx-dmg.sh MyApp.app MyApp ```
    
5. 问题
    - *import子模块时，相对引用时加不加'.'的问题*
    - *dmg体积大，怎么进一步缩小* 



## 参考
1. https://kivy.org
2. https://kivy.org/doc/stable/gettingstarted
3. https://github.com/kivy/kivy-sdk-packager
4. [Kivy中文编程指南](https://cycleuser.gitbooks.io/kivy-guide-chinese/content/17-Kivy-Pack-Mac.html)
5. [把Python的setup.py给整明白](https://zhuanlan.zhihu.com/p/276461821)





