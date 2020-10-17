## JetPack 总结

### ViewModel

#### 1、ViewModel 生命周期问题

首先，ViewModel 的 Scope 只能是它所对应的 LifeCycleOwner 的 Scope，不会超出这个 Scope，所以 Activity 被除屏幕旋转所导致的强制销毁以后，ViewModel 同样无法存活。

Activity 正常销毁，比如按返回按钮，也是会清空 ViewModel 的。

ViewModel 只有在 Configuration 改变导致 Activity 销毁重建时，才会存活得比 Activity 长。

#### 2、ViewModel 是如何 Config 改变时做到保存自己的？

参考：https://www.cnblogs.com/lotz/p/13680574.html

* ViewModelStore

  Acitivity 等实现了 ViewModelStoreOwner 接口的类，内部包含一个 ViewModelStore 成员变量。

  ViewModelStore 代码非常简单，如下：

  ```java
  public class ViewModelStore {
  
      private final HashMap<String, ViewModel> mMap = new HashMap<>();
  
      final void put(String key, ViewModel viewModel) {
          ViewModel oldViewModel = mMap.put(key, viewModel);
          if (oldViewModel != null) {
              oldViewModel.onCleared();
          }
      }
  
      final ViewModel get(String key) {
          return mMap.get(key);
      }
  
      Set<String> keys() {
          return new HashSet<>(mMap.keySet());
      }
      public final void clear() {
          for (ViewModel vm : mMap.values()) {
              vm.clear();
          }
          mMap.clear();
      }
  }
  ```

  如上述代码所示，ViewModelStore 保存了一个存储 ViewModel 的 Map， key 为 ViewModel 的类名，value 是 ViewModel 对象。在创建 ViewModelProvider 对象的时候，会把这个 store 对象传给 ViewModelProvider ，这样就可以从 ViewModelProvider 对象中拿到 ViewModel 了。

  在 Activity 因为 Config 改变而销毁时，会调用 onRetainNonConfigurationInstance() 来保存 ViewModelStore， 如下：

  ```java
   public final Object onRetainNonConfigurationInstance() {
          Object custom = onRetainCustomNonConfigurationInstance();
  
          ViewModelStore viewModelStore = mViewModelStore;
          if (viewModelStore == null) {
              // No one called getViewModelStore(), so see if there was an existing
              // ViewModelStore from our last NonConfigurationInstance
              NonConfigurationInstances nc =
                      (NonConfigurationInstances) getLastNonConfigurationInstance();
              if (nc != null) {
                  viewModelStore = nc.viewModelStore;
              }
          }
  
          if (viewModelStore == null && custom == null) {
              return null;
          }
  
          NonConfigurationInstances nci = new NonConfigurationInstances();
          nci.custom = custom;
          nci.viewModelStore = viewModelStore; // 将 viweModelStore 保存在 nci 中
          return nci;
      }
  ```

  上述方法调用发生在 onStop 和 onDestroy 之间。

  在 Activity 重建的时候，ViewModelProvider 通过 provide 函数从 Activity 中拿到 ViewModelStore，而从 Activity 中拿 ViewModelStore 的方法如下：

  ```java
  public ViewModelStore getViewModelStore() {
          if (getApplication() == null) {
              throw new IllegalStateException("Your activity is not yet attached to the "
                      + "Application instance. You can't request ViewModel before onCreate call.");
          }
          if (mViewModelStore == null) {
              NonConfigurationInstances nc =
                      (NonConfigurationInstances) getLastNonConfigurationInstance();
              if (nc != null) {
                  // Restore the ViewModelStore from NonConfigurationInstances
                  mViewModelStore = nc.viewModelStore;
              }
              if (mViewModelStore == null) {
                  mViewModelStore = new ViewModelStore();
              }
          }
          return mViewModelStore;
      }
  ```

  可以看到，在这里，从 nci 中拿到了之前保存的 ViewModelStore。

