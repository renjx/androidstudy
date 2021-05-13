## task5 菜单和对话框设计



### 5.1 侧滑菜单设计

#### 1 修改菜单资源

修改res->layout->nav_header_main.xml布局文件内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/nav_head"
    android:layout_width="match_parent"
    android:layout_height="@dimen/nav_header_height"
    android:background="@drawable/side_nav_bar"
    android:gravity="bottom"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:theme="@style/ThemeOverlay.AppCompat.Dark">
    <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingTop="@dimen/nav_header_vertical_spacing"
        android:src="@mipmap/ic_launcher" />
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

修改res->menu->activity_main_drawer.xml布局类容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
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

运行程序应该看到如下效果：



### 5.2 显示各类型文件

#### 1. 修改FileHelper类添加isDoc

修改FileHelper类，增加静态方法isDoc，进行文档类型的识别。具体如下：

```java
/**
 * 文档判断
 *
 * @param path
 * @return
 */
public static boolean isDoc(String path) {
    String[] types = new String[]{".pdf", ".xml", ".html", ".asm", ".text/x-asm", ".def", ".in", ".rc",
            ".list", ".log", ".pl", ".prop", ".properties", ".rc",
            ".doc", ".docx", ".msg", ".odt", ".pages", ".rtf", ".txt", ".wpd", ".wps"};

    for (String type : types)
        if (path.endsWith(type))
            return true;

    return false;
}
```



#### 2. 添加类LoadSpecFiles

添加异步任务类LoadSpecFiles，用于获取特别类型的文件比如文档，视频，安装包等。包名为pub.renge.filemanage.asynctasks。代码如下：

