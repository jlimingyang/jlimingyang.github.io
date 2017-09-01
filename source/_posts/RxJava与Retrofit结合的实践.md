---
title: RxJava与Retrofit结合的实践
date: 2017-02-03 10:58:33
categories: [RxJava,Retrofit]
tags: [RxJava,Retrofit]
---
# RxJava如何与Retrofit结合

## 准备工作
添加以下依赖
```java
   compile 'io.reactivex:rxandroid:+'
   compile 'io.reactivex:rxjava:+'
   compile 'com.squareup.retrofit2:retrofit:+'
   compile 'com.squareup.retrofit2:converter-gson:+'
   compile 'com.squareup.retrofit2:adapter-rxjava:+'
   compile 'com.google.code.gson:gson:+'
```
## 只使用Retrofit
使用豆瓣电影Top250作为测试接口，api地址：
https://api.douban.com/v2/movie/top250?start=0&count=10

### 封装Entity
根据接口返回的json数据封装Entity

### 创建访问数据的接口
实例代码如下：
```java
public interface MovieService {
  @GET("top250")
  Call<MovieEntity> getTop250(@Query("start") int start,@Query("count") int count);

}
```
### 请求
示例代码：
```Java
/***
    *  只使用Retrofit进行访问
    * @param start 开始的位置
    * @param count 每次获取多少条数据
    */

   private void RetrofitGetMovie(int start,int count)
   {
     String BaseUrl="https://api.douban.com/v2/movie/"; //此处注意链接的结尾

       Retrofit retrofit=new Retrofit.Builder()
               .baseUrl(BaseUrl)
               .addConverterFactory(GsonConverterFactory.create())
               .build();
       MovieService movieService=retrofit.create(MovieService.class);

       Call<MovieEntity> repos=movieService.getTopMovie(start,count);
       repos.enqueue(new Callback<MovieEntity>() {
           @Override
           public void onResponse(Call<MovieEntity> call, Response<MovieEntity> response) {
               for (int i = 0; i < response.body().getSubjects().size(); i++) {
                   Log.d("MainActivity", response.body().getSubjects().get(i).getOriginal_title());
               }
           }
           @Override
           public void onFailure(Call<MovieEntity> call, Throwable t) {

           }
       });

   }
```
 此处的repos.enqueue是使用异步请求。也可以使用repos.execute进行同步请求（值得注意的是这个方法不能直接放在主线程中，否则会有<font color="#ff0000">NetworkOnMainThreadException</font>的异常）


## 加入RxJava

 创建Retrofit的过程变为：
 ```java
 Retrofit retrofit=new Retrofit.Builder()
               .baseUrl(BaseUrl)
               .addConverterFactory(GsonConverterFactory.create())
               .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
               .build();
 ```

修改MovieService
```java

@GET("top250")
   Observable<MovieEntity> RxJavaGetMovie(@Query("start") int start, @Query("count") int count);
```

完整请求代码：
```java

private void RxJavaRetrofitGetMovie(int start,int count)
  {
    String BaseUrl="https://api.douban.com/v2/movie/";
      Retrofit retrofit=new Retrofit.Builder()
              .baseUrl(BaseUrl)
              .addConverterFactory(GsonConverterFactory.create())
              .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
              .build();

      MovieService movieService=retrofit.create(MovieService.class);
      movieService.RxJavaGetMovie(start,count)
             .subscribeOn(Schedulers.io())
              .observeOn(AndroidSchedulers.mainThread())
              .subscribe(new Subscriber<MovieEntity>() {
                  @Override
                  public void onCompleted() {

                  }

                  @Override
                  public void onError(Throwable e) {

                  }

                  @Override
                  public void onNext(MovieEntity movieEntity) {
                      for (int i = 0; i < movieEntity.getSubjects().size(); i++) {
                          Log.d("MainActivity", movieEntity.getSubjects().get(i).getTitle());
                      }
                  }
              });

  }
```

## 封装请求过程

创建一个HttpMethods


## 遇到的问题
- <font color="#ff0000" >java.lang.IllegalArgumentException: Unable to create converter for class cn.shay.rxjavaretrofit.Entity.MovieEntity</font>

<font color="#24ff00">解决方案：</font>
在Retrofit 1.9中，GsonConverter 包含在了package 中而且自动在RestAdapter创建的时候被初始化。这样来自服务器的son结果会自动解析成定义好了的Data Access Object（DAO）
但是在Retrofit 2.0中，Converter 不再包含在package 中了。你需要自己插入一个Converter 不然的话Retrofit 只能接收字符串结果。同样的，Retrofit 2.0也不再依赖于Gson 。
如果你想接收json 结果并解析成DAO，你必须把Gson Converter 作为一个独立的依赖添加进来,然后使用addConverterFactory把它添加进来。
