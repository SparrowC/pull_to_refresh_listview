# pull_to_refresh_listview
anroid中的下拉刷新列表

![功能演示](https://raw.githubusercontent.com/SparrowC/NewsAPP/master/news.gif)

##用法：##
```java
PullRefreshListView list = (PullRefreshListView) findViewById(R.id.lv_newList);
list .setOnRefreshListener(new PullRefreshListView.OnRefreshListener() {
            @Override
            public void OnPullRefresh() {
                //向下拉刷新列表时
            }

            @Override
            public void OnReleaseRefresh() {
                //向下拉后，释放手时
            }

            @Override
            public void OnRefreshing() {
                //刷新时
            }
        });
```

##源码##
```java

import android.content.Context;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;
import android.view.animation.Animation;
import android.view.animation.RotateAnimation;
import android.widget.ImageView;
import android.widget.ListView;
import android.widget.ProgressBar;
import android.widget.TextView;


/**
 * Created by Vonnie on 2016/3/1.
 */
public class PullRefreshListView extends ListView {
    private Context mContext;
    private View header;
    private TextView tv_refreshTitle,tv_refreshTime;
    private ImageView iv_refreshArrow;
    private ProgressBar pb_refresh;

    private int headerHeight;
    private int startY;
    private OnRefreshListener mRefreshListener;

    final private int NORMAL=1;
    final private int PULL_REFRESH=1;
    final private int RELEASE_REFRESH=2;
    final private int REFRESHING=3;

    private int curState;
    public PullRefreshListView(Context context) {
        super(context);
        initHeaderView(context);
    }

    public PullRefreshListView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initHeaderView(context);
    }

    public PullRefreshListView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initHeaderView(context);
    }

    /**
     * 初始化头View
     * @param context
     */
    private void initHeaderView(Context context) {
        mContext=context;
        header= LayoutInflater.from(mContext).inflate(R.layout.header_view_layout,null);
        tv_refreshTitle= (TextView) header.findViewById(R.id.tv_refreshTitle);
        tv_refreshTime= (TextView) header.findViewById(R.id.tv_refreshTime);
        iv_refreshArrow= (ImageView) header.findViewById(R.id.iv_refreshArrow);
        pb_refresh= (ProgressBar) header.findViewById(R.id.pb_refresh);

        addHeaderView(header);

        header.measure(0, 0);
        headerHeight=header.getMeasuredHeight();
        header.setPadding(0, -headerHeight, 0, 0);
        curState=NORMAL;
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {

        switch (ev.getAction())
        {
            case MotionEvent.ACTION_DOWN:
                startY= (int) ev.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                if(startY>0)
                {
                    int endY= (int) ev.getY();
                    int scrollY=endY-startY;
                    if(this.getFirstVisiblePosition()==0&&scrollY>0)
                    {
                        int moveY=scrollY-headerHeight;
                        if(moveY<=0)
                        {//向下滑动，显示下拉刷新
                            curState=PULL_REFRESH;

                        }else {
                            //向下滑动，显示释放刷新
                            curState=RELEASE_REFRESH;
                        }
                        onStateChange(curState);
                        header.setPadding(0,moveY,0,0);
                        return true;
                    }
                }
                break;
            case MotionEvent.ACTION_UP:
                if(curState!=NORMAL)
                {
                    if(curState==RELEASE_REFRESH)
                    {//释放刷新，开始刷新
                        curState=REFRESHING;
                        onStateChange(curState);
                    }else {
                        header.setPadding(0,-headerHeight,0,0);
                        startY=-1;
                    }
                    return true;
                }
                break;
        }

       // return true;
        return super.onTouchEvent(ev);
    }

    private void onStateChange(int state) {
        switch (state){
            case PULL_REFRESH:
                pullRefresh();
                break;
            case RELEASE_REFRESH:
                releaseRefresh();
                break;
            case REFRESHING:
                refreshing();
                break;
        }
    }

    //正在刷新
    private void refreshing() {
        iv_refreshArrow.clearAnimation();
        header.setPadding(0, 0, 0, 0);
        pb_refresh.setVisibility(View.VISIBLE);
        iv_refreshArrow.setVisibility(View.INVISIBLE);
        tv_refreshTime.setText("最后刷新："+DateUtils.getStringDate());
        tv_refreshTitle.setText("正在刷新");
        if(mRefreshListener!=null)
            mRefreshListener.OnRefreshing();
    }

    //释放刷新
    private void releaseRefresh() {
        pb_refresh.setVisibility(View.INVISIBLE);
        iv_refreshArrow.setVisibility(View.VISIBLE);
        tv_refreshTitle.setText("释放刷新");
        RotateAnimation ra=new RotateAnimation(0,180, Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
        ra.setDuration(200);
        ra.setFillAfter(true);
        iv_refreshArrow.startAnimation(ra);
        if(mRefreshListener!=null)
            mRefreshListener.OnReleaseRefresh();
    }

    //下拉刷新
    private void pullRefresh() {
        iv_refreshArrow.clearAnimation();
        pb_refresh.setVisibility(View.INVISIBLE);
        iv_refreshArrow.setVisibility(View.VISIBLE);
        tv_refreshTitle.setText("下拉刷新");
        if(mRefreshListener!=null)
            mRefreshListener.OnPullRefresh();
    }

    public void setListNormal()
    {
        iv_refreshArrow.clearAnimation();
        pb_refresh.setVisibility(View.INVISIBLE);
        iv_refreshArrow.setVisibility(View.VISIBLE);
        tv_refreshTitle.setText("下拉刷新");
        header.setPadding(0,-headerHeight,0,0);
        curState=NORMAL;
    }

    public void setOnRefreshListener(OnRefreshListener refreshListener)
    {
        mRefreshListener=refreshListener;
    }
    public interface OnRefreshListener{
        void OnPullRefresh();
        void OnReleaseRefresh();
        void OnRefreshing();
    }

}

```
header_view_layout的布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="100dp"
    android:orientation="horizontal">
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="5dp">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">
            <FrameLayout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content">
                <ImageView
                    android:id="@+id/iv_refreshArrow"
                    android:layout_width="80dp"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:visibility="invisible"
                    android:src="@drawable/common_listview_headview_red_arrow"/>
                <ProgressBar
                    android:id="@+id/pb_refresh"
                    android:layout_width="80dp"
                    android:layout_height="wrap_content"
                    android:visibility="gone"/>
            </FrameLayout>
            <RelativeLayout
                android:layout_width="0dp"
                android:layout_height="match_parent"
                android:layout_weight="1">
                <TextView
                    android:id="@+id/tv_refreshTitle"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="下拉刷新"
                    android:textSize="25dp"
                    android:layout_centerHorizontal="true"
                    android:paddingTop="5dp"
                    android:textColor="#f00"/>
                <TextView
                    android:id="@+id/tv_refreshTime"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_below="@id/tv_refreshTitle"
                    android:text="最后刷新 2016年3月1日 11:31:32"
                    android:textSize="16dp"
                    android:layout_centerHorizontal="true"
                    android:paddingTop="5dp"/>
            </RelativeLayout>
        </LinearLayout>
    </FrameLayout>
</LinearLayout>
```
图片资源：箭头common_listview_headview_red_arrow