```java
public class LoadSpecFiles extends AsyncTask<String, Integer, Void> {

    private Context c;
    private MainFragment mainFragment;
    private int type;
    private FileHelper utils;

    public LoadSpecFiles(Context c, MainFragment mainFragment, int key) {
        this.mainFragment = mainFragment;
        this.c = c;
        this.type = key;

        utils = new FileHelper();
    }

    @Override
    protected void onPreExecute() {
        if (mainFragment != null && mainFragment.mSwipeRefreshLayout != null) {
            mainFragment.mSwipeRefreshLayout.setRefreshing(true);
        }
        mainFragment.fileDes.setText(R.string.loading);

    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        super.onProgressUpdate(values);
    }

    @Override
    protected Void doInBackground(String... params) {
        mainFragment.files.clear();
        switch (type) {
            case 2:
                mainFragment.files.addAll(listImages());
                break;
            case 3:
                mainFragment.files.addAll(listVideos());
                break;
            case 4:
                mainFragment.files.addAll(listAudios());
                break;
            case 5:
                mainFragment.files.addAll(listDocs());
                break;
            case 6:
                mainFragment.files.addAll(listApks());
                break;
            default:
                break;
        }

        return null;
    }

    @Override
    public void onPostExecute(Void c) {

        mainFragment.folder_count = 0;
        mainFragment.file_count = mainFragment.files.size();

        mainFragment.refreshViews();
        mainFragment.mSwipeRefreshLayout.setRefreshing(false);
        mainFragment.mSwipeRefreshLayout.setEnabled(true);

        if (mainFragment.files.size() == 0)
            Toast.makeText(mainFragment.getActivity(), R.string.filenotexists, Toast.LENGTH_SHORT).show();
    }

    ArrayList<FileBean> listImages() {
        ArrayList<FileBean> images = new ArrayList<>();
        final String[] projection = {MediaStore.Images.Media.DATA};
        final Cursor cursor = c.getContentResolver().query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                projection,
                null,
                null,
                null);
        if (cursor.getCount() > 0 && cursor.moveToFirst()) {
            do {
                String path = cursor.getString(cursor.getColumnIndex
                        (MediaStore.Files.FileColumns.DATA));
                File f = new File(path);

                if (!utils.getFileSize(f).equals("")) {
                    FileBean fileBean = new FileBean();
                    fileBean.setFile(f);
                    fileBean.setName(f.getName());
                    fileBean.setPath(f.getPath());
                    fileBean.setImageId(Icons.loadMimeIcon(c, f.getPath(), !mainFragment.IS_LIST, c.getResources()));
                    fileBean.setFileSize(new FileHelper().getFileSize(f));
                    fileBean.setFileType("1");
                    images.add(fileBean);
                }
            } while (cursor.moveToNext());
        }
        cursor.close();
        return images;
    }

    ArrayList<FileBean> listVideos() {
        ArrayList<FileBean> videos = new ArrayList<>();
        final String[] projection = {MediaStore.Images.Media.DATA};
        final Cursor cursor = c.getContentResolver().query(MediaStore.Video.Media
                        .EXTERNAL_CONTENT_URI,
                projection,
                null,
                null,
                null);
        if (cursor.getCount() > 0 && cursor.moveToFirst()) {
            do {
                String path = cursor.getString(cursor.getColumnIndex
                        (MediaStore.Files.FileColumns.DATA));
                File f = new File(path);

                if (!utils.getFileSize(f).equals("")) {
                    FileBean fileBean = new FileBean();
                    fileBean.setFile(f);
                    fileBean.setName(f.getName());
                    fileBean.setPath(f.getPath());
                    fileBean.setImageId(Icons.loadMimeIcon(c, f.getPath(), !mainFragment.IS_LIST, c.getResources()));
                    fileBean.setFileSize(new FileHelper().getFileSize(f));
                    fileBean.setFileType("1");
                    videos.add(fileBean);
                }
            } while (cursor.moveToNext());
        }
        cursor.close();
        return videos;
    }

    ArrayList<FileBean> listAudios() {
        ArrayList<FileBean> songs = new ArrayList<>();
        String selection = MediaStore.Audio.Media.IS_MUSIC + " != 0";
        String[] projection = {
                MediaStore.Audio.Media.DATA
        };

        Cursor cursor = c.getContentResolver().query(
                MediaStore.Audio.Media.EXTERNAL_CONTENT_URI,
                projection,
                selection,
                null,
                null);

        if (cursor.getCount() > 0 && cursor.moveToFirst()) {
            do {
                String path = cursor.getString(cursor.getColumnIndex
                        (MediaStore.Files.FileColumns.DATA));
                File f = new File(path);

                if (!utils.getFileSize(f).equals("")) {
                    FileBean fileBean = new FileBean();
                    fileBean.setFile(f);
                    fileBean.setName(f.getName());
                    fileBean.setPath(f.getPath());
                    fileBean.setImageId(Icons.loadMimeIcon(c, f.getPath(), !mainFragment.IS_LIST, c.getResources()));
                    fileBean.setFileSize(new FileHelper().getFileSize(f));
                    fileBean.setFileType("1");
                    songs.add(fileBean);
                }
            } while (cursor.moveToNext());
        }
        cursor.close();
        return songs;
    }

    ArrayList<FileBean> listDocs() {
        ArrayList<FileBean> docs = new ArrayList<>();
        final String[] projection = {MediaStore.Files.FileColumns.DATA};

        Cursor cursor = c.getContentResolver().query(MediaStore.Files
                        .getContentUri("external"), projection,
                null,
                null, null);
        if (cursor.getCount() > 0 && cursor.moveToFirst()) {
            do {
                String path = cursor.getString(cursor.getColumnIndex
                        (MediaStore.Files.FileColumns.DATA));
                File f = new File(path);

                if (path != null && !utils.getFileSize(f).equals("") && utils.isDoc(path)) {
                    FileBean fileBean = new FileBean();
                    fileBean.setFile(f);
                    fileBean.setName(f.getName());
                    fileBean.setPath(f.getPath());
                    fileBean.setImageId(Icons.loadMimeIcon(c, f.getPath(), !mainFragment.IS_LIST, c.getResources()));
                    fileBean.setFileSize(new FileHelper().getFileSize(f));
                    fileBean.setFileType("1");
                    docs.add(fileBean);
                }
            } while (cursor.moveToNext());
        }
        cursor.close();
        return docs;
    }

    ArrayList<FileBean> listApks() {
        ArrayList<FileBean> apks = new ArrayList<FileBean>();
        final String[] projection = {MediaStore.Files.FileColumns.DATA};

        Cursor cursor = c.getContentResolver().query(MediaStore.Files
                        .getContentUri("external"), projection,
                null,
                null, null);
        if (cursor.getCount() > 0 && cursor.moveToFirst()) {
            do {
                String path = cursor.getString(cursor.getColumnIndex
                        (MediaStore.Files.FileColumns.DATA));
                if (path != null && path.endsWith(".apk")) {
                    File f = new File(path);

                    if (path != null && !utils.getFileSize(f).equals("")) {
                        FileBean fileBean = new FileBean();
                        fileBean.setFile(f);
                        fileBean.setName(f.getName());
                        fileBean.setPath(f.getPath());
                        fileBean.setImageId(Icons.loadMimeIcon(c, f.getPath(), !mainFragment.IS_LIST, c.getResources()));
                        fileBean.setFileSize(new FileHelper().getFileSize(f));
                        fileBean.setFileType("1");
                        apks.add(fileBean);
                    }
                }
            } while (cursor.moveToNext());
        }
        cursor.close();
        return apks;
    }


}
```



