---
title: Android Fragment监听返回键
copyright: true
date: 2018-01-02 22:31:29
categories: Android
tags: [Android, Fragment]
---
场景：在项目中做联系人界面时，需要按名字和按部门显示联系人，此处使用2个fragment切换显示，按部门显示需要体现部门层级关系，需要实现点击返回上级部门。因为Fragment并不能像在Actvity重写onBackPressed即可，此时就需要在Fragment监听处理返回，否则返回事件在Activity中，并不能返回上级部门。

1. Fragment中没有可以主动获取焦点的控件（**<small>如：edittext</small>**）
```
    //主界面获取焦点
    @SuppressWarnings("ConstantConditions")
    private void getFocus() {
        getView().setFocusableInTouchMode(true);
        getView().requestFocus();
        getView().setOnKeyListener(new View.OnKeyListener() {
            @Override
            public boolean onKey(View v, int keyCode, KeyEvent event) {
                if (event.getAction() == KeyEvent.ACTION_UP && keyCode == KeyEvent.KEYCODE_BACK) {
                    //TODO: handle back button
                    return true;
                }
                return false;
            }
        });
    }
```

2. Fragment中有可以主动获取焦点的控件,需要对它进行处理
```
        editText.setOnKeyListener(new View.OnKeyListener() {
            @Override
            public boolean onKey(View v, int keyCode, KeyEvent event) {
                if (keyCode == KeyEvent.KEYCODE_BACK && event.getAction() == KeyEvent.ACTION_UP) {
                    //关闭软键盘
                    InputMethodManager imm = (InputMethodManager) getActivity().getSystemService(Context.INPUT_METHOD_SERVICE);
                    imm.hideSoftInputFromWindow(editText.getWindowToken(), 0);
                    //TODO: handle back button
                    return true;
                }
                return false;
            }
        });
```

参考来源：https://stackoverflow.com/questions/22552958/handling-back-press-when-using-fragments-in-android

以上方法测试有效，另附上其他方法（**<small>未测试</small>**）
1. [两步搞定Fragment的返回键](http://www.jianshu.com/p/fff1ef649fc0)