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

由于用到AsyncTask类中的静态成员THREAD_POOL_EXECUTOR，所以需要在import列表的最后一行，public class  MainActivity之前加入如下一句：

```java
import static android.os.AsyncTask.THREAD_POOL_EXECUTOR;
```



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





#### 10. 修改MainActivity类，响应菜单事件

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



### 5.3 OptionsMenu及相关设计



#### 1. 添加资源文件arrays.xml用于显示各种列表信息

在res->values下添加Values Resource File文件，文件名称为arrays.xml，文件内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>

<resources
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="MissingTranslation">
    <string-array name="uimode">
        <item>@string/simple</item>
        <item>@string/card</item>

    </string-array>

    <string-array name="theme">
        <item>@string/light</item>
        <item>@string/dark</item>
        <item>@string/daytime</item>

    </string-array>

    <string-array name="columns">
        <item>2</item>
        <item>3</item>
        <item>4</item>
        <item>5</item>
        <item>6</item>
    </string-array>

    <string-array name="directorysortmode">
        <item>@string/foldersOnTop</item>
        <item>@string/filesOnTop</item>
        <item>@string/noneOnTop</item>
    </string-array>
    <string-array name="sortby">
        <item>@string/sortName</item>
        <item>@string/sortSize</item>
        <item>@string/type</item>
        <item>@string/lastModified</item>
    </string-array>

    <string-array name="sortbyApps">
        <item>@string/sortName</item>
        <item>@string/lastModified</item>
        <item>@string/sortSize</item>
    </string-array>

    <string-array name="ints2">
        <item>0</item>
        <item>1</item>
        <item>2</item>
        <item>3</item>
        <item>4</item>
        <item>5</item>
        <item>6</item>
        <item>7</item>
    </string-array>
    <string-array name="ints1">
        <item>0</item>
        <item>1</item>
        <item>2</item>
    </string-array>
    <string-array name="listmode">
        <item>@string/detailedView</item>
        <item>@string/simpleView</item>
    </string-array>
    <string-array name="folder">
        <item>@string/orange</item>
        <item>@string/Yellow</item>
        <item>@string/blue</item>
    </string-array>

    <string-array name="ints">
        <item>0</item>
        <item>1</item>
    </string-array>

</resources>
```



#### 2.修改main.xml

修改main.xml实现工具栏的设计。具体代码如下：

```xml
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





#### 2. 在MainFragment类中添加成员currentPath

```java
public static String currentPath;  // 当前路径
```





#### 3. 在asysnctasks包中添加类SearchTask用于文件检索

在asynctasks包中添加类SearchTask，用于文件检索，具体代码为：

```java
public class SearchTask extends AsyncTask<String, Integer, Void> {

    private Context c;
    private MainFragment mainFragment;
    private String mInput;

    // 文件夾集合
    List<FileBean> data_dir = new ArrayList<FileBean>();
    // 文件集合
    List<FileBean> data_file = new ArrayList<FileBean>();

    public SearchTask(Context c, MainFragment mainFragment, String key) {
        this.mainFragment = mainFragment;
        this.c = c;
        this.mInput = key;
    }

    @Override
    protected void onPreExecute() {
        if (mainFragment != null && mainFragment.mSwipeRefreshLayout != null) {
            mainFragment.mSwipeRefreshLayout.setRefreshing(true);
        }

    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        super.onProgressUpdate(values);
    }

    @Override
    protected Void doInBackground(String... params) {
        search(params[0], mInput);
        return null;
    }

    @Override
    public void onPostExecute(Void c){
        // 对搜索结果按照文件名排序
        Collections.sort(data_dir, new FileListSorter(1));
        Collections.sort(data_file, new FileListSorter(1));
        // 分别将文件夹和排序后的文件放置到文件集合
        mainFragment.files.addAll(data_dir);
        mainFragment.files.addAll(data_file);

        mainFragment.folder_count=data_dir.size();
        mainFragment.file_count=data_file.size();

        mainFragment.refreshViews();
        mainFragment.mSwipeRefreshLayout.setRefreshing(false);
        mainFragment.mSwipeRefreshLayout.setEnabled(true);

        if(data_dir.size()+data_file.size()==0)
            Toast.makeText(mainFragment.getActivity(), R.string.filenotexists,Toast.LENGTH_SHORT).show();
    }



    /**
     * Recursively search for occurrences of a given text in file names and publish the result
     * @param path the current path
     * @param query the searched text
     */
    private void search(String path, String query) {

        File root_dir = new File(path);
        String[] list = root_dir.list();

        if (list != null && root_dir.canRead()) {
            int len = list.length;

            for (int i = 0; i < len; i++) {
                File check = new File(path + "/" + list[i]);
                String name = check.getName();

                if (check.isFile()
                        && name.toLowerCase().contains(query.toLowerCase())) {
                    FileBean fileBean = new FileBean();
                    fileBean.setFile(check);
                    fileBean.setName(check.getName());
                    fileBean.setPath(check.getPath());
                    fileBean.setImageId(Icons.loadMimeIcon(c, check.getPath(), !mainFragment.IS_LIST, c.getResources()));
                    fileBean.setFileSize(FileHelper.getFileSize(check));
                    fileBean.setFileType("2");
                    data_file.add(fileBean);

                } else if (check.isDirectory()) {
                    if (name.toLowerCase().contains(query.toLowerCase())) {
                        FileBean fileBean = new FileBean();
                        fileBean.setFile(check);
                        fileBean.setName(check.getName());
                        fileBean.setPath(check.getPath());
                        fileBean.setImageId(c.getResources().getDrawable(R.mipmap.ic_grid_folder_new));
                        fileBean.setFileSize(check.listFiles().length + c.getResources().getString(R.string.items));
                        fileBean.setFileType("1");
                        data_dir.add(fileBean);
                    }

                    if (!check.getName().startsWith("."))
                        search(check.getPath(), query);
                }
            }
        }
    }

}
```







