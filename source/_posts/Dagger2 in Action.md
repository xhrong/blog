---
title: Dagger2 in Action 
tags: [Android, Dagger2, DI]
grammar_cjkRuby: true
categories: [Android]
---

### Dagger2是什么


Dagger是为Android和Java平台提供的一个完全静态的，在编译时进行**依赖注入（DI）的框架**，原来是由Square公司维护，现在由Google维护。

优点是：解耦(DI的特性)，易于测试（DI的特性），高效（不使用反射，google官方说名比Dagger快13%），易混淆（apt方式生成代码，混淆后依然正常使用）

### Dagger2核心概念
**@Inject：** 在需要依赖的地方使用这个注解。换句话说，你用它告诉Dagger这个类或者字段需要依赖注入。这样，Dagger就会构造一个这个类的实例并满足他们的依赖。

**@Module：** Modules类里面的方法专门提供依赖，所以我们定义一个类，用@Module注解，这样Dagger在构造类的实例的时候，就知道从哪里去找到需要的 依赖。modules的一个重要特征是它们设计为分区并组合在一起（比如说，在我们的app中可以有多个组成在一起的modules）。

**@Provide：** 在modules中，我们定义的方法是用这个注解，以此来告诉Dagger我们想要构造对象并提供这些依赖。

**@Component：** Components从根本上来说就是一个注入器，也可以说是@Inject和@Module的桥梁，它的主要作用就是连接这两个部分。 Components可以提供所有定义了的类型的实例，比如：我们必须用@Component注解一个接口然后列出所有的@Modules组成该组件，如 果缺失了任何一块都会在编译的时候报错。所有的组件都可以通过它的modules知道依赖的范围。

**@Scope：** Scopes可是非常的有用，Dagger2可以通过自定义注解限定注解作用域。这是一个非常强大的特点，因为就如前面说的一样，没 必要让每个对象都去了解如何管理他们的实例。在scope的例子中，我们用自定义的@PerActivity注解一个类，所以这个对象存活时间就和 activity的一样。简单来说就是我们可以定义所有范围的粒度(@PerFragment, @PerUser, 等等)。

**@Qualifier：** 当类的类型不足以鉴别一个依赖的时候，我们就可以使用这个注解标示。例如：在Android中，我们会需要不同类型的context，所以我们就可以定义 qualifier注解“@ForApplication”和“@ForActivity”，这样当注入一个context的时候，我们就可以告诉 Dagger我们想要哪种类型的context。

**@SubComponent：** 该注解从名字上就能知道，它是子Component，会完全继承父Component的所有依赖注入对象！

**@Sigleton：** 被注解的对象，在App中是单例存在的！

**@Named：** 因为Dagger2 的以来注入是使用类型推断的，所以同一类型的对象就无法区分，可以使用@Named注解区分同一类型对象，可以理解为对象的别名！

### Dagger2的使用
![enter description here][1]

#### Creating Singletons
![enter description here][2]

整体过程是：
1、创建Module，即依赖注入的提供者，并申明为@Singleton（应用的整个生命周期内单例）
2、创建Compnent，即依赖的注入器，实例的管理都
3、申明需要依赖注入的地方，如Activity内等
4、关联依赖注入关系

```java
@Module
public class AppModule {

    Application mApplication;

    public AppModule(Application application) {
        mApplication = application;
    }

    @Provides
    @Singleton
    Application providesApplication() {
        return mApplication;
    }
}
```
```java
@Module
public class NetModule {

    String mBaseUrl;
    
    // Constructor needs one parameter to instantiate.  
    public NetModule(String baseUrl) {
        this.mBaseUrl = baseUrl;
    }

    // Dagger will only look for methods annotated with @Provides
    @Provides
    @Singleton
    // Application reference must come from AppModule.class
    SharedPreferences providesSharedPreferences(Application application) {
        return PreferenceManager.getDefaultSharedPreferences(application);
    }

    @Provides
    @Singleton
    Cache provideOkHttpCache(Application application) { 
        int cacheSize = 10 * 1024 * 1024; // 10 MiB
        Cache cache = new Cache(application.getCacheDir(), cacheSize);
        return cache;
    }

   @Provides 
   @Singleton
   Gson provideGson() {  
       GsonBuilder gsonBuilder = new GsonBuilder();
       gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
       return gsonBuilder.create();
   }

   @Provides
   @Singleton
   OkHttpClient provideOkHttpClient(Cache cache) {
      OkHttpClient client = new OkHttpClient();
      client.setCache(cache);
      return client;
   }

   @Provides
   @Singleton
   Retrofit provideRetrofit(Gson gson, OkHttpClient okHttpClient) {
      Retrofit retrofit = new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create(gson))
                .baseUrl(mBaseUrl)
                .client(okHttpClient)
                .build();
        return retrofit;
    }
}
```

```java
@Singleton
@Component(modules={AppModule.class, NetModule.class})
public interface NetComponent {
   void inject(MainActivity activity);
   // void inject(MyFragment fragment);
   // void inject(MyService service);
}
```
```java
public class MyApp extends Application {

    private NetComponent mNetComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        
        // Dagger%COMPONENT_NAME%
        mNetComponent = DaggerNetComponent.builder()
                // list of modules that are part of this component need to be created here too
                .appModule(new AppModule(this)) // This also corresponds to the name of your module: %component_name%Module
                .netModule(new NetModule("https://api.github.com"))
                .build();

        // If a Dagger 2 component does not have any constructor arguments for any of its modules,
        // then we can use .create() as a shortcut instead:
        //  mNetComponent = com.codepath.dagger.components.DaggerNetComponent.create();
    }

    public NetComponent getNetComponent() {
       return mNetComponent;
    }
}
```

```java
public class MyActivity extends Activity {
  @Inject OkHttpClient mOkHttpClient;
  @Inject SharedPreferences sharedPreferences;

  public void onCreate(Bundle savedInstance) {
        // assign singleton instances to fields
        // We need to cast to `MyApp` in order to get the right method
        ((MyApp) getApplication()).getNetComponent().inject(this);
    } 
```



  [1]: ./images/QQ%E5%9B%BE%E7%89%8720170331141658.png " "

  [2]: ./images/1490940883681.jpg " "
  
  

### Reference
https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2