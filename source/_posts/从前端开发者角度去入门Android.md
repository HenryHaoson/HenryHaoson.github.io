# 从前端开发者角度去入门 Android

## html -> xml

Android 目前主流是使用 xml 去配置布局。

### 主流布局方式

1. LinearLayout(线性布局)：
2. RelativeLayout(相对布局)
3. FrameLayout（帧布局）
4. ConstraintLayout（约束布局-功能更强大）

举例：

```xml

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
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

```xml

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
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
```

### 存储

1. SharedPreference（小数据存储）
2. File（大数据存储）
3. sqlite（未使用）

### 网络请求

okhttp（已经高度封装，便于使用）

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

    // 2.实现

```

## Tips

1.目前没有使用状态管理的框架

2.要有 OO 的思想