#### 4. 在Filehelper中添加showSearchDialog方法用于搜索

在Filehelper文件中，添加静态方法showSearchDialog，显示搜索对话框，用户可以属于要搜索的文件名称。

```java
public static void showSearchDialog(final MainFragment m) {
    MaterialDialog dialog = new MaterialDialog(m.getActivity(),MaterialDialog.getDEFAULT_BEHAVIOR());
    dialog.title(R.string.search,"搜索");
    DialogInputExtKt.input(dialog, null, R.string.efn, null, null,
            InputType.TYPE_CLASS_TEXT | InputType.TYPE_TEXT_FLAG_CAP_WORDS ,
            null, true, false, (materialDialog, text) -> {
                m.files.clear();//删除集合中原有的数据
                SearchTask loadList = new SearchTask(m.getActivity(), m, text.toString());
                loadList.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, m.currentPath);
                return null;
            });

    dialog.positiveButton(R.string.search,"搜索",null);
    dialog.negativeButton(R.string.cancel,"撤销",null);

    dialog.show();
}
```



#### 6. 在类MainActvity中，实现onOptionsItemSelected回调。

```
public boolean onOptionsItemSelected(MenuItem item) {
    if (mainFragment.mActionMode != null) {
        mainFragment.mActionMode.finish();
        return true;
    }

    Fragment fragment = getSupportFragmentManager().findFragmentById(R.id.nav_host_fragment);
    if (fragment instanceof AppsListFragment) {
        FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
        transaction.replace(R.id.nav_host_fragment, mainFragment);
        transaction.commit();
        toolbar.setTitle(R.string.app_name);
        pathHead.setVisibility(View.VISIBLE);
    }

    int id = item.getItemId();

    //noinspection SimplifiableIfStatement
    switch (id) {

        case R.id.home:
            if (mainFragment != null)
                mainFragment.loadFileList(rootPath);
            break;
        case R.id.exit:
            finish();
            break;
        case R.id.sortby:
            //todo:排序功能
            break;
        case R.id.view:
            if (mainFragment.IS_LIST) {
                mainFragment.IS_LIST = false;
            } else {
                mainFragment.IS_LIST = true;
            }
            if (mainFragment.currentPath.equals("/storage"))
                mainFragment.loadRootFiles();
            else
                mainFragment.loadFileList(mainFragment.currentPath);
            break;

        case R.id.search:
            if (mainFragment.currentPath.equalsIgnoreCase("/storage") || rootPath.equalsIgnoreCase("/storage/emulated")) {
                Toast.makeText(getApplicationContext(), R.string.cantsearchfiles, Toast.LENGTH_SHORT).show();
                break;
            }
            if (mainFragment != null)
                fileHelper.showSearchDialog(mainFragment);
            break;
        case R.id.paste:
            //粘贴
            break;
    }
    return super.onOptionsItemSelected(item);
}
```



### 5.4 使用ActionMode设计动作栏-查看文件属性

#### 1. 创建action_mode_menu

