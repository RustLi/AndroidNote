# M-V-VM

### 概述

全称为Model，View，ViewModel。

> Model：实体模型，数据
>
> View：UI相关
>
> ViewModel：View和Model间的交互，业务逻辑和数据处理

核心采用数据绑定，数据驱动UI。

![img](F:\LearningDocs\Mvvm\flowPic\architecture.png)

### 实例

* Model

  ~~~java
  package com.example.lwl.mvvm;
  
  public class MvvmModel {
      //获取Id
      private String mId;
      //回调接口
      private OnCallbackListener mOnCallbackListener;
  
      public MvvmModel(OnCallbackListener onCallbackListener) {
          this.mOnCallbackListener = onCallbackListener;
      }
  
      public String getmId() {
          return mId;
      }
  
      public void setmId(String mId) {
          this.mId = mId;
      }
  
      public void loadModel(String mId){
          setmId(mId);
          if (mOnCallbackListener != null) {
              mOnCallbackListener.onSuccess(this);
          }
      }
  
      interface  OnCallbackListener{
          void onSuccess(MvvmModel mvvmModel);
      }
  }
  ~~~

* View

  代码部分：

  ~~~java
  public class MvvmActivity extends AppCompatActivity {
      @Override
      protected void onCreate(@Nullable Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          ActivityMvvmBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_mvvm);
          final  MvvmViewModel viewModel = new MvvmViewModel();
          binding.setViewModel(viewModel);
      }
  }
  ~~~

  XML部分：

  * 总体布局采用layout包裹
  * 定义<data>变量作为ViewModel
  * xml中引用ViewModel中的字段和方法，注意方法中需要传递view对象

  ~~~java
  <?xml version="1.0" encoding="utf-8"?>
  <layout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">
      <data>
          <variable
              name="viewModel"
              type="com.example.lwl.mvvm.MvvmViewModel" />
      </data>
  
      <android.support.constraint.ConstraintLayout
          android:layout_width="match_parent"
          android:layout_height="match_parent">
  
          <TextView
              android:id="@+id/tv_test"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_marginTop="176dp"
              android:text="@{viewModel.mId}"
              app:layout_constraintEnd_toEndOf="parent"
              app:layout_constraintHorizontal_bias="0.501"
              app:layout_constraintStart_toStartOf="parent"
              app:layout_constraintTop_toTopOf="parent" />
  
          <Button
              android:id="@+id/btn_test"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_marginTop="64dp"
              android:onClick="@{viewModel.getData}"
              android:text="@string/get_data"
              app:layout_constraintEnd_toEndOf="parent"
              app:layout_constraintStart_toStartOf="parent"
              app:layout_constraintTop_toBottomOf="@+id/tv_test" />
  
      </android.support.constraint.ConstraintLayout>
  </layout>
  ~~~

  

* ViewModel

~~~java
public class MvvmViewModel extends BaseObservable {
    public ObservableField<String> mId = new ObservableField<>();

    public void getData(View view){
        loadData();
    }

    private void loadData(){
        MvvmModel mvvmModel = new MvvmModel(model -> mId.set(model.getmId()));
        String mTestStr = "测试测试";
        mvvmModel.loadModel(mTestStr);
    }
}
~~~

