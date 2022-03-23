##### [Android 代码规范文档](https://github.com/getActivity/AndroidCodeStandard)

#### 常规规范

- 使用 **0px** 代替 **0dp**，这样就可以在获取时避免系统进行换算，提升代码的执行效率。

- 字符串比较，应该用 `"xxx".equals(object)`，而不应该用 `object.equals("xxx")`，因为 **object** 对象可能为空，我们应该把不为空的条件放置在表达式的前面。

- 尽量采用 **switch case** 来判断，如果不能实现则再考虑用 **if else**，因为在多条件下使用 **switch case** 语句判断会更加简洁。

- 严禁用 **switch case** 语句来判断资源 id，因为 Gradle 在 5.0 之后的版本，资源 ID 将不会以常量的形式存在，而 **switch case** 语句只能判断常量，所以不能再继续使用 **switch case** 来判断资源 ID 了。

- 不推荐用 **layout_marginLeft**，而应该用 **layout_marginStart**；不推荐用 **layout_marginRight**，而应该用 **layout_marginEnd**，原因有两个，一个是适配 Android 4.4 **反方向特性**（可在开发者选项中开启），第二个是 XML 布局中使用 **layout_marginLeft** 和 **layout_marginRight** 会有代码警告，**padding** 属性也是同理，这里不再赘述。另外有一点需要注意：严禁 **Left、Right** 属性和 **Start、End** 属性同时使用，两者只能二选一。

- 如果在 **layout_marginStart** 和 **layout_marginEnd** 的值相同的情况下，请替换使用 **layout_marginHorizontal**，而 **layout_marginTop** 和 **layout_marginBottom** 也同理，请替换使用 **layout_marginVertical**，能用一句代码能做的事不要写两句，**padding** 属性也是同理，这里不再赘述。

- **过期** 和 **高版本** 的 API 一定要做对应的兼容（包含 Java 代码和 XML 属性），如果不需要兼容处理的，需要对警告进行抑制。

- 在能满足需求的情况下，尽量用 **invisible** 来代替 **gone**，因为 **gone** 会触发当前整个 View 树进行重新测量和绘制。