#### 3. 修改MainActivity类中程序响应菜单事件

在MainActivity中，添加成员变量

```java
private LoadSpecFiles loadList;
```



在MainActivity中，首先确保MainActivity实现了implements NavigationView.OnNavigationItemSelectedListener

```java
public class MainActivity extends AppCompatActivity
        implements NavigationView.OnNavigationItemSelectedListener
```



然后，实现onNavigationItemSelected方法，可以响应用户点击菜单事件

```java
@Override
public boolean onNavigationItemSelected(@NonNull MenuItem item) {
    if (mainFragment.selection) {
        mainFragment.mActionMode.finish();
    }

    if (loadList != null) loadList.cancel(true);

    mainFragment.files.clear();//删除集合中原有的数据

    // Handle navigation view item clicks here.
    int id = item.getItemId();
		//todo:书签
    if (id == R.id.quick) {
        FileHelper.showBookMarkDialog(this, mainFragment);
    } else if (id == R.id.image) {
        loadList = new LoadSpecFiles(this, mainFragment, 2);
        loadList.executeOnExecutor(THREAD_POOL_EXECUTOR);
    } else if (id == R.id.video) {
        loadList = new LoadSpecFiles(this, mainFragment, 3);
        loadList.executeOnExecutor(THREAD_POOL_EXECUTOR);
    } else if (id == R.id.audio) {
        loadList = new LoadSpecFiles(this, mainFragment, 4);
        loadList.executeOnExecutor(THREAD_POOL_EXECUTOR);
    } else if (id == R.id.document) {
        loadList = new LoadSpecFiles(this, mainFragment, 5);
        loadList.executeOnExecutor(THREAD_POOL_EXECUTOR);
    } else if (id == R.id.apk) {
        loadList = new LoadSpecFiles(this, mainFragment, 6);
        loadList.executeOnExecutor(THREAD_POOL_EXECUTOR);
    } else if (id == R.id.nav_apk) {
        //todo:显示系统各应用
    } else if (id == R.id.nav_setting) {
        //todo:系统设置
    }

    DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
    drawer.closeDrawer(GravityCompat.START);
    return true;
}
```





### 5.2 应用管理

#### 1. 添加文件管理弹出菜单

在res->menu中添加menu resource file，文件名为app_options.xml，具体内容为：

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/open"
        android:title="@string/open" />
    <item
        android:id="@+id/unins"
        android:title="@string/uninstall" />
    <item
        android:id="@+id/properties"
        android:title="@string/properties" />
    <item
        android:id="@+id/play"
        android:title="@string/play" />
    <item
        android:id="@+id/share"
        android:title="@string/share" />
</menu>
```



#### 2. 添加layout文件simplerow.xml

在res->layout下添加布局文件simplerow.xml，具体代码如下：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/second"
    android:layout_width="match_parent"
    android:minHeight="50dip"
    android:layout_height="wrap_content"
    android:clickable="true"
    android:background="?selectableItemBackground"
    android:padding="6dip">

    <ImageView
        android:id="@+id/icon"
        android:padding="10dip"
        android:layout_width="50dip"
        android:layout_height="50dip"
        android:clickable="false"
        android:gravity="center"
        android:layout_gravity="center"
        android:contentDescription="@string/icon" />

    <TextView
        android:id="@+id/firstline"
        android:layout_width="wrap_content"
        android:layout_toRightOf="@id/icon"
        android:layout_height="match_parent"
        android:clickable="false"
        android:paddingTop="13dp"
        android:paddingLeft="15dp"
        android:layout_gravity="center_vertical"
        android:singleLine="true"
        android:textSize="17sp"
        android:gravity="center_vertical"

        />
</RelativeLayout>
```