右键menu->New->Menu Resource File，文件名为action_mode_menu.xml。具体内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/cpy"
        android:title="@string/copy"
        android:icon="@mipmap/ic_content_copy_white_36dp"
        app:showAsAction="always" />
    <item
        android:id="@+id/delete"
        android:title="@string/delete"
        android:icon="@mipmap/ic_delete_white_36dp"
        app:showAsAction="always" />
    <item
        android:id="@+id/cut"
        android:title="@string/cut"
        android:icon="@mipmap/ic_content_cut_white_36dp"
        app:showAsAction="always" />
    <item
        android:id="@+id/rename"
        android:title="@string/rename" />
    <item
        android:id="@+id/properties"
        android:title="@string/properties" />
    <item
        android:id="@+id/books"
        android:title="@string/addtobook" />
    <item
        android:id="@+id/all"
        android:title="@string/selectall"
        android:icon="@mipmap/ic_select_all_white_36dp"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/share"
        android:title="@string/share" />
    <item
        android:id="@+id/upload"
        android:title="@string/upload_to_baidu" />
    <item
        android:id="@+id/in_play"
        android:title="@string/in_play"
        android:visible="false"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/extract"
        android:title="@string/extract"
        android:visible="false"/>
    <item
        android:id="@+id/addshortcut"
        android:title="@string/addshortcut" />
</menu>
```



#### 2. 创建layout文件actionmode.xml

右键res->layout，然后New->New Layout Resource File，文件名为actionmode.xml。具体代码如下：

```java
<?xml version="1.0" encoding="utf-8"?>

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/item_count"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_marginLeft="16dp"
        android:gravity="start|center_vertical"
        android:textColor="@android:color/white"
        android:textSize="20sp" />
</RelativeLayout>
```



#### 3. 修改RecyclerAdapter

在RecylerAdapter中添加方法，具体代码如下：

```java
public ArrayList<Integer> getCheckedItemPositions() {
    ArrayList<Integer> checkedItemPositions = new ArrayList<Integer>();

    for (int i = 0; i < myChecked.size(); i++) {
        if (myChecked.get(i)) {
            (checkedItemPositions).add(i);
        }
    }

    return checkedItemPositions;
}

public boolean areAllChecked(String path) {
        boolean b = true;
        int a;
        if (path.equals("/storage") ) a = 0;
        else a = 1;
        for (int i = a; i < myChecked.size(); i++) {
            if (!myChecked.get(i)) {
                b = false;
            }
        }
        return b;
}

public void toggleChecked(boolean b, String path) {
        int a;
        if (path.equals("/storage") ) a = 0;
        else a = 1;
        for (int i = a; i < items.size(); i++) {
            myChecked.put(i, b);
            notifyItemChanged(i);
        }
        if (mainFragment.mActionMode != null)
            mainFragment.mActionMode.invalidate();
        if (getCheckedItemPositions().size() == 0) {
            mainFragment.selection = false;
            if (mainFragment.mActionMode != null)
                mainFragment.mActionMode.finish();
            mainFragment.mActionMode = null;
        }
    }

public void toggleChecked(boolean b) {
        int a = 0;
        for (int i = a; i < items.size(); i++) {
            myChecked.put(i, b);
            notifyItemChanged(i);
        }

        if (mainFragment.mActionMode != null) mainFragment.mActionMode.invalidate();
        if (getCheckedItemPositions().size() == 0) {
            mainFragment.selection = false;
            if (mainFragment.mActionMode != null)
                mainFragment.mActionMode.finish();
            mainFragment.mActionMode = null;
        }
    }

public void toggleChecked(int position, ImageView imageView) {
        if (myChecked.get(position)) {
            // if the view at position is checked, un-check it
            myChecked.put(position, false);
        } else {
            // if view is un-checked, check it
            myChecked.put(position, true);
        }

        notifyDataSetChanged();
        //notifyItemChanged(position);
        if (mainFragment.selection == false || mainFragment.mActionMode == null) {
            mainFragment.selection = true;
            mainFragment.mActionMode = mainFragment.MAIN_ACTIVITY.startSupportActionMode(mainFragment.mActionModeCallback);

        }
        if (mainFragment.mActionMode != null && mainFragment.selection)
            mainFragment.mActionMode.invalidate();
        if (getCheckedItemPositions().size() == 0) {
            mainFragment.selection = false;
            mainFragment.mActionMode.finish();
            mainFragment.mActionMode = null;
        }
    }
