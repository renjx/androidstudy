## task2 布局与基本widgets控件设计

android的布局是通过xml文件来描述的，放在res->layout目录下，本章我们学习做一个抽屉式布局的APP，叫“文件管理”。

### 2.1 创建Navigation Drawer Activity模板项目。

#### 2.1.1 创建FileManage项目

打开android studio，选择File->New->New Project，然后选择Navigation Drawer Activity模板，点击Next。设置Name为FileManage，包名自取。

系统自动生成了如下Java类：

MainActivity，主Activity；还有多个Fragment和ViewModel，其中Franment展示数据，ViewModel保存数据。

系统生成的布局比较复杂：

activity_main.xml：页面整体布局，包括app_bar_main和NavigationView两个部分，app_bar_main中是界面主体，NavigationView包含抽屉的菜单内容。

app_bar_main.xml：包含三个部分AppBarLayout，content_main和FloatingActionButton。其中content_main是界面主体，AppBarLayout用于在界面上方显示文件管理器的路径信息，最后的FloatingActionButton是个浮动按钮。

content_main.xml：使用constraintlayout布局，里面放了个fragment，用于切换各个fragment，下个任务会讲解fragment的具体使用。

fragment_gallery.xml，fragment_home.xml，fragment_slideshow.xml：三个系统默认生成的fragment布局，后面会被我们实际的文件管理功能的各个fragment所取代。

nav_header_main.xml：存放抽屉头部信息。

activity_main_drawer.xml：存放抽屉中的菜单信息。

main.xml：存放主菜单，比如首页，搜索等按钮。

mobile_navigation.xml：实现应用类节目的跳转，让你可以很优雅的管理Fragment，让单Activity应用成为首选架构。

下面我们来设计一下这些界面，界面中遇到的所有素材，尤其是图片，请通过[material](/files/material.rar)下载。图片需要复制到mipmap目录下。

#### 2.1.2 基本UI设计

string.xml中添加如下字符串信息。我们的文件管理器里面会用到许多字符串信息，这里尽可能一次性列出所有的字符串。可以把下列文件复制到strings.xml中。