#### 3. 添加ShareAdapter适配器

添加ShareAdapter类，实现文件分享，包名为pub.renge.filemanage.adapters。具体代码如下：

```java
public class ShareAdapter extends ArrayAdapter<Intent> {

    MaterialDialog b;
    ArrayList<String> labels;
    IconHolder iconHolder;
    ArrayList<Drawable> arrayList;

    public void updateMatDialog(MaterialDialog b) {
        this.b = b;
    }

    public ShareAdapter(Context context, ArrayList<Intent> arrayList, ArrayList<String> labels,
                        ArrayList<Drawable> arrayList1) {
        super(context, R.layout.list_item, arrayList);
        this.labels = labels;
        iconHolder = new IconHolder(context, true, true);
        this.arrayList = arrayList1;
    }

    @Override
    public View getView(final int position, View convertView, ViewGroup parent) {

        LayoutInflater inflater = (LayoutInflater) getContext()
                .getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        View rowView = (View) inflater.inflate(R.layout.simplerow, parent, false);
        TextView a = ((TextView) rowView.findViewById(R.id.firstline));
        ImageView v = (ImageView) rowView.findViewById(R.id.icon);
        if (arrayList.get(position) != null)
            v.setImageDrawable(arrayList.get(position));
        a.setVisibility(View.VISIBLE);
        a.setText(labels.get(position));
        rowView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                if (b != null && b.isShowing()) b.dismiss();
                getContext().startActivity(getItem(position));

            }
        });
        return rowView;
    }
}
```





#### 4. 添加ShareTask类

添加shareTask类实现文件分享，包名pub.renge.filemanage.asynctasks。这是个AsyncTask任务。具体代码如下：

```java
public class ShareTask extends AsyncTask<String, String, Void> {
    private Activity context;
    private ArrayList<Uri> arrayList;
    private ArrayList<Intent> targetShareIntents = new ArrayList<Intent>();
    private ArrayList<String> arrayList1 = new ArrayList<>();
    private ArrayList<Drawable> arrayList2 = new ArrayList<>();

    public ShareTask(Activity context, ArrayList<Uri> arrayList) {
        this.context = context;
        this.arrayList = arrayList;
    }

    @Override
    protected Void doInBackground(String... strings) {
        String mime = strings[0];
        Intent shareIntent = new Intent();
        boolean bluetooth_present = false;
        shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
        shareIntent.setType(mime);
        PackageManager packageManager = context.getPackageManager();
        List<ResolveInfo> resInfos = packageManager.queryIntentActivities(shareIntent, 0);
        if (!resInfos.isEmpty()) {
            for (ResolveInfo resInfo : resInfos) {
                String packageName = resInfo.activityInfo.packageName;
                arrayList2.add(resInfo.loadIcon(packageManager));
                arrayList1.add(resInfo.loadLabel(packageManager).toString());
                if (packageName.contains("android.bluetooth")) bluetooth_present = true;
                Intent intent = new Intent();
                System.out.println(resInfo.activityInfo.packageName + "\t" + resInfo.activityInfo.name);
                intent.setComponent(new ComponentName(packageName, resInfo.activityInfo.name));
                intent.setAction(Intent.ACTION_SEND_MULTIPLE);
                intent.setType(mime);
                intent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, arrayList);
                intent.setPackage(packageName);
                targetShareIntents.add(intent);

            }
        }
        if (!bluetooth_present && appInstalledOrNot("com.android.bluetooth", packageManager)) {
            Intent intent = new Intent();
            intent.setComponent(new ComponentName("com.android.bluetooth", "com.android.bluetooth.opp.BluetoothOppLauncherActivity"));
            intent.setAction(Intent.ACTION_SEND_MULTIPLE);
            intent.setType(mime);
            intent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, arrayList);
            intent.setPackage("com.android.bluetooth");
            targetShareIntents.add(intent);
            arrayList1.add(context.getResources().getString(R.string.bluetooth));
            arrayList2.add(context.getResources().getDrawable(R.mipmap.ic_settings_bluetooth_white_36dp));
        }
        return null;
    }

    private boolean appInstalledOrNot(String uri, PackageManager pm) {
        boolean app_installed;
        try {
            pm.getPackageInfo(uri, PackageManager.GET_ACTIVITIES);
            app_installed = true;
        } catch (PackageManager.NameNotFoundException e) {
            app_installed = false;
        }
        return app_installed;
    }

    @Override
    public void onPostExecute(Void v) {
        if (!targetShareIntents.isEmpty()) {
            MaterialDialog dialog = new MaterialDialog(context,MaterialDialog.getDEFAULT_BEHAVIOR());

            dialog.title(R.string.share,null);

            ShareAdapter shareAdapter = new ShareAdapter(context, targetShareIntents, arrayList1, arrayList2);
            //todo:下面并没有用，需要重写
//            builder.adapter(shareAdapter, new MaterialDialog.ListCallback() {
//                @Override
//                public void onSelection(MaterialDialog materialDialog, View view, int i, CharSequence charSequence) {
//
//                }
//            });
            dialog.negativeButton(R.string.cancel,null,null);

            shareAdapter.updateMatDialog(dialog);
            dialog.show();
        } else {
            Toast.makeText(context, R.string.noappfound, Toast.LENGTH_SHORT).show();
        }
    }
}
```



