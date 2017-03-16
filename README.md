# Android之修改用户头像并上传服务器（实现手机拍照和SD卡选择上传）
<div class="edit-box">
				<textarea name="content" class="editor-content"  id="ckeditor" > &lt;p&gt;写了这么多个的APP，最近才把他这个功能写上来，就抽取其中的&lt;span style="color:#800000"&gt;&lt;strong&gt;用户修改头像的相关操作&lt;/strong&gt;&lt;/span&gt;这个功能写了这篇博客，来与大家分享，希望对你有所帮助。&lt;/p&gt; 
&lt;p&gt;&lt;span style="color:#800000"&gt;&lt;span style="color:#000000"&gt;案例包含了:&lt;/span&gt;&lt;/span&gt;&lt;/p&gt; 
&lt;ol&gt; 
 &lt;li&gt;&lt;span style="color:#800000"&gt;&lt;span style="color:#000000"&gt;Xutil图片上传&lt;/span&gt;&lt;/span&gt;&lt;/li&gt; 
 &lt;li&gt;&lt;span style="color:#800000"&gt;&lt;span style="color:#000000"&gt;拍照和SD卡选择&lt;span style="color:#800000"&gt;&lt;span style="color:#000000"&gt;图片&lt;/span&gt;&lt;/span&gt;&lt;/span&gt;&lt;/span&gt;&lt;/li&gt; 
 &lt;li&gt;&lt;span style="color:#800000"&gt;&lt;span style="color:#000000"&gt;图片缓存和界面逻辑处理&lt;/span&gt;&lt;/span&gt;&lt;/li&gt; 
 &lt;li&gt;&lt;span style="color:#800000"&gt;&lt;span style="color:#000000"&gt;图片压缩和图片处理&lt;/span&gt;&lt;/span&gt;&lt;/li&gt; 
 &lt;li&gt;自定义圆形头像&lt;/li&gt; 