- **api** 和 **implementation**，在能满足使用的情况下，优先选用 **implementation**，因为这样可以[减少一些编译时间](https://www.jianshu.com/p/8962d6ba936e)。

- **ListView** 和 **RecyclerView** 都能实现需求的前提下，优先选用 **RecyclerView**，具体原因不解释，大家应该都懂。

- **ScrollView** 和 **NestedScrollView** 都能实现需求的前提下，优先选用 **NestedScrollView**，是因为 **NestedScrollView** 和 **RecyclerView** 支持相互嵌套，而 **ScrollView** 是不支持嵌套滚动的。

- 不能在项目中创建副本文件，例如创建 `HomeActivity2.java`、`home_activity_v2.xml` 类似的副本文件，因为这样不仅会增加项目的维护难度，同时对编译速度也会造成一定的影响，正确的做法应该是在原有的文件基础上面修改，如果出现需求变更的情况，请直接使用 **Git** 或者 **SVN** 进行版本回退。

- 如果一个类不需要被继承，请直接用 **final** 进行修饰，如果一个字段在类初始化过程中已经赋值并且没有地方进行二次赋值，也应当用 **final** 修饰，如果一个字段不需要被外部访问，那么需要用 **private** 进行修饰。

- 每个小组成员应当安装[阿里巴巴代码约束插件](https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines)，并及时地对插件所提示的**代码警告**进行处理或者抑制警告。

- 应用图标应该放在 **mipmap** 目录下，其他图片资源应当放到 **drawable** 目录下，具体原因可以看[谷歌官方文档](https://developer.android.google.cn/guide/topics/resources/providing-resources)对这两个文件夹给出的介绍：

- | 目录       | 资源类型                                                     |
  | ---------- | ------------------------------------------------------------ |
  | `drawable` | 位图文件（`.png`、`.9.png`、`.jpg`、`.gif`）或编译为以下可绘制对象资源子类型的 XML 文件：位图文件九宫格（可调整大小的位图）状态列表形状动画可绘制对象其他可绘制对象请参阅 [Drawable 资源](https://developer.android.google.cn/guide/topics/resources/drawable-resource)。 |
  | `mipmap`   | 适用于不同启动器图标密度的可绘制对象文件。如需了解有关使用 `mipmap` 文件夹管理启动器图标的详细信息，请参阅[管理项目概览](https://developer.android.google.cn/tools/projects#mipmap)。 |

#### 后台接口规范

- 后台返回的 **id 值**，不要使用 **int** 或者 **long** 类型来接收，而应该用 **string** 类型来接收，因为我们不需要对这个 **id 值**进行运算，所以我们不需要关心它是什么类型的。
- 后台返回的**金额数值**应该使用 **String** 来接收，而不能用**浮点数**来接收，因为 **float** 或者 **double** 在数值比较大的时候会容易丢失精度，并且还需要自己手动转换出想要保留的小数位，最好的方式是后台返回什么前端就展示什么，而到了运算的时候，则应该用 **BigDecimal** 类来进行转换和计算，当然金额在前端一般展示居多，运算的情况还算是比较少的。
- 我们在定义后台返回的 Bean 类时，不应当将一些我们没有使用到的字段添加到代码中，因为这样会消耗性能，因为 Gson 是通过**反射**将后台字段赋值到 Java 字段中，所以我们应当避免一些不必要的字段解析，另外臃余的字段也会给我们排查问题造成一定的阻碍。
- 如果后台给定的字段名不符合代码命名的时候，例如当遇到 `student_name` 这种命名时，我们应当使用 Gson 框架中的 **@SerializedName** 注解进行重命名。
- 请求的接口参数和返回字段必须要写上注释，除此之外还应该备注对应的后台接口文档地址，以便我们后续能够更好地进行维护和迭代。
- 后台返回的 Bean 类字段不能直接访问，而应该通过生成 **Get** 方法，然后使用这个 **Get** 方法来访问字段。
- 接口请求成功的提示可以不显示，但请求失败的提示需要显示给到用户，否则会加大排查问题的难度，也极有可能会把问题掩盖掉，从而导致问题遗留到线上去。

#### 变量命名规范

- **严禁使用中文或者中文拼音**进行重命名

- 使用**驼峰式命名风格**（单词最好控制在三个以内）

- 局部变量或者公开的成员变量应该以作用来命名，例如：

  ```
  public String name;
  public TextView nameView;
  public FrameLayout nameLayout;
  // 命名规范附带技巧（当布局中同个类型的控件只有一个的时候，也可以这样命名）
  public TextView textView;
  public RecyclerView recyclerView;
  ```

  - 非公开的成员变量必须以小 **m** 开头，例如：

  ```
  private String mName;
  private TextView mNameView;
  private FrameLayout mNameLayout;
  // 命名规范附带技巧（当布局中同个类型的控件只有一个的时候，也可以这样命名）
  private TextView mTextView;
  private RecyclerView mRecyclerView;
  ```

  - 布尔值命名规范，无论是局部变量还是成员变量，都不应该携带 **is**，例如：

  ```
  // 不规范写法示例
  private boolean mIsDebug;
  boolean isDebug;
  // 规范写法示例
  private boolean mDebug;
  boolean debug;
  ```

  - 静态变量则用小 **s** 开头，例如：

  ```
  static Handler sHandler;
  ```

  - 常量则需要用大写，并且用下划线代替驼峰，例如：

  ```
  static final String REQUEST_INSTALL_PACKAGES;
  ```

- 有细心的同学可能会发现一个问题，为什么我们最常用的 Glide 和 OkHttp 的源码中，非公开的成员变量为什么没有用小 `m` 开头？但是谷歌的 SDK 源码和 Support 库就有呢？那究竟是用还是不用呢？这个问题其实很好回答，我们可以先从体量上分析，首先谷歌的开发人员和项目数量肯定是最多的，那么谷歌在这块的探索和研究肯定是多于 Glie 和 OkHttp 的，其次是 Glide 和 OkHttp 的源码都有一个特点，很多类都维持在 1k 行代码左右，而谷歌源码很多类都接近 10k 行代码，例如 Activity 的源码在 API 30 上面有 8.8k 行代码，所以谷歌在这块略胜一筹，如果非要二选一，我选择谷歌的代码风格，并不是说 Glide 和 OkHttp 命名风格不好，是因为或许在未来的某一天，可能会有新的图片请求框架和网络请求框架来代替 Glide 和 OkHttp，但是基本不可能会出现有代替 Android SDK 或者 Support 库的一天。

- 最后让我们静静地欣赏一下 **Activity** 类中成员变量的命名：

  ```java
  public class Activity {
  
      public static final int RESULT_CANCELED    = 0;
      public static final int RESULT_OK          = -1;
  
      private Instrumentation mInstrumentation;
  
      private IBinder mToken;
      private IBinder mAssistToken;
  
      private Application mApplication;
  
      /*package*/ Intent mIntent;
  
      /*package*/ String mReferrer;
  
      private ComponentName mComponent;
  
      /*package*/ ActivityInfo mActivityInfo;
  
      /*package*/ ActivityThread mMainThread;
  
      Activity mParent;
  
      boolean mCalled;
  
      /*package*/ boolean mResumed;
  
      /*package*/ boolean mStopped;
  
      boolean mFinished;
      boolean mStartedActivity;
  
      private boolean mDestroyed;
      private boolean mDoReportFullyDrawn = true;
      private boolean mRestoredFromBundle;
  
      private final ArrayList<Application.ActivityLifecycleCallbacks> mActivityLifecycleCallbacks =
              new ArrayList<Application.ActivityLifecycleCallbacks>();
  
      private Window mWindow;
  
      private WindowManager mWindowManager;
  
      private CharSequence mTitle;
      private int mTitleColor = 0;
  
  
      final Handler mHandler = new Handler();
  
      final FragmentController mFragments = FragmentController.createController(new HostCallbacks());
  }
  ```

  

#### 包名命名规范

- 不允许包名中携带**英文大写**
- 包名应该以**简洁的方式**命名
- 包名要按照**模块**或者**作用**来划分
- 请不要在某一包名下放置**一些无关的类**

#### 方法命名规范

- initXX：初始化相关方法，使用 **init** 为前缀标识，如初始化布局 **initView**
- isXX：方法返回值为 boolean 型的请使用 **is** 或 **check** 为前缀标识
- getXX：返回某个值的方法，使用 **get** 为前缀标识，例如 **getName**
- setXX：设置某个属性值，使用 **set** 为前缀标识，例如 **setName**
- handleXX/processXX：对数据进行处理的方法，例如 **handleMessage**
- displayXX/showXX：弹出提示框和提示信息，例如 **showDialog**
- updateXX：更新某个东西，例如 **updateData**
- saveXX：保存某个东西，例如 **saveData**
- resetXX：重置某个东西，例如 **resetData**
- clearXX：清除某个东西，例如 **clearData**
- removeXX：移除数据或者视图等，例如 **removeView**
- drawXX：绘制数据或效果相关的，使用 **draw** 前缀标识，例如 **drawText**

#### 类文件命名规范

- 业务模块：请以 **模块 + 类型** 来命名，例如：

```
HomeActivity.java

SettingFragment.java

HomeAdapter.java

AddressDialog.java
```

- 技术模块：请以类的 **作用** 来命名，例如：

```
CrashHandler.java

GridSpaceDecoration.java

PickerLayoutManager.java
```

#### 接口文件命名规范

- 如果是监听事件可以参考 **View** 的写法及命名：

```
public class View {

    private View.OnClickListener mListener;

    public void setOnClickListener(OnClickListener listener) {
        mListener = listener;
    }

    public interface OnClickListener {

        void onClick(View v);
    }
}
```

- 如果是回调事件可以参考 **Handler** 的写法及命名：

```
public class Handler {

    public interface Callback {

        boolean handleMessage(Message msg);
    }
}
```

- 至于接口写在内部还是外部，具体可以视实际情况而定，如果功能比较庞大，就可以考虑抽取成外部的，只作用在某个类上的，则就可以直接写成内部的。

#### 接口实现规范

- 一般情况下，我们会在类中这样实现接口，这样写的好处是，可以减少对象的创建，并且代码也比较美观。

```
public final class PasswordEditText extends EditText implements View.OnTouchListener, View.OnFocusChangeListener, TextWatcher {

    public PasswordEditText(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        setOnTouchListener(this);
        setOnFocusChangeListener(this);
        addTextChangedListener(this);
    }

    @Override
    public void onFocusChange(View view, boolean hasFocus) {
        ......
    }

    @Override
    public boolean onTouch(View view, MotionEvent event) {
        ......
    }

    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
        ......
    }

    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
        ......
    }

    @Override
    public void afterTextChanged(Editable s) {
        ......
    }
}
```

- 但是有两个美中不足的地方，就是在实现的接口过多时，我们很难分辨是哪个方法是哪个接口的，这个时候可以使用注释的方式来解决这个问题，加上 **@link** 还可以帮助我们快速定位接口类在项目中所在的位置；另外一个是 **implements** 修饰符换行的问题，合理的换行会使代码更加简单直观。

```
public final class PasswordEditText extends EditText
        implements View.OnTouchListener,
        View.OnFocusChangeListener, TextWatcher {

    public PasswordEditText(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        setOnTouchListener(this);
        setOnFocusChangeListener(this);
        addTextChangedListener(this);
    }

    /**
     * {@link OnFocusChangeListener}
     */

    @Override
    public void onFocusChange(View view, boolean hasFocus) {
        ......
    }

    /**
     * {@link OnTouchListener}
     */

    @Override
    public boolean onTouch(View view, MotionEvent event) {
        ......
    }

    /**
     * {@link TextWatcher}
     */

    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
        ......
    }

    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
        ......
    }

    @Override
    public void afterTextChanged(Editable s) {
        ......
    }
}
```

#### 异常捕获规范

- 请不要使用此方式捕获异常，因为这种方式会把问题给隐藏掉，从而会加大后续排查问题的难度。

```
try {
    Xxx.xxx();
} catch (Exception e) {}
```

- 如需捕获异常，请用以下方式进行捕获，列出具体的异常类型，并在代码中输出对应的日志。

```
// 捕获这个异常，避免程序崩溃
try {
    // 目前发现在 Android 7.1 主线程被阻塞之后弹吐司会导致崩溃，可使用 Thread.sleep(5000) 进行复现
    // 查看源码得知 Google 已经在 Android 8.0 已经修复了此问题
    // 主线程阻塞之后 Toast 也会被阻塞，Toast 因为显示超时导致 Window Token 失效
    mHandler.handleMessage(msg);
} catch (WindowManager.BadTokenException | IllegalStateException e) {
    // android.view.WindowManager$BadTokenException：Unable to add window -- token android.os.BinderProxy is not valid; is your activity running?
    // java.lang.IllegalStateException：View android.widget.TextView has already been added to the window manager.
    e.printStackTrace();
    // 又或者上报到 Bugly 错误分析中
    // CrashReport.postCatchedException(e);
}
```

- 如果这个异常不是通过方法 throws 关键字抛出，则需要在 try 块中说明崩溃的缘由，并注明抛出的异常信息。
- 还有一个问题，有异常就一定要 `try catch` ？，这种想法其实是错的，例如我们项目用 Glide 加载图片会抛出以下异常：

```
Caused by: java.lang.IllegalArgumentException: You cannot start a load for a destroyed activity
   at com.bumptech.glide.manager.RequestManagerRetriever.assertNotDestroyed(RequestManagerRetriever.java:348)
   at com.bumptech.glide.manager.RequestManagerRetriever.get(RequestManagerRetriever.java:148)
   at com.bumptech.glide.Glide.with(Glide.java:826)
```