```



#### 4. 添加显示文件属性对话框自定义布局文件properties_dialog.xml

右键Res->Layout，New->Layout Resource File，取名properties_dialog.xml。具体代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="center_vertical"
        android:gravity="center_vertical"
        android:minHeight="100dp"
        android:orientation="vertical"
        android:paddingTop="5dp">

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:fontFamily="sans-serif-medium"
                android:text="@string/name" />

            <TextView
                android:id="@+id/t5"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingLeft="3dp" />
        </LinearLayout>

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:fontFamily="sans-serif-medium"
                android:text="@string/location" />

            <TextView
                android:id="@+id/t6"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingLeft="3dp" />
        </LinearLayout>

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">


            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:fontFamily="sans-serif-medium"
                android:text="@string/size" />

            <TextView
                android:id="@+id/t7"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingLeft="3dp" />
        </LinearLayout>

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:fontFamily="sans-serif-medium"
                android:text="@string/date" />

            <TextView
                android:id="@+id/t8"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingLeft="3dp" />
        </LinearLayout>
    </LinearLayout>

    <ImageView
        android:id="@+id/divider"
        android:layout_width="match_parent"
        android:layout_height="9dp"
        android:background="@android:color/transparent"
        android:paddingTop="8dp"
        android:src="@color/divider" />

    <LinearLayout
        android:id="@+id/dirprops"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="5dp"
        android:orientation="horizontal">

        <pub.renge.filemanage.ui.views.SizeDrawable
            android:id="@+id/sizedrawable"
            android:layout_width="40dp"
            android:layout_height="match_parent"
            android:layout_gravity="center_horizontal" />

        <TableLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center_horizontal"
            android:gravity="center_horizontal"
            android:minHeight="100dp">

            <TableRow>

                <ImageButton
                    android:layout_width="30dp"
                    android:layout_height="30dp"
                    android:clickable="false"
                    android:focusable="false"
                    android:minWidth="10dp"
                    android:padding="5dp"
                    android:src="@android:color/transparent" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:fontFamily="sans-serif-medium"
                    android:text="@string/total" />

                <TextView
                    android:id="@+id/t1"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:layout_marginLeft="10dp"
                    android:padding="3dp" />

            </TableRow>

            <TableRow>

                <ImageButton
                    android:layout_width="30dp"
                    android:layout_height="30dp"
                    android:clickable="false"
                    android:focusable="false"
                    android:minWidth="10dp"
                    android:padding="5dp"
                    android:src="#4CAF50" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:fontFamily="sans-serif-medium"
                    android:text="@string/free" />

                <TextView
                    android:id="@+id/t2"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:layout_marginLeft="10dp"
                    android:padding="3dp"/>

            </TableRow>

            <TableRow>

                <ImageButton
                    android:layout_width="30dp"
                    android:layout_height="30dp"
                    android:clickable="false"
                    android:focusable="false"
                    android:minWidth="10dp"
                    android:padding="5dp"
                    android:src="#E53930" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:fontFamily="sans-serif-medium"
                    android:text="@string/used" />

                <TextView
                    android:id="@+id/t3"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:layout_marginLeft="10dp"
                    android:padding="3dp"/>

            </TableRow>

            <TableRow>

                <ImageView
                    android:layout_width="30dp"
                    android:layout_height="30dp"
                    android:clickable="false"
                    android:focusable="false"
                    android:minWidth="10dp"
                    android:padding="5dp"
                    android:src="#0D47A1" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:fontFamily="sans-serif-medium"
                    android:text="@string/size" />

                <TextView
                    android:id="@+id/t4"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:layout_marginLeft="10dp"
                    android:padding="3dp" />

            </TableRow>
        </TableLayout>

    </LinearLayout>

    <Button
        android:id="@+id/appX"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/permission"
        android:visibility="gone" />

    <Button
        android:id="@+id/set"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/set"
        android:visibility="gone" />
</LinearLayout>
```



#### 5. 修改FileHelper

修改FileHelper添加如下方法。

```java
public static long folderSize(File directory) {
    long length = 0;
    try {
        for (File file : directory.listFiles()) {

            if (file.isFile())
                length += file.length();
            else
                length += folderSize(file);
        }
    } catch (Exception e) {
    }
    return length;
}

public static long[] getSpaces(FileBean fileBean) {
    if (fileBean.getFile().isDirectory()) {
        try {
            File file = new File(fileBean.getPath());
            long[] ints = new long[]{file.getTotalSpace(), file.getFreeSpace(), folderSize
                    (new File(fileBean.getPath()))};
            return ints;
        } catch (Exception e) {
            return new long[]{-1, -1, -1};
        }
    }
    return new long[]{-1, -1, -1};
}

public static String readableFileSize(long size) {
        if (size <= 0)
            return "0 B";
        final String[] units = new String[]{"B", "KB", "MB", "GB", "TB"};
        int digitGroups = (int) (Math.log10(size) / Math.log10(1024));
        return new DecimalFormat("#,##0.##").format(size
                / Math.pow(1024, digitGroups))
                + "" + units[digitGroups];
}
```



#### 6. 在ui->views下添加类CircleAnimation类

在ui->views下添加类CircleAnimation，包名为pub.renge.filemanage.ui.views。具体代码如下：