#### 5. FileHelper中添加静态方法shareFiles

在FileHelper中添加静态方法shareFiles，实现文件分享。具体代码如下：

```java
/**
 * 分享文件
 *
 * @param files
 * @param c
 */
public static void shareFiles(ArrayList<File> files, Activity c) {
    ArrayList<Uri> uris = new ArrayList<Uri>();
    boolean b = true;
    for (File f : files) {
        uris.add(Uri.fromFile(f));
    }

    String mime = MimeTypes.getMimeType(files.get(0));
    if (files.size() > 1)
        for (File f : files) {
            if (!mime.equals(MimeTypes.getMimeType(f))) {
                b = false;
            }
        }

    if (!b || mime == (null))
        mime = "*/*";
    try {
        new ShareTask(c, uris).execute(mime);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



#### 6. FileHelper中添加uninApp卸载应用

在FileHelper类中田间uninApp方法，用于应用的卸载，具体代码如下：

```java
public static boolean uninApp(Context context, String pkg) {

    try {
        Intent intent = new Intent(Intent.ACTION_DELETE);
        intent.setData(Uri.parse("package:" + pkg));
        context.startActivity(intent);
    } catch (Exception e) {
        Toast.makeText(context, "" + e, Toast.LENGTH_SHORT).show();
        e.printStackTrace();
        return false;
    }
    return true;
}
```





#### 7. 添加适配器AppsAdapter用于应用管理

添加类AppsAdapter，包名为pub.renge.filemanage.adapters，用于显示APP列表并响应。具体代码如下：

```java
public class AppsAdapter extends ArrayAdapter<FileBean> {
    Context context;
    AppsListFragment fragment;
    List<FileBean> appList;
    ArrayList<PackageInfo> packageInfoList;
    IconHolder ic;

    public AppsAdapter(Context context, AppsListFragment fragment, int resourceId,
                       List<FileBean> items, ArrayList<PackageInfo> packageInfoList) {
        super(context, resourceId, items);
        this.context = context;
        this.fragment = fragment;
        this.appList = items;
        this.packageInfoList = packageInfoList;

        ic = new IconHolder(context, true, true);

    }

    private class ViewHolder {
        ImageView apkIcon;
        TextView txtTitle;
        TextView txtDesc;
        View rl;
    }

