`Fragment`的管理方式对Fragment生命周期的影响

(1) 通过 replace 的方式来管理底部的几个 tab 相互切换

> replace 这种切换方式就是每次切换都要重新创建 Fragment，触发 onResume() 和 onPause()

因此只会触发`Fragment`的`onResume()`、`onPause()`生命周期

    @Override
    public void onResume() {
        super.onResume();
        DATracker.getInstance().onFragmentResume(this);
    }

    @Override
    public void onPause() {
        super.onPause();
        DATracker.getInstance().onFragmentPause(this);
    }

(2) 通过 show / hide 的方式来管理底部的几个 tab 相互切换

> 当底部几个 Fragment 全部创建入栈之后，通过 show 和 hide 来管理 Fragment，此时只有 onHiddenChanged 方法回调，不再触发 onResume() 和 onPause()

因此只会触发`onResume()`、`onPause()`、`onHiddenChanged()`三个方法：

    @Override
    public void onResume() {
        super.onResume();
        DATracker.getInstance().onFragmentResume(this);
    }

    @Override
    public void onPause() {
        super.onPause();
        DATracker.getInstance().onFragmentPause(this);
    }

    @Override
    public void onHiddenChanged(boolean hidden) {
        super.onHiddenChanged(hidden);
        DATracker.getInstance().onFragmentHiddenChanged(this, hidden);
    }
	
(3) Fragment + ViewPager 预加载

> ViewPager 切换 Fragment 时，首先将上一个 Fragment 的 setUserVisibleHint 置为 false，然后将要展示的 setUserVisibleHint 置为 true，也就是说，此时只有 setUserVisibleHint 方法回调，不再触发 onResume() 和 onPause()

因此只会触发`onResume()`、`onPause()`、`setUserVisibleHint()`三个方法：

    @Override
    public void onResume() {
        super.onResume();
        DATracker.getInstance().onFragmentResume(this);
    }

    @Override
    public void onPause() {
        super.onPause();
        DATracker.getInstance().onFragmentPause(this);
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        DATracker.getInstance().setFragmentUserVisibleHint(this, isVisibleToUser);
    }
	