```java
public class CircleAnimation extends Animation {

    private SizeDrawable circle;

    private float newAngle2;
    private float newAngle;
    private float newAngle1;
    float p1,p2,p3;
    public CircleAnimation(SizeDrawable circle, float newAngle, Float secondAngle) {
        this.newAngle1 = newAngle;
        this.newAngle = secondAngle;
        newAngle2=360;
        this.circle = circle;
        p2=-90+this.newAngle;
        p1=-90;p3=-90+newAngle1;
    }

    @Override
    protected void applyTransformation(float interpolatedTime, Transformation transformation) {

        float angle = (interpolatedTime)*360;
        if(angle<newAngle){
            circle.setAngle(interpolatedTime, newAngle / 360);
        }else if(angle<newAngle1){
            circle.setAngle( newAngle/360,newAngle/360);
            circle.setAngle1(interpolatedTime,newAngle1/360);
        }else {
            circle.setAngle( newAngle/360,newAngle/360);
            circle.setAngle1(newAngle1 / 360, newAngle1 / 360);
            circle.setAngle2(interpolatedTime,newAngle2/360);
        }
        /*if(angle<newAngle){
            circle.setAngle( angle,p1);
        }else if(angle<newAngle1){
            circle.setAngle( newAngle,p1);
            circle.setAngle1(angle-newAngle,p2);
        }else {
            circle.setAngle(newAngle,p1);
            circle.setAngle1(newAngle1-newAngle,p2);
            circle.setAngle2(angle-newAngle1,p3);
        }*/
        circle.requestLayout();
    }
}
```



#### 6. 在ui->views下添加类SizeDrawable类

在ui->views下添加类SizeDrawable，包名为pub.renge.filemanage.ui.views。具体代码如下：

```java
public class SizeDrawable extends View {
    Paint mPaint, mPaint1,mPaint2;
    RectF rectF;
    float startangle=-90,angle = 0;
    float startangle1=-90,angle1 = 0;

    float startangle2=-90,angle2=0;

    public SizeDrawable(Context context) {
        super(context);
    }


    int twenty;


    public SizeDrawable(Context context, AttributeSet attributeSet) {
        super(context, attributeSet);
        int strokeWidth = dpToPx(40);
        rectF=new RectF(getLeft(),getTop(),getRight(),getBottom());
        //rectF = new RectF(dpToPx(0), dpToPx(0), dpToPx(200), dpToPx(200));
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setColor(ContextCompat.getColor(getContext(), R.color.accent_indigo));
       // mPaint.setStrokeCap(Paint.Cap.BUTT);
        mPaint.setStrokeWidth(strokeWidth);
        mPaint1 = new Paint();
        mPaint1.setAntiAlias(true);
        mPaint1.setStyle(Paint.Style.FILL);
        mPaint1.setColor(ContextCompat.getColor(getContext(), R.color.accent_red));
      //  mPaint1.setStrokeCap(Paint.Cap.BUTT);
        mPaint1.setStrokeWidth(strokeWidth);
        mPaint2 = new Paint();
        mPaint2.setAntiAlias(true);
        mPaint2.setStyle(Paint.Style.FILL);
        mPaint2.setColor(ContextCompat.getColor(getContext(), R.color.accent_green));
       // mPaint2.setStrokeCap(Paint.Cap.BUTT);
        mPaint2.setStrokeWidth(strokeWidth);
        twenty = dpToPx(10);
    }


    DisplayMetrics displayMetrics;


    public int dpToPx(int dp) {
        if (displayMetrics == null) displayMetrics = getResources().getDisplayMetrics();
        int px = Math.round(dp * (displayMetrics.xdpi / DisplayMetrics.DENSITY_DEFAULT));
        return px;
    }


    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

       // canvas.drawLine((getWidth() - twenty)-2,0,(getWidth() - twenty),0,mPaint1);
        if(angle2!=0)canvas.drawLine(0,getHeight()- (getHeight() * angle1),0,getHeight()-(getHeight()*angle2),mPaint2);
        canvas.drawLine(0, getHeight(),0, getHeight()- (getHeight() * angle), mPaint);
        if(angle1!=0)canvas.drawLine(0,getHeight()- (getHeight() * angle),0,getHeight()-(getHeight()*angle1),mPaint1);
       /* Paint paint = new Paint();
        paint.setColor(Color.WHITE);
        paint.setStyle(Paint.Style.STROKE);
        paint.setTextSize(20);
        canvas.drawText(Math.round(angle * 100)+"%",(getWidth() - twenty)*angle/2, 25,paint);
        if(angle1>0.20)canvas.drawText(Math.round((angle1-angle)*100)+"%",(getWidth() - twenty)*angle+(getWidth() - twenty)*(angle1-angle)/2, 25,paint);
        if(angle2>0.20)canvas.drawText(Math.round((angle2-angle1)*100)+"%",(getWidth() - twenty)*angle1+(getWidth() - twenty)*(angle2-angle1)/2, 25,paint);


        canvas.drawArc(rectF, startangle, angle, true, mPaint);
        canvas.drawArc(rectF, startangle1, angle1, true, mPaint1);
        canvas.drawArc(rectF, startangle2, angle2, true, mPaint2);
*/
    }

    public void setAngle(float angle,float startangle)
    {
        this.angle = angle;
        this.startangle=startangle;
    }
    public void setAngle1(float angle,float startangle1) {
        this.angle1 = angle;
        this.startangle1=startangle1;
    }
    public void setAngle2(float angle2,float startangle2) {
        this.angle2 = angle2;
        this.startangle2=startangle2;
    }


}
```





