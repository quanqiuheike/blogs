```kotlin
//公用布局设置了top
if (mDataBinding.mRecyclerView.itemDecorationCount == 0) {
    mDataBinding.mRecyclerView.addItemDecoration(VerticalGridDecoration(mLinearVerSpacing=24.dp))
}
```



```kotlin
package com.cc.commonui.tools

import android.graphics.Rect
import android.view.View
import androidx.recyclerview.widget.GridLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.cc.commonui.extension.dp

private const val TAG = "VerticalGridDecoration"

/**
 *
 * mGridOutHorMargin : 多行最左和最右的外间距
 * mGridOutVerMargin : 多行的上下外间距。第一行或最后一行是多行下使用
 * mGridHorSpacing： 多行item水平内间距
 * mGridVerSpacing : 多行item上下内间距
 *
 * mLinearVerSpacing： 单行垂直间距
 * mLinearOutVerMargin：单行的上下外边界，第一行或最后一行是多行下使用
 * mLinearOutHorMargin：单行的水平外边界
 *
 * mDiffBottomBorder ： 单行和多行的底部分界线（单行下面是多行）, 以单行为主体，与单行的bottom相邻的分界线，
 * mDiffTopBorder : 单行和多行的定部分界线（单行上面是多行）, 以单行为主体，与单行的top相邻的分界线，
 */


class VerticalGridDecoration(val mGridHorSpacing : Int= 2.dp, val mGridVerSpacing:Int = mGridHorSpacing,
                             val mGridOutHorMargin:Int =8.dp, val mGridOutVerMargin:Int = 6.dp,

                             val mLinearVerSpacing:Int=10.dp, val mLinearOutVerMargin:Int = 10.dp,
                             val mLinearOutHorMargin:Int = 0,

                             val mDiffBottomBorder:Int = 6.dp, val mDiffTopBorder:Int = 6.dp

) : RecyclerView.ItemDecoration() {

    /**
     * 每行个数
     */
    private var mSpanCount = 0

    /**
     * 头部 不显示间距的item个数
     */
    private val mStartFromSize = 0

    /**
     * 尾部 不显示间距的item个数 默认不处理最后一个item的间距
     */
    private val mEndFromSize = 0

    override fun getItemOffsets(
        outRect: Rect,
        view: View,
        parent: RecyclerView,
        state: RecyclerView.State
    ) {
        val lastPosition = state.itemCount - 1
        var position = parent.getChildAdapterPosition(view)
        if (mStartFromSize <= position && position <= lastPosition - mEndFromSize) {
            // 行，如果是横向就是列
            var spanGroupIndex = -1
            // 列，如果是横向就是行
            var column = 0
            val layoutManager = parent.layoutManager
            if (layoutManager is GridLayoutManager) {
                val spanSizeLookup = layoutManager.spanSizeLookup
                val spanCount = layoutManager.spanCount
                // 当前position的spanSize
                val spanSize = spanSizeLookup.getSpanSize(position)
                // 一行几个，如果是横向就是一列几个
                mSpanCount = spanCount / spanSize
                // =0 表示是最左边 0 2 4
                val spanIndex = spanSizeLookup.getSpanIndex(position, spanCount)
                // 列
                column = spanIndex / spanSize
                // 行 减去mStartFromSize,得到从0开始的行
                spanGroupIndex =
                    spanSizeLookup.getSpanGroupIndex(position, spanCount) - mStartFromSize
            }
            // 减掉不设置间距的position,得到从0开始的position
            position = position - mStartFromSize


            //计算位置

            if (mSpanCount == 1) {
                // 一整行不设置间距，可以自己在布局设置
                outRect.left = mLinearOutHorMargin
                outRect.right = mLinearOutHorMargin
                if (position == 0) {
                    outRect.top = mLinearOutVerMargin

                    //判断下一行是否是多行
                    if(isDiffBottomBorder(layoutManager, position)){
                        outRect.bottom = mDiffBottomBorder
                    }else{
                        outRect.bottom = mLinearVerSpacing
                    }

                } else if (position == lastPosition) {
                    outRect.bottom = mLinearOutVerMargin
                } else {

                    //判断下一行是否是多行
                    if(isDiffBottomBorder(layoutManager, position)){
                        outRect.bottom = mDiffBottomBorder
                    }else{
                        outRect.bottom = mLinearVerSpacing
                    }
                }
            } else {
                //多行下
                outRect.left = if (column == 0) {
                    mGridOutHorMargin
                } else {
                    mGridHorSpacing/2
                }
                outRect.right = if (column == 0) {
                    mGridHorSpacing/2
                } else {
                    mGridOutHorMargin
                }

                if (spanGroupIndex == 0) {
                    outRect.top = mGridOutVerMargin

                    //区分下方是否有但行
                    if(isDiffTopBorder(layoutManager, position, column)){
                        outRect.bottom = mDiffTopBorder
                    }else {
                        outRect.bottom = mGridVerSpacing
                    }

                } else if (spanGroupIndex == lastPosition - mSpanCount + 1) {
                    outRect.bottom = mGridOutVerMargin
                } else {
                    //区分下方是否有但行
                    if(isDiffTopBorder(layoutManager, position, column)){
                        outRect.bottom = mDiffTopBorder
                    }else {
                        outRect.bottom = mGridVerSpacing
                    }
                }
            }


        }
    }


    fun isDiffBottomBorder(layoutManager:RecyclerView.LayoutManager?, position:Int):Boolean{
        if(layoutManager is GridLayoutManager){
            val mSpanCount =layoutManager.spanCount / layoutManager.spanSizeLookup.getSpanSize( position + 1)
            return mSpanCount > 1
        }else{
            return false
        }
    }

    fun isDiffTopBorder(layoutManager:RecyclerView.LayoutManager?, position:Int, column:Int):Boolean{
        if(layoutManager is GridLayoutManager){
            val nextPos = position + (layoutManager.spanCount - column -1) + 1 //当前位置 + 当前行剩余位置（总格数 - 当前所处列<从0开始> -1） + 1
            val mSpanCount = layoutManager.spanCount / layoutManager.spanSizeLookup.getSpanSize(nextPos)
            return mSpanCount == 1
        }else{
            return false
        }
    }
}
```