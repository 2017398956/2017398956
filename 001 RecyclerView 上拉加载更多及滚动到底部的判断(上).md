# RecyclerView 上拉加载更多及滚动到底部的判断(上) #
关于下拉刷新上拉加载更多，网上有很多例子；下拉刷新比较简单直接使用系统提供 SwipeRefreshLayout 即可，比较麻烦的是上拉加载更多，实现上拉的方法多种多样，这里对各个方法总结一下。

## 需求分析 ##
RecyclerView 滚动到底部后，用户再往上拖拽（这里使用场景是拖拽，而不是手指离屏后的自动滚动到底部）时，RecyclerView 展示出 加载更多 的字样并请求更多的数据，请求成功后更新 RecyclerView ；在默认情况下 RecyclerView 的高度大于可展示高度，即 RecyclerView 没有展示出全部 item ，可滑动，反之则说明不需要加载更多。
## 技术点 ##
- 判断 RecyclerView 滚动到了底部
- 判断 RecyclerView 拖拽

## 判断 RecyclerView 滚动到了底部 ##
判断 RecyclerView 是否到达底部网上流传着下面几种方法，这些方法都存在些许问题，最后我们再给出推荐的方式。

**1. 根据 item 判断是否到达底部**
这种方法最常见，一般都是像下面这样实现：

	public static boolean isVisBottom(RecyclerView recyclerView){  
	  LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();  
	  //屏幕中最后一个可见子项的position
	  int lastVisibleItemPosition = layoutManager.findLastVisibleItemPosition();  
	  //当前屏幕所看到的子项个数
	  int visibleItemCount = layoutManager.getChildCount();  
	  //当前RecyclerView的所有子项个数
	  int totalItemCount = layoutManager.getItemCount();  
	  //RecyclerView的滑动状态
	  int state = recyclerView.getScrollState();  
	  if(visibleItemCount > 0 && lastVisibleItemPosition == totalItemCount - 1 && state == recyclerView.SCROLL_STATE_IDLE){   
	     return true; 
	  }else {   
	     return false;  
	  }
	}

用这种方式判断是否滚动到底部时，只要最后一个 item 显示出一点，就会触发加载更多，用户此时看不到在 FooterView 处的 加载更多 字样（与拖拽展示出加载更多的需求不符）；另外，当 RecyclerView 的 item 过少不足填满整个 RecyclerView 时，也会触发 加载更多 ；因此，这种方式不符合我们的要求。

**2. 使用 canScrollVertically(int direction) 判断是否到达底部**

	RecyclerView.canScrollVertically(1)的值表示是否能向上滚动，false表示已经滚动到底部
	RecyclerView.canScrollVertically(-1)的值表示是否能向下滚动，false表示已经滚动到顶部

这种方法看似简单，其实同样存在一些陷阱。当 RecyclerView 的 item 过少不足填满整个 RecyclerView 时，无论上拉还是下拉都会触发加载更多；另外，direction 不只可取1和-1，只需保证正负就能达到一样的效果。

	// View#canScrollVertically(int direction) 源码
    public boolean canScrollVertically(int direction) {
        final int offset = computeVerticalScrollOffset();
        final int range = computeVerticalScrollRange() - computeVerticalScrollExtent();
        if (range == 0) return false;
        if (direction < 0) {
            return offset > 0;
        } else {
            return offset < range - 1;
        }
    }

**3. 通过 LinearLayoutManager 进行一系列的计算**
这种方法极不推荐使用，过程很复杂，不过对于理解 View 的布局有很大的帮助。这种方法共分为四步，下面将网上的方法抄录如下（请读者自行验证是否也存在方法1,2同样的问题）：

- 算出一个子项的高度


	public static int getItemHeight(RecyclerView recyclerView) {  
	  int itemHeight = 0;  
	  View child = null;  
	  LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();  
	  int firstPos = layoutManager.findFirstCompletelyVisibleItemPosition(); 
	  int lastPos = layoutManager.findLastCompletelyVisibleItemPosition();  
	  child = layoutManager.findViewByPosition(lastPos);  
	  if (child != null) {   
	     RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child.getLayoutParams();   
	     itemHeight = child.getHeight() + params.topMargin + params.bottomMargin;  
	  }   
	  return itemHeight;
	}

- 算出滑过的子项的总距离


	public static int getLinearScrollY(RecyclerView recyclerView) {  
	  int scrollY = 0;  
	  LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();  
	  int headerCildHeight = getHeaderHeight(recyclerView);  
	  int firstPos = layoutManager.findFirstVisibleItemPosition();  
	  View child = layoutManager.findViewByPosition(firstPos);  
	  int itemHeight = getItemHeight(recyclerView);  
	  if (child != null) {   
	     int firstItemBottom = layoutManager.getDecoratedBottom(child);   
	     scrollY = headerCildHeight + itemHeight * firstPos - firstItemBottom;    
	     if(scrollY < 0){    
	         scrollY = 0;    
	     }  
	  }  
	  return scrollY;
	}

- 算出所有子项的总高度


	public static int getLinearTotalHeight(RecyclerView recyclerView) {    int totalHeight = 0;  
	  LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();  
	  View child = layoutManager.findViewByPosition(layoutManager.findFirstVisibleItemPosition());  
	  int headerCildHeight = getHeaderHeight(recyclerView);  
	  if (child != null) {   
	     int itemHeight = getItemHeight(recyclerView);    
	     int childCount = layoutManager.getItemCount();    
	     totalHeight = headerCildHeight + (childCount - 1) * itemHeight;  
	  }  
	  return totalHeight;
	}

- 高度作比较


	public static boolean isLinearBottom(RecyclerView recyclerView) {    
	boolean isBottom = true;  
	  int scrollY = getLinearScrollY(recyclerView);  
	  int totalHeight = getLinearTotalHeight(recyclerView); 
	  int height = recyclerView.getHeight();
	 //    Log.e("height","scrollY  " + scrollY + "  totalHeight  " +  totalHeight + "  recyclerHeight  " + height);  
	  if (scrollY + height < totalHeight) {    
	    isBottom = false;  
	  }  
	  return isBottom;
	}

## 判断 RecyclerView 拖拽 ##
这一步比较简单，直接监听滚动即可。

	RecyclerView.addOnScrollListener(new OnScrollListener() {
            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                if (newState == SCROLL_STATE_DRAGGING) {
                    // 拖拽状态，实际使用中还需要判断 加载更多 是否已显示
                }
            }

            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
            }
        });

## 推荐方法 ##
该方法在 View#canScrollVertically(int direction) 的基础上，针对上拉拖拽且有可能 items 没有填充满整个 RecyclerView 这个场景做了优化，代码如下：

	RecyclerView.addOnScrollListener(new OnScrollListener() {
            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                if (newState == SCROLL_STATE_DRAGGING && 没有触发加载更多) {
					if (RecyclerView.computeVerticalScrollOffset() > 0) {// 有滚动距离，说明可以加载更多，解决了 items 不能充满 RecyclerView 的问题及滑动方向问题
						boolean isBottom = false ;
						isBottom = RecyclerView.computeVerticalScrollExtent()
				        	+ RecyclerView.computeVerticalScrollOffset()
				        	== RecyclerView.computeVerticalScrollRange() ;
						// 也可以使用 方法2
						// isBottom = !RecyclerView.canScrollVertically(1) ;
				    	if (isBottom) {
				            // 说明滚动到底部,触发加载更多
							...
				    	}
				    }
                }
            }

            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
            }
        });

到这里，上拉的判断就处理完成了，下一篇将处理 加载更多 视图的显示。

附一张原理图：

![](/images/001_01.png)