```
<resources>
    <string name="app_name">文件管理</string>

    <string name="navigation_drawer_open">Open navigation drawer</string>
    <string name="navigation_drawer_close">Close navigation drawer</string>

    <string name="action_settings">设置</string>
    <string name="title_activity_settings">设置</string>

    <!-- Strings related to Settings -->

    <!-- Example General settings -->
    <string name="pref_header_general">General</string>

    <string name="pref_title_social_recommendations">Enable social recommendations</string>
    <string name="pref_description_social_recommendations">Recommendations for people to contact
        based on your message history
    </string>

    <string name="pref_title_display_name">Display name</string>
    <string name="pref_default_display_name">John Smith</string>

    <string name="pref_title_add_friends_to_messages">Add friends to messages</string>
    <string-array name="pref_example_list_titles">
        <item>Always</item>
        <item>When possible</item>
        <item>Never</item>
    </string-array>
    <string-array name="pref_example_list_values">
        <item>1</item>
        <item>0</item>
        <item>-1</item>
    </string-array>

    <!-- Example settings for Data & Sync -->
    <string name="pref_header_data_sync">Data &amp; sync</string>

    <string name="pref_title_sync_frequency">Sync frequency</string>
    <string-array name="pref_sync_frequency_titles">
        <item>15 minutes</item>
        <item>30 minutes</item>
        <item>1 hour</item>
        <item>3 hours</item>
        <item>6 hours</item>
        <item>Never</item>
    </string-array>
    <string-array name="pref_sync_frequency_values">
        <item>15</item>
        <item>30</item>
        <item>60</item>
        <item>180</item>
        <item>360</item>
        <item>-1</item>
    </string-array>

    <string-array name="list_preference_entries">
        <item>Entry 1</item>
        <item>Entry 2</item>
        <item>Entry 3</item>
    </string-array>

    <string-array name="list_preference_entry_values">
        <item>1</item>
        <item>2</item>
        <item>3</item>
    </string-array>

    <string-array name="multi_select_list_preference_default_value" />

    <string name="pref_title_system_sync_settings">System sync settings</string>

    <!-- Example settings for Notifications -->
    <string name="pref_header_notifications">Notifications</string>

    <string name="pref_title_new_message_notifications">New message notifications</string>

    <string name="pref_title_ringtone">Ringtone</string>
    <string name="pref_ringtone_silent">Silent</string>

    <string name="pref_title_vibrate">Vibrate</string>

    <!-- TODO: Remove or change this placeholder text -->
    <string name="hello_blank_fragment">Hello blank fragment</string>

    <string name="welcom_head_01">Spreadsheets on the go</string>
    <string name="welcom_content_01">Get stuff done with or without an internet connection.</string>
    <string name="welcom_head_02">Share and edit together</string>
    <string name="welcom_content_02">Write on your own or invite more people to contribute.</string>
    <string name="welcom_head_03">Automatically save to the web</string>
    <string name="welcom_content_03">Never lose your progress, so you can keep working from any computer or device.</string>
    <string name="welcom_head_04">Edit Excel spreadsheets</string>
    <string name="welcom_content_04">Open, edit, and save Excel files - all within Sheets.</string>


    <string name="icon_credits">Graphics by &lt;a href="http://www.njcit.cn/">Njcit&lt;/a></string>

    <!--仅作打包-->
    <string name="splash_copyright">Copyright &#169; 2016 FileManager All Rights Reserved\nPowered by Li Weiyong</string>
    <string name="splash_slogan">一切尽在掌握中</string>
    <string name="splash_version">V %1$s</string>
    <string name="drawer_open">打开导航抽屉</string>
    <string name="drawer_close">关闭导航抽屉</string>
    <string name="books">书签</string>
    <string name="icon">图标</string>
    <string name="efn">请输入文件名</string>
    <string name="empty"></string>
    <string name="storage">存储</string>
    <string name="gallery">图库</string>
    <string name="apps">应用管理</string>
    <string name="nav_set">配置与管理</string>
    <string name="bookmanag">书签管理</string>
    <string name="setting">应用设置</string>
    <string name="select">选择项目</string>
    <string name="pressagain">再按一次「返回键」退出</string>
    <string name="open">打开</string>
    <string name="backup">备份</string>
    <string name="uninstall">卸载</string>
    <string name="play">在 Play 商店中查看</string>
    <string name="noplay">未发现应用商店</string>
    <string name="properties">属性</string>
    <string name="copyingapk">将 APK 复制到</string>
    <string name="addbook">添加书签</string>
    <string name="addtobook">添加到书签</string>
    <string name="enterpath">输入路径</string>
    <string name="cancel">取消</string>
    <string name="create">新建</string>
    <string name="success">成功</string>
    <string name="filenotexists">文件不存在</string>
    <string name="error">错误</string>
    <string name="cantlistfiles">非 Root 权限无法列出文件</string>
    <string name="cantsearchfiles">当前路径不支持搜索</string>
    <string name="searchresults">搜索结果</string>
    <string name="itemsselected">项目已选中</string>
    <string name="newhomedirectory">新首页目录</string>
    <string name="rename">重命名</string>
    <string name="renaming">正在重命名</string>
    <string name="save">保存</string>
    <string name="renameerror">重命名时出现问题</string>
    <string name="renamed">重命名成功</string>
    <string name="bookmarksadded">书签已添加</string>
    <string name="msg_bookmark_repeat">书签已存在</string>
    <string name="fileexist">同名文件已存在</string>
    <string name="filecreated">文件已创建成功</string>
    <string name="searchpath">搜索路径</string>
    <string name="search">搜索</string>
    <string name="enterfile">请输入文件名</string>
    <string name="name">名称：</string>
    <string name="date">时间戳：</string>
    <string name="location">路径：</string>
    <string name="total">总计：</string>
    <string name="size">大小：</string>
    <string name="totalitems">项目数：</string>
    <string name="enterzipname">请输入 ZIP 文件名</string>
    <string name="noappfound">找不到可以打开该文件的应用</string>
    <string name="copying">正在复制</string>
    <string name="moving">正在移动</string>
    <string name="loading">正在加载</string>
    <string name="playing">正在播放</string>
    <string name="add">添加</string>
    <string name="tab">标签</string>
    <string name="file">文件</string>
    <string name="folder">文件夹</string>
    <string name="newfolder">新文件夹</string>
    <string name="entername">请输入名称</string>
    <string name="extracting">正在解压</string>
    <string name="extracted">解压完毕</string>
    <string name="zipping">正在压缩</string>
    <string name="stopping">正在停止</string>
    <string name="guide1">欢迎使用 Amaze 文件管理器！</string>
    <string name="guide2">触摸蓝条以显示或隐藏地址栏</string>
    <string name="guide3">长按条目以使用更多选项</string>
    <string name="guide4">例如复制、移动、查看属性等等</string>
    <string name="guide5">按返回键退出应用</string>
    <string name="guide6">此外也可以使用左侧栏进行浏览</string>
    <string name="maintab">无法移除主标签页</string>
    <string name="openparent">打开父目录</string>
    <string name="openwith">打开方式</string>
    <string name="share">分享</string>
    <string name="in_play">内部播放</string>
    <string name="about">关于</string>
    <string name="extract">提取</string>
    <string name="compress">压缩</string>
    <string name="yes">是</string>
    <string name="no">否</string>
    <string name="confirm">确认</string>
    <string name="deleting">正在删除</string>
    <string name="questiondelete">您确定要删除以下文件吗？</string>
    <string name="viewmode">视图模式</string>
    <string name="uimode">界面模式</string>
    <string name="directorysort">目录排序方式</string>
    <string name="sortby">排序方式</string>
    <string name="ascending">升序</string>
    <string name="descending">降序</string>
    <string name="theme">主题</string>
    <string name="skin">皮肤</string>
    <string name="random">随机皮肤</string>
    <string name="random_summary">每次启动都显示随机主题</string>
    <string name="colorize">多彩图标</string>
    <string name="colorize_summary">设置静态图标的颜色</string>
    <string name="back">后退</string>
    <string name="home">首页目录</string>
    <string name="paste">粘贴</string>
    <string name="pasteover">粘贴完成</string>
    <string name="close">关闭</string>
    <string name="history">历史记录</string>
    <string name="refresh">刷新</string>
    <string name="clear">清除</string>
    <string name="copy">复制</string>
    <string name="rootmode">Root 模式</string>
    <string name="rootmodesummary">仅供已获取 Root 权限的设备使用</string>
    <string name="mountsystemsummary">如果您想修改系统文件请启用此选项</string>
    <string name="mountsystem">挂载 System 分区为可读写</string>
    <string name="media_unmounted">存储卡被移除</string>
    <string name="rootfailure">未获取 Root 访问权限</string>
    <string name="cut">剪切</string>
    <string name="selectall">全选</string>
    <string name="delete">删除</string>
    <string name="setashome">设为为首页目录</string>
    <string name="sethome">设置首页目录</string>
    <string name="foldercolor">文件夹颜色</string>
    <string name="imagegallery">图库</string>
    <string name="fileexplorer">文件管理器</string>
    <string name="changegallerypath">选择图库目录</string>
    <string name="usedefaultgallerypath">使用默认图库目录</string>
    <string name="files">文件</string>
    <string name="directories">文件夹</string>
    <!-- New -->
    <!-- Array -->

    <!-- UI -->
    <string name="simple">简洁式</string>
    <string name="card">卡片式</string>
    <!-- Theme -->
    <string name="light">明亮</string>
    <string name="dark">黑暗</string>
    <string name="daytime">随昼夜变化</string>
    <!--Skin-->
    <string name="red">红色</string>
    <string name="pink">粉色</string>
    <string name="purple">淡紫</string>
    <string name="deep_purple">深紫</string>
    <string name="indigo">靛蓝</string>
    <string name="blue">蓝色</string>
    <string name="light_blue">亮蓝</string>
    <string name="cyan">青色</string>
    <string name="teal">蓝绿</string>
    <string name="green">绿色</string>
    <string name="light_green">草绿</string>
    <string name="amber">琥珀</string>
    <string name="orange">橙色</string>
    <string name="deep_orange">暗橙</string>
    <string name="brown">棕色</string>
    <string name="black">黑色</string>
    <string name="blue_grey">蓝灰</string>
    <string name="supersu">超级授权</string>
    <!--DirectorySortMode-->
    <string name="foldersOnTop">文件夹显示在顶部</string>
    <string name="filesOnTop">文件夹显示在底部</string>
    <string name="noneOnTop">文件与文件夹混合显示</string>
    <!--SortBy-->
    <string name="sortName">名称</string>
    <string name="lastModified">日期</string>
    <string name="sortSize">大小</string>
    <string name="sortNameDesc">名称（降序）</string>
    <string name="lastModifiedDesc">日期（降序）</string>
    <string name="sortSizeDesc">大小（降序）</string>
    <!--ListMode-->
    <string name="detailedView">详细</string>
    <string name="simpleView">简洁</string>
    <!--Folder-->
    <string name="Yellow">黄色</string>
    <string name="pick_a_file">选择一个文件</string>
    <string name="process_viewer">进程管理器</string>
    <string name="in_safe">空间不足</string>
    <string name="no_file_overwrite">未覆盖任何文件</string>
    <string name="skip">跳过</string>
    <string name="overwrite">覆盖</string>
    <string name="cant_read_file">无法读取文件</string>
    <string name="not_allowed">不允许的操作</string>
    <string name="processes">进程列表</string>
    <string name="searching">搜索中</string>
    <string name="copying_fles">正在复制文件</string>
    <string name="done">完成</string>
    <string name="Extracting_fles">正在提取文件</string>
    <string name="Zipping_fles">正在创建压缩文件</string>
    <string name="items">个项目</string>
    <string name="file_des">%1$d个文件夹 %2$d个文件</string>
    <string name="fail_count">%1$d个文件粘贴失败</string>
    <string name="set">应用设置</string>
    <string name="enablerootmde">启用 Root 模式</string>
    <string name="goback">返回上一级</string>
    <string name="creatingfolder">创建文件夹</string>
    <string name="foldercreated">文件夹已创建</string>
    <string name="newfile">新文件</string>
    <string name="nofiles">无文件</string>
    <string name="tapnhold">长按文件或文件夹，显示更多选项</string>
    <string name="pathcopied">路径已复制到剪贴板</string>
    <string name="used">已用：</string>
    <string name="free">剩余：</string>
    <string name="atroot">已进入 Root 模式</string>
    <string name="listview">列表模式</string>
    <string name="gridview">网格模式</string>
    <string name="setlistview">设置列表模式</string>
    <string name="setgridview">设置网格模式</string>
    <string name="permission">权限</string>
    <string name="hide">排除</string>
    <string name="bluetooth">蓝牙</string>
    <string name="exit">退出应用</string>
    <string name="enternewname">请输入新的文件名</string>
    <string name="invalid_name">无效的名称</string>
    <string name="doforall">为所有项目执行操作</string>
    <string name="gridcolumnno">网格模式列数</string>
    <string name="randomDialog">随机</string>
    <string name="setRandom">更改将会在您重启应用后生效。</string>
    <string name="authors">开发人员</string>
    <string name="changelog">更新日志</string>
    <string name="fullChangelog">完整更新日志</string>
    <string name="ok">确定</string>
    <string name="chooseUI">选择项目显示模式</string>
    <string name="choose_color">选择颜色</string>
    <string name="primary_color_title">主题颜色</string>
    <string name="primary_color_summary">设置UI的主题颜色</string>
    <string name="showDividers">显示分割线</string>
    <string name="ui">界面</string>
    <string name="general">常规设置</string>
    <string name="thumbSummary">显示应用和图片的缩略图</string>
    <string name="thumb">显示缩略图</string>
    <string name="hidden">显示隐藏文件</string>
    <string name="lastModifiedSummary">显示文件的最后修改时间</string>
    <string name="lastModifiedPref">显示修改时间</string>
    <string name="sizePrefSummary">显示文件大小及文件夹内文件数量</string>
    <string name="sizePref">显示大小</string>
    <string name="miscellaneous">杂项</string>
    <string name="rootPrefSummary">显示文件及文件夹的权限</string>
    <string name="rootPref">显示权限</string>
    <string name="aboutFileManager">关于应用</string>
    <string name="version">应用版本</string>
    <string name="versions">应用版本：V1.0</string>
    <string name="rate">为本应用评分</string>
    <string name="license">开源许可</string>
    <string name="feedback">反馈</string>
    <string name="hiddenfiles">排除文件</string>
    <string name="type">类型</string>
    <string name="typeDesc">类型（降序）</string>
    <string name="rootdirectory">根目录</string>
    <string name="new_string">新建</string>
    <string name="setringtone">设置为铃声</string>
    <string name="copy_path">复制路径</string>
    <string name="addshortcut">创建快捷方式</string>
    <string name="unsavedchanges">未保存修改</string>
    <string name="unsavedchangesdesc">您已经修改了该文件，是否要在退出前保存您所作的修改？</string>
    <string name="saving">保存</string>
    <string name="packageinstaller">应用包安装程序</string>
    <string name="view">查看</string>
    <string name="install">安装</string>
    <string name="archive">压缩包</string>
    <string name="archtext">这是一个压缩包文件，您要执行哪一项操作？</string>
    <string name="pitext">这是一个应用安装包，您要执行哪一项操作？</string>
    <string name="xda_title">XDA 链接</string>
    <string name="xda_summary">获取应用的问答及帮助</string>
    <string name="invalid_dir">无效的目录</string>
    <string name="circularimages">对图像和视频使用圆形图标</string>
    <string name="circularicons">使用圆形图标</string>
    <string name="back_title">显示返回选项</string>
    <string name="back_summary">在列表的顶部显示返回选项</string>
    <string name="details">详细</string>
    <string name="translators">翻译人员</string>
    <string name="german_translation_title">德语</string>
    <string name="italian_translation_title">意大利语</string>
    <string name="french_translation_title">法语</string>
    <string name="russian_translation_title">俄语</string>
    <string name="spanish_translation_title">西班牙语</string>
    <string name="basque_translation_title">巴斯克语</string>
    <string name="chinese_translation_title">简体中文</string>
    <string name="serbian_translation_title">塞尔维亚语</string>
    <string name="turkish_translation_title">土耳其语</string>
    <string name="ukrainian_translation_title">乌克兰语</string>
    <string name="portuguese_translation_title">葡萄牙语</string>
    <string name="polish_translation_title">波兰语</string>
    <string name="dutch_translation_title">荷兰语</string>
    <string name="zip_viewer">ZIP 压缩包查看器</string>
    <string name="unin_system_apk">这是一个系统应用，删除它可能导致系统不稳定。\n是否继续？</string>
    <string name="unin_system_apk_fail">这是一个系统应用，不能卸载！</string>
    <string name="warning">警告</string>
    <string name="donate">捐赠</string>
    <string name="archive_preferences">压缩</string>
    <string name="archive_extract_folder">压缩包提取目录</string>
    <string name="zip_create_folder">ZIP 创建目录</string>
    <string name="archive_summary">压缩包文件将会被提取到该目录，默认提取到压缩包所在目录</string>
    <string name="zip_summary">新 ZIP 文件将会创建到该目录，默认创建压缩包到当前目录</string>
    <string name="openas">打开方式</string>
    <string name="text">文本</string>
    <string name="audio">音频</string>
    <string name="video">视频</string>
    <string name="other">其他</string>
    <string name="image">图像</string>
    <string name="document">文档</string>
    <string name="apk">Apks</string>
    <string name="apks">应用管理</string>
    <string name="quick">书签</string>
    <string name="contributors">捐赠人员</string>
    <string name="savepaths">记住路径</string>
    <string name="savepathsummary">当您开启文件管理器时，将会显示您上次访问的目录</string>
    <string name="select_intent">选择</string>
    <string name="md5copied">MD5 码已复制到剪切板</string>
    <string name="generating">正在生成…</string>
    <string name="tables">表格</string>
    <string name="database">数据库</string>
    <string name="colourednavigation">导航栏变色</string>
    <string name="colourednavigationsum">设置导航栏是否跟随皮肤颜色变色</string>
    <string name="no_newwork">当前没有可用的网络</string>
    <string name="set_newwork">当前没有可用的网络，是否设置？</string>


    <string name="upload_to_baidu">上传百度网盘</string>
    <string name="upload">上传</string>
    <string name="notice">提示</string>
    <string name="baidu_login_success">百度网盘登录成功</string>
    <string name="baidu_login_error">百度网盘登录失败</string>
    <string name="baidu_upload_success">文件上传成功</string>
    <string name="baidu_upload_stard">文件开始上传</string>
    <string name="baidu_upload_error">文件上传失败</string>
    <string name="update_waiting">请稍候... (%s%%)</string>
    <string name="update_title">升级提示</string>
    <string name="update_noticing">当前版本号是：%1$s，代号是：%2$d\n新的版本号是：%3$s，代号是：%4$d\n是否升级？</string>
    <string name="check_update">检查更新</string>
    <string name="update">更新</string>
    <string name="download_fail">下载新版应用失败</string>
    <string name="update_download_title">下载新版应用</string>
    <string name="update_downloading">正在下载应用程序，请稍后...</string>
    <string name="nav_header_desc">文件管理</string>
    <string name="nav_header_title">文件管理</string>
    <string name="menu_home">home</string>
    <string name="menu_gallery">gallery</string>
    <string name="menu_slideshow">slideshow</string>
</resources>
```