    public View getView(final int position, View convertView, ViewGroup parent) {

        final FileBean rowItem = getItem(position);

        View view;
        final int p = position;
        if (convertView == null) {
            LayoutInflater mInflater = (LayoutInflater) context
                    .getSystemService(Activity.LAYOUT_INFLATER_SERVICE);
            view = mInflater.inflate(R.layout.list_item, null);

            final ViewHolder vholder = new ViewHolder();
            vholder.txtTitle = (TextView) view.findViewById(R.id.firstLine);
            vholder.apkIcon = (ImageView) view.findViewById(R.id.apk_icon);
            vholder.txtDesc = (TextView) view.findViewById(R.id.secondLine);
            vholder.rl = (View) view.findViewById(R.id.second);
            vholder.apkIcon.setVisibility(View.VISIBLE);
            view.findViewById(R.id.generic_icon).setVisibility(View.GONE);
            view.findViewById(R.id.picture_icon).setVisibility(View.GONE);
            view.setTag(vholder);

        } else {
            view = convertView;

        }
        final ViewHolder holder = (ViewHolder) view.getTag();
        holder.apkIcon.setImageDrawable(rowItem.getImageId());
        ic.cancelLoad(holder.apkIcon);
        ic.loadDrawable(holder.apkIcon, (rowItem.getPath()), null);

        holder.txtTitle.setText(rowItem.getName());
        // File f = new File(rowItem.getDesc());
        holder.txtDesc.setText(rowItem.getFileSize());
        holder.rl.setClickable(true);
        holder.rl.setOnClickListener(new View.OnClickListener() {

            public void onClick(View v) {
                onListItemClicked(holder.rl, p);
            }
        });

        return view;
    }

    /**
     * method called when list item is clicked in the adapter
     *
     * @param position the {@link int} position of the list item
     */
    public void onListItemClicked(View v, int position) {
        if (position >= appList.size()) return;

        showPopup(v, position);
    }

    private void showPopup(View view, final int position) {
        PopupMenu popupMenu = new PopupMenu(context, view);
        popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                switch (item.getItemId()) {
                    case R.id.open:
                        Intent i1 = context.getPackageManager().getLaunchIntentForPackage(packageInfoList.get(position).packageName);
                        if (i1 != null)
                            context.startActivity(i1);
                        else
                            Toast.makeText(context, R.string.not_allowed, Toast.LENGTH_LONG).show();
                        return true;
                    case R.id.play:
                        Intent intent1 = new Intent(Intent.ACTION_VIEW);
                        intent1.setData(Uri.parse("market://search?q=pname:" + packageInfoList.get(position).packageName));
                        try {
                            context.startActivity(intent1);
                        } catch (ActivityNotFoundException exception) {
                            Toast.makeText(context, R.string.noplay, Toast.LENGTH_SHORT).show();
                        }
                        return true;
                    case R.id.properties:

                        context.startActivity(new Intent(
                                android.provider.Settings.ACTION_APPLICATION_DETAILS_SETTINGS,
                                Uri.parse("package:" + packageInfoList.get(position).packageName)));
                        return true;
                    case R.id.share:
                        ArrayList<File> arrayList2 = new ArrayList<File>();
                        arrayList2.add(appList.get(position).getFile());
                        int color1 = Color.parseColor("#e91e63");
                        FileHelper.shareFiles(arrayList2, fragment.getActivity());
                        return true;
                    case R.id.unins:
                        final File f1 = appList.get(position).getFile();
                        ApplicationInfo info1 = null;
                        for (PackageInfo info : packageInfoList) {
                            if (info.applicationInfo.publicSourceDir.equals(f1.getPath())) {
                                info1 = info.applicationInfo;
                            }
                        }
                        if ((info1.flags & ApplicationInfo.FLAG_SYSTEM) != 0) {
                            // system package
                            Toast.makeText(context, R.string.unin_system_apk_fail, Toast.LENGTH_SHORT).show();
                        } else {
                            FileHelper.uninApp(context, packageInfoList.get(position).packageName);
                        }
                        return true;
                }
                return false;
            }
        });
        popupMenu.inflate(R.menu.app_options);
        popupMenu.show();
    }
}
```



#### 8. 修改MainActiviy

在MainActivity类中添加成员变量toolbar

```java
public Toolbar toolbar;
```

在MainActivity类的onCreate方法中的setContentView(R.layout.activiy_main)下修改为如下代码：

```java
setContentView(R.layout.activity_main);
toolbar = findViewById(R.id.toolbar);
setSupportActionBar(toolbar);
```



#### 9.添加AppsListFragment显示APP列表

添加Fragment实现APP列表的显示，名称为AppsListFragment，报名为pub.renge.filemanage.fragments。具体代码如下：

```java
public class AppsListFragment extends ListFragment {
    AppsListFragment fragment = this;
    AppsAdapter adapter;
    private MainActivity mainActivity;
    ListView vl;
    public IconHolder iconHolder;
    private ArrayList<PackageInfo> packageInfoList = new ArrayList<>();
    private ArrayList<FileBean> appLists = new ArrayList<FileBean>();
    private PackageManager packageManager;
    Context context;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setHasOptionsMenu(false);

