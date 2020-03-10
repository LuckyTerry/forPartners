# 第4章 ListView使用技巧

---

ListView基础优化
==

使用ViewHolder,复用convertView

    @Override
    public View getView(int position, View convertView, ViewGroup parent){
        ViewHolder holder = null;
        if(convertView == null){
            holder = new ViewHolder();
            convertView = LayoutInflater.from(mContext).inflate(R.layout.viewholder_item, null);
            holder.img = (ImageView) convertView.findViewById(R.id.imageView);
            holder.title = (TextView) convertView.findViewById(R.id.textView);
            convertView.setTag(holder);
    
        }else{
            holder = (ViewHolder) convertView.getTag();
        }
        holder.img.setBackgroundResource(R.drawable.ic_launcher);
        holder.title.setText(mData.get(position));
        return convertView;
    }
    
    public final class ViewHolder{
        public ImageView img;
        publlic TextView title;
    }
    
---

ListView常用配置
==

分割线divider

    android:divider="@android:color/dark_gray"
    android:dividerHeight=""dp
    android:divider="@null"//透明
    
滚动条scrollBar

    android:scrollBars="none"
    
item点击效果

    android:listSelector="#00000000"//取消点击效果
    android:listSelector="@android:color/transparent"//取消点击效果
    
设置ListView显示在第几页

    listView.setSelection(N);
    
动态修改ListView

    mData.add("new");
    mAdapter.notifyDataSetChanged();
    
遍历ListView中的所有Item

    for(int i = 0; i < mListView.getChildCount(); i++){
        View view = mListView.getChildAt(i);
    }
    
处理空ListView

    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        
        <ListView
            android:id="@+id/listview"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
        
        <TextView
            android:id="@+id/empty_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:gravity="center"
            android:text="Empty" />
    </FrameLayout>
    
    listView.setEmptyView(findViewById(R.id.empty_view));
    
ListView滑动监听

    OnTouchListener
    
    mListView.setOnTouchListener(new View.OnTouchListener(){
        @Override
        public boolean onTouch(View v, MotionEvent event){
            switch(event.getAction()){
                case MotionEvent.ACTION_DOWN:
                    break;
                case MotionEvent.ACTION_MOVE:
                    break;
                case MotionEvent.ACTION_UP:
                    break;
            }
            return false;
        }
    });
    
    OnScrollListener
    
    mListView.setOnScrollListener(new OnScrollListener(){
        @Override
        public void onScrollStateChanged(AbsListView view, int scrollState){
            switch(scrollState){
                case OnScrollListener.SCROLL_STATE_IDLE:
                    break;
                case OnScrollListener.SCROLL_STATE_TOUCH_SCROLL:
                    break;
                case OnScrollListener.SCROLL_STATE_FLYING:
                    break;
            }
        }
        @Override
        public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount){
            if(firstVisibleItem + visibleItemCount == totalItemCount && totalItemCount > 0){
                //滚动到最后一行
            }
            
            if(firstVisibleItem > mLastVisiblePosition){
                //上滑
            }else if(firstVisibleItem < mLastVisiblePosition){
                //下滑
            }
            mLastVisiblePosition = firstVisibleItem;
        }
    });
    
---

ListView常用扩展
==

## 1. 具有弹性的ListView

核心代码如下

    privite int mMaxOverDistance = 50;

    private void initView(){
        DisplayMetrics metrics = mContext.getResources().getDisplayMetrics();
        float density = metrics.density;
        mMaxOverDistance = (int) (density * mMaxOverDistance);
    
        /*
        WindowManager windowManager = (WindowManager) mContext.getSystemService(Context.WINDOW_SERVICE);
        DisplayMetrics metrics = new DisplayMetrics();// 创建了一张白纸
        windowManager.getDefaultDisplay().getMetrics(metrics);// 给白纸设置宽高
        float density = metrics.density;
        mMaxOverDistance = (int) (density * mMaxOverDistance);
        */
    }


    ＠Override
    protected boolean overScrollBy（int deltaX， int deltaY， 
                                    int scrollX， int scrollY， 
                                    int scrollRangeX， int scrollRangeY， 
                                    int maxOverScrollX， int maxOverScrollY， 
                                    boolean isTouchEvent）{ 
        return super.overScrollBy（deltaX， deltaY， 
                                    scrollX， scrollY， 
                                    scrollRangeX， scrollRangeY， 
                                    maxOverScrollX， mMaxOverDistance， 
                                    isTouchEvent）;  
    }
    