&lt;/ol&gt; 
&lt;p&gt;&lt;a href="http://download.csdn.net/detail/dickyqie/9662223" target="_blank" rel="nofollow"&gt;XUtils.Jar 下载&lt;/a&gt;&lt;/p&gt; 
&lt;p&gt;其他图片上传方式请看博客&amp;nbsp; ：&lt;a href="http://www.cnblogs.com/zhangqie/p/6211752.html" rel="nofollow"&gt;Volley-XUtils-OkHttp三种方式实现单张多张图片上传&lt;/a&gt;&lt;/p&gt; 
&lt;p&gt;效果图： （注：&lt;span style="color:#800000"&gt;模拟器没拍照功能，效果图只有SD卡上传&lt;/span&gt;，&lt;span style="color:#800000"&gt;&lt;strong&gt;手机测试拍照上传也是可以的&lt;/strong&gt;&lt;/span&gt;）&lt;/p&gt; 
&lt;p&gt;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp; &lt;img alt="" src="https://static.oschina.net/uploads/img/201703/16174940_LeCK.gif"&gt;&lt;/p&gt; 
&lt;p&gt;代码：&lt;/p&gt; 
&lt;p&gt;MainActivity.java&lt;/p&gt; 
&lt;pre&gt;&lt;code class="language-java"&gt;public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    ZQRoundOvalImageView zqRoundOvalImageView;
    ACache aCache;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        zqRoundOvalImageView = (ZQRoundOvalImageView) findViewById(R.id.my_sign_sub_img);
        zqRoundOvalImageView.setOnClickListener(this);
        findViewById(R.id.my_sign_sub_txt).setOnClickListener(this);
        aCache = ACache.get(MainActivity.this);
        initData();

    }

    private void initData() {
        if (UtilFileDB.SELETEFile(aCache, "stscimage") != null) {
            if (aCache.getAsBitmap("myimg") == null) {
                getImage(UtilFileDB.LOGINIMGURL(aCache));
            } else {
                zqRoundOvalImageView.setImageBitmap(aCache.getAsBitmap("myimg"));
            }
        }
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.my_sign_sub_img:
            case R.id.my_sign_sub_txt:
                Intent intents = new Intent(MainActivity.this, SettingActivity.class);
                startActivityForResult(intents, 1);
                break;
        }
    }

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 1) {
            if (resultCode == 3) {
                getImage(data.getStringExtra("urlimg"));
            } else {
                zqRoundOvalImageView.setImageResource(R.mipmap.zhangwo_hometop1);
            }
        }
    }

    public void getImage(String url) {
        UtilFileDB.DELETEFile(aCache, "myimg");
        OkHttpUtils.get().url(url).tag(this).build().connTimeOut(20000)
                .readTimeOut(20000).writeTimeOut(20000)
                .execute(new BitmapCallback() {
                    @Override
                    public void onError(Call call, Exception e, int id) {
                    }

                    @Override
                    public void onResponse(Bitmap bitmap, int id) {
                        zqRoundOvalImageView.setImageBitmap(bitmap);
                        aCache.put("myimg", bitmap);
                    }
                });
    }
}&lt;/code&gt;&lt;/pre&gt; 
&lt;p&gt;&amp;nbsp;SettingActivity.Java&lt;/p&gt; 
&lt;pre&gt;&lt;code class="language-java"&gt;public class SettingActivity extends AppCompatActivity implements View.OnClickListener {

    String URL = "url";
    TextView homeTopName;
    ZQRoundOvalImageView zqRoundOvalImageView;
    PopupWindow pop;
    LinearLayout ll_popup;
    Intent intent;
    String urlsf = "";
    int img = 1;
    ACache aCache;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.my_setting);
        initView();
    }

    private void initView() {
        homeTopName = (TextView) findViewById(R.id.home_top_name);
        homeTopName.setText("设置");
        aCache = ACache.get(SettingActivity.this);
        zqRoundOvalImageView = (ZQRoundOvalImageView) findViewById(R.id.my_setting_txlehs);
        zqRoundOvalImageView.setOnClickListener(this);
        findViewById(R.id.home_tour_close).setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.my_setting_txlehs:
                showPopupWindow();
                ll_popup.startAnimation(AnimationUtils.loadAnimation(
                        SettingActivity.this, R.anim.activity_translate_in));
                pop.showAtLocation(v, Gravity.BOTTOM, 0, 0);
                break;
            case R.id.home_tour_close:
                intent = new Intent();
                intent.putExtra("urlimg", urlsf);
                setResult(img, intent);
                finish();
                break;

        }
    }

    /****
     * 头像提示框
     */
    public void showPopupWindow() {
        pop = new PopupWindow(SettingActivity.this);
        View view = getLayoutInflater().inflate(R.layout.item_popupwindows,
                null);
        ll_popup = (LinearLayout) view.findViewById(R.id.ll_popup);
        pop.setWidth(ViewGroup.LayoutParams.MATCH_PARENT);
        pop.setHeight(ViewGroup.LayoutParams.WRAP_CONTENT);
        pop.setBackgroundDrawable(new BitmapDrawable());
        pop.setFocusable(true);
        pop.setOutsideTouchable(true);
        pop.setContentView(view);
        RelativeLayout parent = (RelativeLayout) view.findViewById(R.id.parent);
        Button bt1 = (Button) view.findViewById(R.id.item_popupwindows_camera);
        Button bt2 = (Button) view.findViewById(R.id.item_popupwindows_Photo);
        Button bt3 = (Button) view.findViewById(R.id.item_popupwindows_cancel);
        parent.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                pop.dismiss();
                ll_popup.clearAnimation();
            }
        });
        bt1.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                Intent camera = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
                startActivityForResult(camera, 1);
                pop.dismiss();
                ll_popup.clearAnimation();
            }
        });
        bt2.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                Intent picture = new Intent(
                        Intent.ACTION_PICK,
                        android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
                startActivityForResult(picture, 2);
                pop.dismiss();
                ll_popup.clearAnimation();
            }
        });
        bt3.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                pop.dismiss();
                ll_popup.clearAnimation();
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 1 &amp;amp;&amp;amp; resultCode == Activity.RESULT_OK
                &amp;amp;&amp;amp; null != data) {
            String sdState = Environment.getExternalStorageState();
            if (!sdState.equals(Environment.MEDIA_MOUNTED)) {
                return;
            }
            new DateFormat();
            String name = DateFormat.format("yyyyMMdd_hhmmss",
                    Calendar.getInstance(Locale.CHINA)) + ".jpg";
            Bundle bundle = data.getExtras();
            // 获取相机返回的数据，并转换为图片格式
            Bitmap bmp = (Bitmap) bundle.get("data");
            FileOutputStream fout = null;
            String filename = null;
            try {
                filename = UtilImags.SHOWFILEURL(SettingActivity.this) + "/" + name;
            } catch (IOException e) {
            }
            try {
                fout = new FileOutputStream(filename);
                bmp.compress(Bitmap.CompressFormat.JPEG, 100, fout);
            } catch (FileNotFoundException e) {
                showToastShort("上传失败");
            } finally {
                try {
                    fout.flush();
                    fout.close();
                } catch (IOException e) {
                    showToastShort("上传失败");
                }
            }
            zqRoundOvalImageView.setImageBitmap(bmp);
            staffFileupload(new File(filename));
        }
        if (requestCode == 2 &amp;amp;&amp;amp; resultCode == Activity.RESULT_OK
                &amp;amp;&amp;amp; null != data) {
            try {
                Uri selectedImage = data.getData();
                String[] filePathColumns = {MediaStore.Images.Media.DATA};
                Cursor c = this.getContentResolver().query(selectedImage,
                        filePathColumns, null, null, null);
                c.moveToFirst();
                int columnIndex = c.getColumnIndex(filePathColumns[0]);
                String picturePath = c.getString(columnIndex);
                c.close();

                Bitmap bmp = BitmapFactory.decodeFile(picturePath);
                // 获取图片并显示
                zqRoundOvalImageView.setImageBitmap(bmp);
                saveBitmapFile(UtilImags.compressScale(bmp), UtilImags.SHOWFILEURL(SettingActivity.this) + "/stscname.jpg");
                staffFileupload(new File(UtilImags.SHOWFILEURL(SettingActivity.this) + "/stscname.jpg"));
            } catch (Exception e) {
                showToastShort("上传失败");
            }
        }
    }

    public void saveBitmapFile(Bitmap bitmap, String path) {
        File file = new File(path);//将要保存图片的路径
        try {
            BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(file));
            bitmap.compress(Bitmap.CompressFormat.JPEG, 100, bos);
            bos.flush();
            bos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void staffFileupload(File file) {
        if (false) {
            showToastShort("网络未连接");
            return;
        }
        HttpUtils http = new HttpUtils();
        http.send(HttpRequest.HttpMethod.POST, URL, MYUPDATEIMG(file),
                new RequestCallBack&amp;lt;String&amp;gt;() {

                    @Override
                    public void onFailure(HttpException arg0, String arg1) {

                    }

                    @Override
                    public void onSuccess(ResponseInfo&amp;lt;String&amp;gt; arg0) {
                        JSONObject jsonobj;
                        try {
                            jsonobj = new JSONObject(arg0.result.toString());
                            String errno = jsonobj.getString("errno");
                            String error = jsonobj.getString("error");
                            if (errno.equals("0") &amp;amp;&amp;amp; error.equals("success")) {
                                JSONArray jsonarray = jsonobj.getJSONArray("data");
                                JSONObject jsonobjq = jsonarray.getJSONObject(0);
                                urlsf = jsonobjq.getString("url");
                                UtilFileDB.ADDFile(aCache, "stscimage", urlsf);
                                img = 3;
                                showToastShort("头像修改成功");

                            } else {
                                showToastShort("头像修改失败");
                            }
                        } catch (JSONException e) {
                            return;
                        }
                    }

                });

    }

    /***
     * 修改头像
     *
     * @return
     */
    public static final RequestParams MYUPDATEIMG(File file) {
        RequestParams params = new RequestParams();
        params.addBodyParameter("c", "profile");
        params.addBodyParameter("a", "setavatar");
        params.addBodyParameter("uid", "");
        params.addBodyParameter("username", "");
        if (file != null) {
            params.addBodyParameter("filedata", file);
        }
        return params;
    }

    private void showToastShort(String string) {
        Toast.makeText(SettingActivity.this, string, Toast.LENGTH_LONG).show();
    }
}