app_bar_main.xml中修改AppBarLayout节的内容如下：

```
<com.google.android.material.appbar.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:theme="@style/Theme.FileManage.AppBarOverlay">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        app:popupTheme="@style/Theme.FileManage.PopupOverlay" />
    <HorizontalScrollView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:clickable="false"
        android:paddingBottom="5dp"
        android:paddingLeft="72dp"
        android:paddingTop="5dp"
        android:scrollbars="none">

        <LinearLayout
            android:id="@+id/path_head"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:clickable="false"
            android:orientation="vertical">

            <TextView
                android:id="@+id/fullpath"
                style="@android:style/TextAppearance.Medium"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:clickable="false"
                android:textColor="@android:color/white" />

            <TextView
                android:id="@+id/pathname"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="start"
                android:clickable="false"
                android:paddingBottom="8dp"
                android:paddingTop="5dp"
                android:textColor="@android:color/white"
                android:textSize="12sp" />
        </LinearLayout>
    </HorizontalScrollView>
</com.google.android.material.appbar.AppBarLayout>
```

nav_header_main.xml中设计如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="@dimen/nav_header_height"
    android:background="@drawable/side_nav_bar"
    android:gravity="bottom"
    android:orientation="vertical"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:theme="@style/ThemeOverlay.AppCompat.Dark">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:contentDescription="@string/nav_header_desc"
        android:paddingTop="@dimen/nav_header_vertical_spacing"
        app:srcCompat="@mipmap/ic_launcher_round" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="@dimen/nav_header_vertical_spacing"
        android:text="@string/app_name"
        android:textAppearance="@style/TextAppearance.AppCompat.Body1" />

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/versions" />
</LinearLayout>
```

activity_main_drawer.xml设计如下：

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:showIn="navigation_view">

    <group android:checkableBehavior="single">
        <item
            android:id="@+id/quick"
            android:icon="@mipmap/ic_star_white_18dp"
            android:title="@string/quick" />
        <item
            android:id="@+id/image"
            android:icon="@mipmap/ic_doc_image"
            android:title="@string/image" />
        <item
            android:id="@+id/video"
            android:icon="@mipmap/ic_doc_video_am"
            android:title="@string/video" />
        <item
            android:id="@+id/audio"
            android:icon="@mipmap/ic_doc_audio_am"
            android:title="@string/audio" />
        <item
            android:id="@+id/document"
            android:icon="@mipmap/ic_doc_doc_am"
            android:title="@string/document" />
        <item
            android:id="@+id/apk"
            android:icon="@mipmap/ic_doc_apk_grid"
            android:title="@string/apk" />

    </group>


    <item android:title="@string/nav_set">
        <menu>
            <item
                android:id="@+id/nav_apk"
                android:icon="@mipmap/ic_doc_apk"
                android:title="@string/apps" />
            <item
                android:id="@+id/nav_setting"
                android:icon="@mipmap/ic_settings_grey600_48dp"
                android:title="@string/setting" />
        </menu>
    </item>
</menu>
```

main.xml设计如下：

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/paste"
        android:icon="@mipmap/ic_content_paste_white_36dp"
        android:title="@string/paste"
        android:visible="false"
        app:showAsAction="always" />
    <item
        android:id="@+id/search"
        android:icon="@mipmap/ic_action_search"
        android:title="@string/search"
        app:showAsAction="always" />
    <item
        android:id="@+id/home"
        android:icon="@mipmap/ic_home_white_48dp"
        android:title="@string/home"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/sortby"
        android:icon="@mipmap/ic_sort_white_48dp"
        android:title="@string/sortby"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/view"
        android:title="@string/gridview" />
    <item
        android:id="@+id/exit"
        android:icon="@mipmap/ic_action_cancel_light"
        android:title="@string/exit" />
</menu>
```

最后运行app查看运行结果。

