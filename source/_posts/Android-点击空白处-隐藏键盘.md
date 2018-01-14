---
title: Android 点击空白处 隐藏键盘
copyright: true
date: 2018-01-02 22:45:51
categories: Android
tags: Android
---
这里给大家介绍3种方案，均可以封装在BaseActivity使用，可以根据自己实际需求酌情使用：

1. 判断焦点（网上介绍最多的方案）
原理：在事件分发的时候判断当前获取焦点的View是不是EditText，是EditText就判断MotionEvent 是否发生在这个View上，然后隐藏键盘，不足的是点击另一个EditText会看到键盘隐藏然后再显示。
```
    /**
     * 获取点击事件
     */
    @CallSuper
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.MotionEvent ) {
            View view = getCurrentFocus();
            if (isShouldHideKeyBord(view, ev)) {
                hideSoftInput(view.getWindowToken());
            }
        }
        return super.dispatchTouchEvent(ev);
    }

    /**
     * 判定当前是否需要隐藏
     */
    protected boolean isShouldHideKeyBord(View v, MotionEvent ev) {
        if (v != null && (v instanceof EditText)) {
            int[] l = {0, 0};
            v.getLocationInWindow(l);
            int left = l[0], top = l[1], bottom = top + v.getHeight(), right = left + v.getWidth();
            return !(ev.getX() > left && ev.getX() < right && ev.getY() > top && ev.getY() < bottom);
        }
        return false;
    }

    /**
     * 隐藏软键盘
     */
    private void hideSoftInput(IBinder token) {
        if (token != null) {
            InputMethodManager manager = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
            manager.hideSoftInputFromWindow(token, InputMethodManager.HIDE_NOT_ALWAYS);
        }
    }
```

2. 根布局设置点击事件
原理：对根布局设置点击事件，点击根布局则隐藏键盘。不足的是点击到能获取焦点的控件上无效。
```
    @CallSuper
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //点击空白处隐藏键盘
        ((ViewGroup) findViewById(android.R.id.content)).getChildAt(0).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                hideSoftInput(view.getWindowToken());
            }
        });
    }

    /**
     * 隐藏软键盘
     */
    private void hideSoftInput(IBinder token) {
        if (token != null) {
            InputMethodManager manager = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
            manager.hideSoftInputFromWindow(token, InputMethodManager.HIDE_NOT_ALWAYS);
        }
```

3. 过滤不需要隐藏键盘的View
原理：在事件分发的时候判断ACTION_DOWN事件是不是在需要过滤的View上，不是就隐藏键盘，不足的是每发生一次ACTION_DOWN都要循环一次filterViews。
```
    /**
     * 获取点击事件
     */
    @CallSuper
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN  && filterViews != null) {
            View focusView = getCurrentFocus();
            if (isShouldHideKeyBord(ev)) {
                hideSoftInput(focusView.getWindowToken());
            }
        }
        return super.dispatchTouchEvent(ev);
    }

    /**
     *设置需要过滤掉的View
     */
    protected void setFilterView(View... view) {
        this.filterViews = view;
    }

    /**
     * 判定当前是否需要隐藏
     */
    protected boolean isShouldHideKeyBord(MotionEvent ev) {
        boolean hide = true;
        for (View v : filterViews) {
            int[] l = {0, 0};
            v.getLocationInWindow(l);
            int left = l[0], top = l[1], bottom = top + v.getHeight(), right = left + v.getWidth();
            if (ev.getX() > left && ev.getX() < right && ev.getY() > top && ev.getY() < bottom) {
                hide = false;
                break;
            }
        }
        return hide;
    }

    /**
     * 隐藏软键盘
     */
    private void hideSoftInput(IBinder token) {
        if (token != null) {
            InputMethodManager manager = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
            manager.hideSoftInputFromWindow(token, InputMethodManager.HIDE_NOT_ALWAYS);
        }
    }
```