## 2. 自动显示、隐藏布局的ListView

核心代码如下

    private static final int UP_HIDE = 0;
    private static final int DOWN_SHOW = 1;
    private float mFirstY;
    private float mTouchSlop;
    private int mDirection;
    private boolean mShow = true;

    mTouchSlop = ViewConfiguration.get(this).getScaledTouchSlop();

    mListView.setOnTouchListener(new View.OnTouchListener() {
        @Override
        public boolean onTouch(View view, MotionEvent motionEvent) {
            switch (motionEvent.getAction()){
                case MotionEvent.ACTION_DOWN:
                    mFirstY = motionEvent.getY();
                    break;
                case MotionEvent.ACTION_MOVE:
                    float currentY = motionEvent.getY();
                    if(currentY - mFirstY > mTouchSlop){
                        mDirection = DOWN_SHOW;//下滑_显示
                    }else if(mFirstY - currentY > mTouchSlop){
                        mDirection = UP_HIDE;//上滑_隐藏
                    }
                    if(mDirection == UP_HIDE){
                        if(mShow){
                            toolbarAnim(UP_HIDE);//上滑_隐藏
                            mShow = false;
                        }
                    }else if(mDirection == DOWN_SHOW){
                        if(!mShow){
                            toolbarAnim(DOWN_SHOW);//下滑_显示
                            mShow = true;
                        }
                    }
                    break;
                case MotionEvent.ACTION_UP:
                    break;
            }
            return false;
        }
    });

    View header = new View(this);
    header.setLayoutParams(new AbsListView.LayoutParams(AbsListView.LayoutParams.MATCH_PARENT,
            (int)getResources().getDimension(R.dimen.abc_action_bar_default_height_material)));
    mListView.addHeaderView(header);
        
        
    private void toolbarAnim(int flag){
        if(mAnimator != null && mAnimator.isRunning()){
            mAnimator.cancel();
        }
        if(flag == UP_HIDE){//上滑_隐藏
            mAnimator = ObjectAnimator.ofFloat(mToolbar,"translationY",mToolbar.getTranslationY(),-mToolbar.getHeight());
        }else if (flag == DOWN_SHOW){//下滑_显示
            mAnimator = ObjectAnimator.ofFloat(mToolbar,"translationY",mToolbar.getTranslationY(),0);
        }
        mAnimator.start();
    }

## 3. 聊天ListView

核心代码如下

    @Override
    public int getViewTypeCount() {
        return 2;
    }
    @Override
    public int getItemViewType(int position) {
        return mDataList.get(position).getType();
    }


    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
        ViewHolder viewHolder = null;
        if(view == null){
            if (getItemViewType(i) == ChatItemBean.TYPE_IN){
                viewHolder = new ViewHolder();
                view = mLayoutInflater.inflate(R.layout.item_chat_in,viewGroup,false);
                viewHolder.mTextView = (TextView) view.findViewById(R.id.tv_msg_in);
                viewHolder.mImageView = (ImageView) view.findViewById(R.id.iv_msg_in);
                view.setTag(viewHolder);
            }else if (getItemViewType(i) == ChatItemBean.TYPE_OUT){
                viewHolder = new ViewHolder();
                view = mLayoutInflater.inflate(R.layout.item_chat_out,viewGroup,false);
                viewHolder.mTextView = (TextView) view.findViewById(R.id.tv_msg_out);
                viewHolder.mImageView = (ImageView) view.findViewById(R.id.iv_msg_out);
                view.setTag(viewHolder);
            }
        }else{
            viewHolder = (ViewHolder) view.getTag();
        }
        viewHolder.mImageView.setImageBitmap(mDataList.get(i).getIcon());
        viewHolder.mTextView.setText(mDataList.get(i).getMsg());
        return view;
    }

    public final class ViewHolder{
        public ImageView mImageView;
        public TextView mTextView;
    }
    
    private void initData(){
        mDataList = new ArrayList<>();
        ChatItemBean bean = null;

        bean = new ChatItemBean();
        bean.setIcon(BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher));
        bean.setMsg("Hello");
        bean.setType(ChatItemBean.TYPE_IN);
        mDataList.add(bean);

        bean = new ChatItemBean();
        bean.setIcon(BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher));
        bean.setMsg("Hello");
        bean.setType(ChatItemBean.TYPE_OUT);
        mDataList.add(bean);
    }
## 4. 动态改变ListView布局

略