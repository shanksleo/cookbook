# view依赖注入
俗话说：“不会偷懒的程序员不是好的程序员！”。作为一名Android开发，是不是经常厌烦了大量的findViewById以及setOnClickListener代码，而*ButterKnife是一个专注于Android系统的View注入框架*，让你从此从这些烦人臃肿的代码中解脱出来。先来看一段代码示例说明下ButterKnife是如何简化代码的：
#####注意：如果你是使用的Eclipse引用该library，你需要参考这里Eclipse Configuration做一些配置，否则会运行出错。
```java
class ExampleActivity extends Activity {
  TextView title;
  TextView subtitle;
  TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    title = (TextView) findViewById(R.id.title);
    subtitle = (TextView) findViewById(R.id.subtitle);
    footer = (TextView) findViewById(R.id.footer);

    // TODO Use views...
  }
}
```
而用ButterKnife之后的代码是这样的：
```java
class ExampleActivity extends Activity {
  @InjectView(R.id.title) TextView title;
  @InjectView(R.id.subtitle) TextView subtitle;
  @InjectView(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    //ButterKnife.inject(this);可以用来标记对应的layout
    ButterKnife.inject(this);
    // TODO Use "injected" views...
  }
}
```
是不是非常简洁易用？下面就来系统的介绍下ButterKnife的用法。
Butter Knife 的特性

* 支持 Activity 中的 View 注入

* 支持 View 中的 View 注入

* 支持 View 事件回调函数注入

目前支持如下事件回调函数：

```java
View: @OnLongClick and @OnFocusChanged.
TextView: @OnEditorAction.
AdapterView: @OnItemClick and @OnItemLongClick.
CompoundButton: @OnCheckedChanged.
```
下面来看一些注入的示例代码：
* Activity 中注入
```java
class ExampleActivity extends Activity {
  @InjectView(R.id.title) TextView title;
  @InjectView(R.id.subtitle) TextView subtitle;
  @InjectView(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.inject(this);
    // TODO Use "injected" views...
  }
}
```



* 在 Fragment 中注入
```java
public class FancyFragment extends Fragment {
  @InjectView(R.id.button1) Button button1;
  @InjectView(R.id.button2) Button button2;

  @Override
  View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.inject(this, view);
    // TODO Use "injected" views...
    return view;
  }
}
```



* 在 ViewHolder 模式中注入

```java
public class MyAdapter extends BaseAdapter {
  @Override public View getView(int position, View view, ViewGroup parent) {
    ViewHolder holder;
    if (view != null) {
      holder = (ViewHolder) view.getTag();
    } else {
      view = inflater.inflate(R.layout.whatever, parent, false);
      holder = new ViewHolder(view);
      view.setTag(holder);
    }

    holder.name.setText("John Doe");
    // etc...

    return convertView;
  }

  static class ViewHolder {
    @InjectView(R.id.title) TextView name;
    @InjectView(R.id.job_title) TextView jobTitle;

    public ViewHolder(View view) {
      ButterKnife.inject(this, view);
    }
  }
}
```


* 注入回调函数


下面是几种注入回调函数的方法示例:
```java
// 带有 Button 参数
@OnClick(R.id.submit)
public void sayHi(Button button) {
  button.setText("Hello!");
}

// 不带参数
@OnClick(R.id.submit)
public void submit() {
  // TODO submit data to server...
}

// 同时注入多个 View 事件
@OnClick({ R.id.door1, R.id.door2, R.id.door3 })
public void pickDoor(DoorView door) {
  if (door.hasPrizeBehind()) {
    Toast.makeText(this, "You win!", LENGTH_SHORT).show();
  } else {
    Toast.makeText(this, "Try again", LENGTH_SHORT).show();
  }
}
```


* Reset函数


如果需要在 界面 销毁的时候，把注入的 View 设置为 Null， 则可以用 reset 函数：
```java
public class FancyFragment extends Fragment {
  @InjectView(R.id.button1) Button button1;
  @InjectView(R.id.button2) Button button2;

  @Override View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.inject(this, view);
    // TODO Use "injected" views...
    return view;
  }

  @Override void onDestroyView() {
    super.onDestroyView();
    Views.reset(this);
  }
}
```

另外 还支持 可选的 View 注入，如果该 View 没有，就没有吧：
```java
@Optional @InjectView(R.id.might_not_be_there) TextView mightNotBeThere;

@Optional @OnClick(R.id.maybe_missing) void onMaybeMissingClicked() {
  // TODO ...
}
```

还有两个 findViewById 函数来简化查找 View 的方式，如果上面都满足不了你的需求，你可以用用他们：
```java
View view = LayoutInflater.from(context).inflate(R.layout.thing, null);
TextView firstName = Views.findById(view, R.id.first_name);
TextView lastName = Views.findById(view, R.id.last_name);
ImageView photo = Views.findById(view, R.id.photo);
```

最后，如果你是用Android Studio来作为IDE的话，那么有一个ButterKnife的插件android-butterknife-zelezny, 该插件可以让你手动生成上述注入代码，从此让自己成为一个更懒惰的程序员，上张截图吧。
![](http://shanks.qiniudn.com/shanks_zelezny_animated.gif)
