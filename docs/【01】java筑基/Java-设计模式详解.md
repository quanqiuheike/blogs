### `一. 观察者模式

> 定义了对象之间的一对多的依赖，这样一来，当一个对象改变时，它的所有的依赖者都会收到通知并自动更新。比如Jetpack-Lifecycle

- 对于JDK或者Andorid中都有很多地方实现了观察者模式，比如XXXView.addXXXListenter ， 当然了 XXXView.setOnXXXListener不一定是观察者模式，因为观察者模式是一种一对多的关系，对于setXXXListener是1对1的关系，应该叫回调。

#### [Jetpack全家桶之Lifecycle](https://www.jianshu.com/p/a213df52da6a)

通过 getLifecycle().addObserver(new LifeCycleListener());被观察（Activity）将观察者添加到集合，当Activity生命周期变化的时候，



* 专题接口：[Subject.java](https://github.com/youlookwhat/DesignPattern/blob/master/app/src/main/java/com/example/jingbin/designpattern/observer/interfaces/Subject.java) ;

  ```java
  /**
   * 注册一个观察者
   */
  public void registerObserver(Observer observer);
  
  /**
   * 移除一个观察者
   */
  public void removeObserver(Observer observer);
  
  /**
   * 通知所有观察者
   */
  public void notifyObservers();
  ```

* 3D服务号的实现类：[ObjectFor3D.java](https://github.com/youlookwhat/DesignPattern/blob/master/app/src/main/java/com/example/jingbin/designpattern/observer/classs/ObjectFor3D.java)

  ```java
  @Override
  public void registerObserver(Observer observer) {
      observers.add(observer);
  }
  @Override
  public void removeObserver(Observer observer) {
      int index = observers.indexOf(observer);
      if (index >= 0) {
          observers.remove(index);
      }
  }
  @Override
  public void notifyObservers() {
      for (Observer observer : observers) {
          observer.update(msg);
      }
  }
  /**
   * 主题更新信息
   */
  public void setMsg(String msg) {
      this.msg = msg;
      notifyObservers();
  }
  ```

* 所有观察者需要实现此接口:[Observer.java](https://github.com/youlookwhat/DesignPattern/blob/master/app/src/main/java/com/example/jingbin/designpattern/observer/interfaces/Observer.java)

  ```java
  public ObserverUser1(Subject subject) {
      subject.registerObserver(this);
  }
  @Override
  public void update(String msg) {
      Log.e("-----ObserverUser1 ", "得到 3D 号码:" + msg + ", 我要记下来。");
  }
  ```

  