#### 7. 在包asynctasks下创建类GetPropertyTask获取文件属性

在asysnctasks下创建类GetPropertyTask，包名pub.renge.filemanage.asynctasks.GetPropertyTask，具体代码如下：

```java
package pub.renge.filemanage.asynctasks;

import android.content.Context;
import android.os.AsyncTask;
import android.view.View;
import android.widget.TextView;

import com.afollestad.materialdialogs.MaterialDialog;


import java.io.FileInputStream;
import java.io.InputStream;
import java.security.MessageDigest;

import pub.renge.filemanage.R;
import pub.renge.filemanage.pub.renge.filemanage.bean.FileBean;
import pub.renge.filemanage.ui.views.CircleAnimation;
import pub.renge.filemanage.ui.views.SizeDrawable;
import pub.renge.filemanage.utils.FileHelper;

public class GetPropertyTask extends AsyncTask<String, String, String> {

    private MaterialDialog a;
    private String name, parent, size, items, date;
    private FileBean f;
    Context c;
    String sizeString;
    View textView;
    SizeDrawable sizeDrawable;
    GetPropertyTask g = this;
    TextView t5, t6, t7, t8;

    public GetPropertyTask(MaterialDialog a, FileBean f, String name, String parent,
                           String size, String items, String date, Context c, final View textView) {
        this.a = a;
        this.c = c;
        this.f = f;
        this.name = name;
        this.parent = parent;
        this.size = size;
        this.items = items;
        this.date = date;
        this.textView = textView;
        this.sizeDrawable = (SizeDrawable) textView.findViewById(R.id.sizedrawable);
        final TextView t1 = (TextView) textView.findViewById(R.id.t1);
        final TextView t2 = (TextView) textView.findViewById(R.id.t2);
        final TextView t3 = (TextView) textView.findViewById(R.id.t3);
        final TextView t4 = (TextView) textView.findViewById(R.id.t4);
        t5 = (TextView) textView.findViewById(R.id.t5);
        t6 = (TextView) textView.findViewById(R.id.t6);
        t7 = (TextView) textView.findViewById(R.id.t7);
        t8 = (TextView) textView.findViewById(R.id.t8);
        if (!f.getFile().isDirectory()) {
            textView.findViewById(R.id.divider).setVisibility(View.GONE);
            textView.findViewById(R.id.dirprops).setVisibility(View.GONE);
        } else {

            new AsyncTask<Void, Void, long[]>() {
                @Override
                protected long[] doInBackground(Void... voids) {
                    long[] longs = FileHelper.getSpaces(g.f);
                    return longs;
                }

                @Override
                protected void onPostExecute(long[] longs) {
                    super.onPostExecute(longs);
                    if (longs[0] != -1 && longs[0] != 0) {
                        float r1 = (longs[0] - longs[1]) * 360 / longs[0];
                        float r2 = (longs[2]) * 360 / longs[0];
                        t1.setText(FileHelper.readableFileSize(longs[0]));
                        t2.setText(FileHelper.readableFileSize(longs[1]));
                        t3.setText(FileHelper.readableFileSize(longs[0] - longs[1] - longs[2]));
                        t4.setText(FileHelper.readableFileSize(longs[2]));

                        CircleAnimation animation = new CircleAnimation(g.sizeDrawable, r1, r2);
                        animation.setDuration(Math.round(r1 * 5));
                        g.sizeDrawable.startAnimation(animation);
                    } else {
                        textView.findViewById(R.id.divider).setVisibility(View.GONE);
                        textView.findViewById(R.id.dirprops).setVisibility(View.GONE);
                    }

                }
            }.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR);
        }
    }

    @Override
    public void onProgressUpdate(String... ab) {
        if (a != null && a.isShowing()) {
            t7.setText(items);
        }
    }

    @Override
    protected void onPreExecute() {
        super.onPreExecute();
        t5.setText(name);
        t6.setText(parent);
        t7.setText(items);
        t8.setText(date);


        //todo:禁用取消按钮
        a.cancelable(false);
        //a.getActionButton(DialogAction.NEGATIVE).setEnabled(false);

    }

    @Override
    protected String doInBackground(String... params) {
        String param = params[0];
        if (f.getFile().isDirectory()) {
            int x = f.getFile().listFiles().length;
            items = x + " " + c.getResources().getString(R.string.items);
        } else {
            items = FileHelper.readableFileSize(f.getFile().length());
        }
        publishProgress("");
        String md5 = "";
        try {
            if (!f.getFile().isDirectory()) md5 = getMD5Checksum(param);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return md5;
    }

    @Override
    protected void onPostExecute(String aVoid) {
        super.onPostExecute(aVoid);
    }

    // see this How-to for a faster way to convert
    // a byte array to a HEX string

    public String getMD5Checksum(String filename) throws Exception {
        byte[] b = createChecksum();
        String result = "";

        for (int i = 0; i < b.length; i++) {
            result += Integer.toString((b[i] & 0xff) + 0x100, 16).substring(1);
        }
        return result;
    }

    public byte[] createChecksum() throws Exception {
        InputStream fis = new FileInputStream(f.getPath());

        byte[] buffer = new byte[8192];
        MessageDigest complete = MessageDigest.getInstance("MD5");
        int numRead;

        do {
            numRead = fis.read(buffer);
            if (numRead > 0) {
                complete.update(buffer, 0, numRead);
            }
        } while (numRead != -1);

        fis.close();
        return complete.digest();
    }
}


```