复制代码&lt;/code&gt;&lt;/pre&gt; 
&lt;p&gt;AndroidManifest.xml权限&lt;/p&gt; 
&lt;pre&gt;&lt;code class="language-html"&gt;    &amp;lt;uses-permission android:name="android.permission.INTERNET" /&amp;gt;
    &amp;lt;uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /&amp;gt;
    &amp;lt;!-- SD卡权限 --&amp;gt;
    &amp;lt;uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" /&amp;gt;
    &amp;lt;!-- 定位 --&amp;gt;
    &amp;lt;!-- 用于进行网络定位 --&amp;gt;
    &amp;lt;uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" &amp;gt;
    &amp;lt;/uses-permission&amp;gt;
    &amp;lt;uses-permission android:name="android.permission.ACCESS_LOCATION_EXTRA_COMMANDS"&amp;gt;
    &amp;lt;/uses-permission&amp;gt;
    &amp;lt;!-- 用于访问wifi网络信息，wifi信息会用于进行网络定位 --&amp;gt;
    &amp;lt;uses-permission android:name="android.permission.ACCESS_WIFI_STATE" &amp;gt;
    &amp;lt;/uses-permission&amp;gt;
    &amp;lt;!-- 这个权限用于获取wifi的获取权限，wifi信息会用来进行网络定位 --&amp;gt;
    &amp;lt;uses-permission android:name="android.permission.CHANGE_WIFI_STATE" &amp;gt;
    &amp;lt;/uses-permission&amp;gt;
    &amp;lt;!-- 用于读取手机当前的状态 --&amp;gt;
    &amp;lt;uses-permission android:name="android.permission.READ_PHONE_STATE" &amp;gt;
    &amp;lt;/uses-permission&amp;gt;
    &amp;lt;!-- 写入扩展存储，向扩展卡写入数据，用于写入缓存定位数据 --&amp;gt;
    &amp;lt;uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" &amp;gt;
    &amp;lt;/uses-permission&amp;gt;&lt;/code&gt;&lt;/pre&gt; 
&lt;p&gt;&amp;nbsp;&lt;/p&gt; </textarea>
			</div>