        context = getActivity();
        iconHolder = new IconHolder(getActivity(), true, true);

        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(Intent.ACTION_PACKAGE_ADDED);
        intentFilter.addAction(Intent.ACTION_PACKAGE_REMOVED);
        intentFilter.addDataScheme("package");
        getActivity().registerReceiver(br, intentFilter);

    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        setRetainInstance(true);
        mainActivity = (MainActivity) getActivity();
        mainActivity.toolbar.setTitle(R.string.apps);
        mainActivity.pathHead.setVisibility(View.GONE);
        mainActivity.supportInvalidateOptionsMenu();

        vl = getListView();
        vl.setDivider(null);

        new LoadListTask().execute();

        setHasOptionsMenu(true);
    }


    @Override
    public void onDestroy() {
        super.onDestroy();
        getActivity().unregisterReceiver(br);
    }

    @Override
    public void onResume() {
        super.onResume();

    }

    BroadcastReceiver br = new BroadcastReceiver() {

        @Override
        public void onReceive(Context context, Intent intent) {
            if (intent != null) {
                new LoadListTask().execute();
            }
        }
    };


    class LoadListTask extends AsyncTask<Void, Void, ArrayList<FileBean>> {

        protected ArrayList<FileBean> doInBackground(Void[] p1) {
            try {
                packageManager = context.getPackageManager();
                List<PackageInfo> all_apps = packageManager.getInstalledPackages(PackageManager.GET_META_DATA);
                appLists = new ArrayList<>();
                packageInfoList = new ArrayList<>();
                for (PackageInfo object : all_apps) {
                    File f = new File(object.applicationInfo.publicSourceDir);
                    packageInfoList.add(object);

                    FileBean fileBean = new FileBean();
                    fileBean.setFile(f);
                    fileBean.setName(f.getName());
                    fileBean.setPath(f.getPath());
                    fileBean.setImageId(Icons.loadMimeIcon(context, f.getPath(), false, getResources()));
                    fileBean.setFileSize(new FileHelper().getFileSize(f));
                    fileBean.setFileType("1");
                    appLists.add(fileBean);
                }
            } catch (Exception e) {
                //Toast.makeText(getActivity(), "" + e, Toast.LENGTH_LONG).show();
            }
            return appLists;
        }

        @Override
        protected void onPreExecute() {
            if (appLists != null)
                appLists.clear();

            if (packageInfoList != null)
                packageInfoList.clear();
        }


        @Override
        // Once the image is downloaded, associates it to the imageView
        protected void onPostExecute(ArrayList<FileBean> appList) {
            if (isCancelled()) {
                appList = null;
            }
            try {
                if (appList != null) {
                    adapter = new AppsAdapter(context, fragment, R.layout.list_item, appList, packageInfoList);
                    setListAdapter(adapter);
                }
            } catch (Exception e) {
            }

        }
    }
}
```







#### 10. 修改FileHelper类添加showBookMarkDialog

修改FileHelper类，增加静态方法showBookMarkDialog，显示BookMark。具体如下：

```java
/**
 * 文档判断
 *
 * @param path
 * @return
 */
public static boolean isDoc(String path) {
    String[] types = new String[]{".pdf", ".xml", ".html", ".asm", ".text/x-asm", ".def", ".in", ".rc",
            ".list", ".log", ".pl", ".prop", ".properties", ".rc",
            ".doc", ".docx", ".msg", ".odt", ".pages", ".rtf", ".txt", ".wpd", ".wps"};

    for (String type : types)
        if (path.endsWith(type))
            return true;

    return false;
}
```



修改



#### 11. 修改MainActivity类，响应菜单事件

在MainActivity类->onNavigationItemSelected方法中找到

else if (id == R.id.nav_apk)

让后在其中输入如下代码(中间三句），实现应用的管理。

```java
@Override
public boolean onNavigationItemSelected(@NonNull MenuItem item) {
    ......
    } else if (id == R.id.nav_apk) {
        FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
        transaction.replace(R.id.nav_host_fragment, new AppsListFragment());
        transaction.commit();
    } ......
    return true;
}
```