#### 8.  在FileHelper中添加方法showPropsDialog显示文件属性

在FileHelper中添加方法，实现显示文件属性对话框。具体代码如下：

```java
public static String getdate(long f) {

    SimpleDateFormat sdf = new SimpleDateFormat("MMM dd yyyy | KK:mm a");
    return (sdf.format(f)).toString();
}

public static void showPropsDialog(final FileBean file, final MainFragment c) {

    long last = file.getFile().lastModified();
    String date = getdate(last);
    String items = "", size = "", name, parent;
    name = file.getName();
    parent = file.getFile().getParent();

    MaterialDialog dialog = new MaterialDialog(c.getActivity(),MaterialDialog.getDEFAULT_BEHAVIOR());

    dialog.title(R.string.properties,null);

    View v = c.getActivity().getLayoutInflater().inflate(R.layout.properties_dialog, null);
    AppCompatButton appCompatButton = (AppCompatButton) v.findViewById(R.id.appX);
    appCompatButton.setAllCaps(true);

    DialogCustomViewExtKt.customView(dialog,R.layout.properties_dialog,v,true,true,true,true);


    dialog.positiveButton(R.string.ok,null,null);
    dialog.negativeButton(R.string.cancel,null,null);

    dialog.show();
    new GetPropertyTask(dialog, file, name, parent, size, items, date, c.getActivity
            (), v).execute(file.getPath());
}
```





#### 9. 在MainFragment中实现onAtionItemClicked方法

当我们长按文件时会调取ActionMode菜单，我要相应ActionMode的话，就需要在MainFragment中实现ActionMode.Callback接口。这个实现在MainFragment类的内部，和getStorageDirectories等方法处于同一级别。具体代码如下：

