# 从前端开发者角度去入门 Android

## html -> xml

Android 目前主流是使用 xml 去配置布局。

### 主流布局方式

1. LinearLayout(线性布局)
2. RelativeLayout(相对布局)
3. FrameLayout（帧布局）
4. ConstraintLayout（约束布局-功能更强大）
5. FlexBoxLayout(不是 Android 自带的，是 google 出的一个第三方布局，前端友好)))

举例：

```xml

<LinearLayout
    android:id="@+id/root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <View xxx/>
    <View xxx/>
</LinearLayout>

```

### 常用控件

1. ImageView（展示图片）
2. TextView（展示文字）
3. Button（按钮）
4. EditText（输入框）
5. ViewPager（类似 swiper）
6. Recyclerview（列表视图,内部实现了复用）
7. Dialog（弹框）

```xml

<LinearLayout
    android:id="@+id/root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

        <TextView
            android:id="@+id/tv_past_exam"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:text="扇贝词典"/>

        <TextView
            android:id="@+id/tv_collins"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:text="柯林斯词典"/>
</LinearLayout>


```

## css -> xml/java code

理解转变：样式->视图属性

样式可以直接在 xml 里边声明，也可以在代码里边设置

```xml
 <TextView
            android:id="@+id/tv_past_exam"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_marginLeft="20dp"
            android:background="@drawable/biz_words_pic_path_1"
            android:gravity="center_vertical"
            android:paddingLeft="5dp"
            android:paddingRight="10dp"
            android:text="扇贝词典"
            android:textColor="@color/biz_words_color_28bea0"
            android:textSize="@dimen/textsize14" />
```

```java
		mCircleView = (CircleImageView) refreshContainer.findViewById(R.id.loading_footer_progress);
		mProgress = new MaterialProgressDrawable(getContext(), mCircleView);
		mProgress.setBackgroundColor(0xFFFAFAFA);
		mProgress.setColorSchemeColors(color);
		mProgress.setAlpha(255);
		mCircleView.setImageDrawable(mProgress);
		mCircleView.setVisibility(GONE);
```

## js -> java code

主要是逻辑代码（kt java 皆可）

## Android 领域知识点

### 项目结构（Android）

推荐使用 pbf(pbl or pbf )

```
├─manifests
    ├─AndroidManifests.xml
├─java
    ├─featureA
    ├─featureB
├─res
    ├─drawable(图片资源文件)
    ├─layout(布局文件)
    ├─values
        ├─colors(颜色)


代码设计可以使用mvp、mvc（没有强制要求）如果有更合理的抽象，推荐使用。


```

### 四大组件

1.  activity（用的最多）
    > 可以理解成 app 里的一个页面
2.  service
    > 后台服务（没有界面）
3.  broadcast receiver
    > 是一种系统通信机制
4.  content provider
    > 一种系统数据访问的方法

### LayoutInflater

它将 xml 转换成视图对象

根据 id 找到视图

```java
TextView tvShanbay = parentView.findViewById(R.id.tv_shanbay)

```

### 线程模型

多线程

线程切换目前使用的是 Rxjava。

> 注：UI 操作只能够在主线程，网络请求一定不能在主线程，耗时操作一般都不会在主线程。

### 存储

1. SharedPreference（小数据存储）
2. File（大数据存储）
3. sqlite（未使用）

### 网络请求

okhttp、retrofit、rxjava（已经高度封装，便于使用）

```java
    // 1. 在API中添加新的api，并且定义模型
	/**
	 * 批量查询派生联想
	 *
	 * @param vocabIds 词汇id列表，格式 "id1,id2,id3....."
	 * @return
	 */
     public interface AffixApi {

	    @GET("/abc/applets/affixes")
	    Observable<List<AffixData>> getAffixes(@Query("vocabulary_ids") String vocabIds);


	    class AffixData extends ApiModel {
	    	public int basewordStatus;
	    	public String vocabularyId;
	    	public List<WordBranchData> wordBranch;
	    	public List<WordTreeData> wordTree;
    	}
     }

    // 2.实现API

    public class AffixApiService extends BaseApiService {

	    private static AffixApiService sInstance;

	    public static synchronized AffixApiService getInstance(Context context) {
	    	if (sInstance == null) {
	    		sInstance = new AffixApiService(SBClient.getInstanceV3(context).getClient().create(AffixApi.class));
	    	}
	    	return sInstance;
    	}

    	private AffixApi mApi;

    	public AffixApiService(AffixApi mApi) {
	    	this.mApi = mApi;
    	}

	    public Observable<List<AffixApi.AffixData>> getAffixes(List<String> vocabIdList) {
    		return mApi.getAffixes(ApiUtil.concatByComma(vocabIdList));
    	}
    }

    // 3.使用
    AffixApiService.getInstance(getContext()).getAffix(mVocabData.id)//调用http请求方法
				.subscribeOn(Schedulers.io())//切换请求线程到io
				.observeOn(AndroidSchedulers.mainThread())//观察在主线程
				.subscribe(new SBRespHandler<List<AffixApi.AffixData>>() {//请求回调
					@Override
					public void onSuccess(List<AffixApi.AffixData> data) {
                                                // 请求成功
                                                render(data)
					}

					@Override
					public void onFailure(RespException e) {
                                                // 请求失败
                                                onNoAffix();
					}
				});

```

### 其他知识点

1. 权限处理
2. 依赖管理
3. 打包
4. 单元测试
5. 性能检测
6. 动画

## Tips

1. 目前没有使用状态管理的框架

2. 要有 OO 的思想

3. Android 是开源的，你可以看到具体的实现逻辑（看源码）
