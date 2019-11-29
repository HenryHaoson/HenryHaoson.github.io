---
layout: post
title: 封装Retrofit
date: 2017-05-07
categories: blog
tags: [android]
description: 封装Retrofit

---
### 用静态工厂来封装retrofit
利用反射机制和静态工厂模式，对retrofit进行简单的封装
```
public class ApiFactory {
    private Retrofit mRetrofit;


    private ApiFactory (){

    };
    public static ApiFactory newInstance(){
        return new ApiFactory();
    }

}
```

利用反射机制，动态获取apiService,
```
 public <T> T createApi(Class<T> clz){
        initRetrofit();
        T api=null;
        try{
            Class<T> clazz=(Class<T>) Class.forName(clz.getName());
            api=mRetrofit.create(clazz);
        }catch (Exception e){
            e.printStackTrace();
        }
        return api;
    }
```
`initRetrofit`对retrofit进行配置，对Rxjava和Gson进行适配。
 注：baseUrl必须以 ' / ' 结尾。
`addCallAdapterFactory`是对R下java进行适配。
`addConverterFactory`对Gson进行适配。
```
private void initRetrofit(){
        mRetrofit=new Retrofit.Builder()
                .baseUrl("http://news-at.zhihu.com/")
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create())
                .client(getOkClient())
                .build();
    }
```
`getOkClient`对okhttpClient进行初始化配置。
详情注释
```
  private OkHttpClient getOkClient(){
        OkHttpClient okHttpClient;
        okhttp3.OkHttpClient.Builder ClientBuilder=new okhttp3.OkHttpClient.Builder();
        ClientBuilder.readTimeout(30, TimeUnit.SECONDS);//读取超时
        ClientBuilder.connectTimeout(10, TimeUnit.SECONDS);//连接超时
        ClientBuilder.writeTimeout(60, TimeUnit.SECONDS);//写入超时
        okHttpClient=ClientBuilder.build();
        return okHttpClient;
    }
```


下面结合Rxjava进行一次使用。
Api:Retrofit基于注解定义的接口
```
public interface ZhihuNewsApi {
    @GET("api/3/news/latest")
    Observable<ZhihuNewsList> getZhihuNews();
}
```
Presenter层对数据的请求和解析。` ZhihuNewsApi api = ApiFactory.newInstance().createApi(ZhihuNewsApi.class);`利用上面的封装方便的获取Service对象。
踩坑日记 : 
1.Rxjava1.x和Rxjava2.x冲突解决：使用Rxjava1.x后改为Rxjava2.x要手动替换包，androidstudio的自动补全包会默认使用Rxjava1.x,会造成冲突。
```
public void getZhihuNewsList() {
        ZhihuNewsApi api = ApiFactory.newInstance().createApi(ZhihuNewsApi.class);
        Observable<ZhihuNewsList> observable = api.getZhihuNews();
        observable.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<ZhihuNewsList>() {
                    @Override
                    public void onError(Throwable e) {
                        Log.e("tag","error"+e.toString());
                        e.printStackTrace();
                    }

                    @Override
                    public void onComplete() {
                        view.loadsuccess(mdata);
                        Log.e("tag","oncompleted");
                    }

                    @Override
                    public void onSubscribe(@NonNull Disposable d) {

                    }

                    @Override
                    public void onNext(ZhihuNewsList zhihuNewsList) {
                        List<ZhihuNewsList.StoriesBean> getlist = zhihuNewsList.getStories();
                        List<ZhihuNewDate> data = new ArrayList<>();
                        for (ZhihuNewsList.StoriesBean bean : getlist) {
                            ZhihuNewDate zhihuNewDate = new ZhihuNewDate();
                            zhihuNewDate.setId(bean.getId() + "");
                            zhihuNewDate.setTitle(bean.getTitle());
                            zhihuNewDate.setPicUrl(bean.getImages().get(0));
                            data.add(zhihuNewDate);
                        }
                        mdata=data;
                        view.loadsuccess(mdata);
                        Log.e("tag","onnext");
                    }
                });

    }
```