```java
/**
     * 定义Item长按动作栏
     */
    public ActionMode.Callback mActionModeCallback = new ActionMode.Callback() {
        View actionModeView;

        private void hideOption(int id, Menu menu) {
            MenuItem item = menu.findItem(id);
            item.setVisible(false);
        }

        private void showOption(int id, Menu menu) {
            MenuItem item = menu.findItem(id);
            item.setVisible(true);
        }

        // called when the action mode is created; startActionMode() was called
        public boolean onCreateActionMode(ActionMode mode, Menu menu) {
            // Inflate a menu resource providing context menu items
            MenuInflater inflater = mode.getMenuInflater();
            actionModeView = getActivity().getLayoutInflater().inflate(R.layout.actionmode, null);
            mode.setCustomView(actionModeView);

            inflater.inflate(R.menu.action_mode_menu, menu);
            hideOption(R.id.addshortcut, menu);
            hideOption(R.id.share, menu);
            hideOption(R.id.properties, menu);
            hideOption(R.id.rename, menu);
            hideOption(R.id.books, menu);

            mode.setTitle(getActivity().getResources().getString(R.string.select));
            if (Build.VERSION.SDK_INT >= 21) {
                Window window = getActivity().getWindow();
                window.setNavigationBarColor(getActivity().getResources().getColor(android.R.color.black));
            }
            return true;
        }

        // the following method is called each time
        // the action mode is shown. Always called after
        // onCreateActionMode, but
        // may be called multiple times if the mode is invalidated.
        public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
            ArrayList<Integer> positions = adapter.getCheckedItemPositions();
            TextView textView1 = (TextView) actionModeView.findViewById(R.id.item_count);
            textView1.setText(positions.size() + "");
            textView1.setOnClickListener(null);
            mode.setTitle(positions.size() + "");

            if (positions.size() == 1) {
                showOption(R.id.share, menu);
                showOption(R.id.properties, menu);
                showOption(R.id.rename, menu);

                File x = new File(files.get(adapter.getCheckedItemPositions().get(0)).getPath());

                if (Icons.isAudio(x.getPath()))
                    showOption(R.id.in_play, menu);

                if (Icons.isArchive(x.getPath()) || Icons.isApk(x.getPath()))
                    showOption(R.id.extract, menu);

                if (x.isDirectory()) {
                    hideOption(R.id.share, menu);
                    showOption(R.id.books, menu);
                    showOption(R.id.addshortcut, menu);
                    hideOption(R.id.upload, menu);
                }
            } else {
                try {
                    showOption(R.id.share, menu);
                    hideOption(R.id.upload, menu);
                    hideOption(R.id.addshortcut, menu);
                    for (int c : adapter.getCheckedItemPositions()) {
                        File x = new File(files.get(c).getPath());
                        if (x.isDirectory()) {
                            hideOption(R.id.share, menu);
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }

                hideOption(R.id.properties, menu);
                hideOption(R.id.rename, menu);
                hideOption(R.id.books, menu);

            }

            return true; // Return false if nothing is done
        }

        // called when the user selects a contextual menu item
        public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
            ArrayList<Integer> plist = adapter.getCheckedItemPositions();
            switch (item.getItemId()) {
                case R.id.properties:
                    FileBean x = files.get((plist.get(0)));
                    FileHelper.showPropsDialog(x, MainFragment.this);
                    mode.finish();
                    break;
                case R.id.delete:
                    //todo:删除
                    break;
                case R.id.share:
                    ArrayList<File> arrayList = new ArrayList<File>();
                    for (int i : plist) {
                        arrayList.add(files.get(i).getFile());
                    }
                    if (arrayList.size() > 100)
                        Toast.makeText(getActivity(), "Can't share more than 100 files", Toast.LENGTH_SHORT).show();
                    else
                        FileHelper.shareFiles(arrayList, getActivity());

                    mode.finish();
                    break;
                case R.id.upload:
                    //todo:上传
                    break;
                case R.id.all:
                    if (adapter.areAllChecked(currentPath)) {
                        adapter.toggleChecked(false, currentPath);
                    } else {
                        adapter.toggleChecked(true, currentPath);
                    }
                    mode.invalidate();
                    break;
                case R.id.rename:
                    //todo:重命名
                    break;
                case R.id.in_play:
                    //todo:播放
                    break;
                case R.id.extract:
                    //todo:解压
                    break;
                case R.id.books:
                    //todo:书签
                    break;

                case R.id.cpy:
                    //todo:复制
                    mode.finish();
                    break;
                case R.id.cut:
                    //todo:剪切
                    break;
                case R.id.addshortcut:
                    //todo:addshortcut
                    break;
                default:
                    return false;

            }

            return true;
        }


        // called when the user exits the action mode
        public void onDestroyActionMode(ActionMode mode) {
            mActionMode = null;
            selection = false;

            if (!false) adapter.toggleChecked(false, currentPath);//liwy
            else adapter.toggleChecked(false);

        }
    };
```





#### 10. 在MainFragment中修改方法onListItemClicked相应长按事件

在MainFragment中修改方法onListItemClicked来响应长按事件，具体代码如下：

```java
public void onListItemClicked(int position, ImageView imageView) {
    if (position >= files.size()) return;

    //单击返回项
    if (files.get(position).getFileSize().equals(goback)) {
        if (selection) {
            selection = false;
            if (mActionMode != null)
                mActionMode.finish();
            mActionMode = null;
        } else {
            if (currentPath.equalsIgnoreCase("/storage")) {

            } else if (currentPath.equalsIgnoreCase("/storage/emulated/0") || currentPath.equalsIgnoreCase("/storage/sdcard0") || currentPath.equalsIgnoreCase("/storage/ext_sd")) {
                loadRootFiles();
            } else {
                File f = new File(currentPath);
                currentPath = f.getParentFile().getPath();
                loadFileList(currentPath);
            }
        }

        return;
    }

    if (selection == true) {   // 长按选择状态
        adapter.toggleChecked(position, imageView);
    } else {
        FileBean fileBean = files.get(position);

        if (fileBean.getFile().isDirectory()) {
            currentPath = fileBean.getPath();
            loadFileList(currentPath);
        }
    }
}

```



#### 11. 修改RecyclerAdapter类，实现长按操作

在RecyclerAdapter适配器类中，找到代码所示位置,在，添加长按相应代码。具体如下：

```java
f (mainFragment.IS_LIST) {
    //单击
    holder.rl.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            mainFragment.onListItemClicked(p, holder.checkImageView);
        }
    });

    //长按
    holder.rl.setOnLongClickListener(new View.OnLongClickListener() {
        public boolean onLongClick(View p1) {
            toggleChecked(p, holder.checkImageView);

            return true;
        }
    });
```



### 5.5 使用ActionMode设计动作栏-删除、重命名、解压、复制、剪切

#### 1. 删除文件











