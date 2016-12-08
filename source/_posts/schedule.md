---
title: 仿课程格子的课表界面
date: 2015-12-09 20:30:31
tags: [Android , View]
thumbnail: /images/schedule/gezi.png
---

在大学里，最常使用的课表软件大概要算课程格子和超级课程表了。课表界面简洁明了，讨人喜欢。它们是如何实现这种效果的呢？

<!--more-->

### 观察
先来看看两者的界面

* 课程格子

![gezi](/images/schedule/gezi.png)


* 超级课程表

![super](/images/schedule/super.png)

可以看到，它们都是以一种类似网格状的排列，每个网格内填充一个包含课程信息的子View。

---

### 思路

首先，肯定不能用LinearLayout或者RelativeLayout将每个包含课程信息的View一个个对应到ViewGroup里，先不说麻烦，布局文件累赘，光是看到一堆的id，头就要大。有种实现方法是将TableLayout作为主布局，然后将每行课表的子View通过addView()添加到TableRow，然后再将每行的TableRow添加到TableLayout中。感觉还是不够直接。所以，最终采用GridLayout作为课表主布局。GridLayout做课表相较于TableLayout有几个天生的优势：
* TableLayout不能同时向水平和垂直方向做控件的对齐，而GridLayout可以
* TableLayout不能跨行跨列，而GridLayout可以
这点尤为重要，比如有些课是包含两节小课，而有些课会连上四节小课，如果课程信息的子View不能跨行的化，处理起来会很笨拙

这里稍微提一下，有个GridLayout布局，还有个叫GridView的东西。GridView是一种适配器布局,它的继承关系是ViewGroup-->AdapterView-->AbsListView-->GridView，它是从一个adapter中取出内容填充到GridView中的每一个子View。而GridLayout是一个布局，它大大简化了对复杂布局的处理，提高了性能。他直接继承自ViewGroup，和LinearLayout这种是类似的。那什么时候用GridLayout，什么时候用GridView？
* 当子View布局不一样的时候，就要用GridLayout，因为gridlayout则是一个布局方式，表格里面的子布局想放什么则可以根据自己需要去写，例如Android系统自带的计算器
* 当子View的布局是一样的时候，使用GidView，因为gridview的每个子view布局使用的都是一样的，例如九宫格

为了获得更高的界面自由度，因此使用GridLayout。
使用GridLayout，先将星期，节次写好后，剩下的课程信息的View通过addView直接添加至对应位置即可

### 方案
首先，写好布局文件
注意：由于GridLayout是在API 14后引入的，若要向下兼容，请使用v7包下的GridLayout！


{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.NestedScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
<!-- 共11行8列，11行中有10行留给课表，对应1-10节小课，1行留给星期显示，8列中有7列留给课表 ，对应一周7天，1列留给节次显示-->
        <android.support.v7.widget.GridLayout
            android:id="@+id/schedule_gridlayout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:columnCount="8"
            app:orientation="vertical"
            app:rowCount="11">

            <TextView
                android:id="@+id/schedule_all_week"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:padding="3dp"
                android:text="20周"
                android:textColor="@android:color/tertiary_text_dark"
                android:textSize="13sp"
                app:layout_column="0"
                app:layout_gravity="center_horizontal"
                app:layout_row="0" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="1"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="2"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="3"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="4"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="5"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="6"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="7"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="8"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="9"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingBottom="@dimen/schedule_item_padding"
                android:paddingTop="@dimen/schedule_item_padding"
                android:text="10"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_gravity="center_horizontal" />

            <TextView
                android:text="周一"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_column="1"
                app:layout_columnWeight="2"
                app:layout_gravity="center"
                app:layout_row="0" />

            <TextView
                android:text="周二"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_column="2"
                app:layout_columnWeight="2"
                app:layout_gravity="center"
                app:layout_row="0" />

            <TextView
                android:text="周三"
                android:textColor="@android:color/tertiary_text_dark"
                app:layout_column="3"
                app:layout_columnWeight="2"
                app:layout_gravity="center"
                app:layout_row="0" />

            <TextView
                android:text="周四"
                app:layout_column="4"
                app:layout_columnWeight="2"
                app:layout_gravity="center"
                app:layout_row="0" />

            <TextView
                android:text="周五"
                app:layout_column="5"
                app:layout_columnWeight="2"
                app:layout_gravity="center"
                app:layout_row="0" />

            <TextView
                android:text="周六"
                app:layout_column="6"
                app:layout_columnWeight="1"
                app:layout_gravity="center"
                app:layout_row="0" />

            <TextView
                android:text="周日"
                app:layout_column="7"
                app:layout_columnWeight="2"
                app:layout_gravity="center"
                app:layout_row="0" />
        </android.support.v7.widget.GridLayout>
</android.support.v4.widget.NestedScrollView>
{% endcodeblock %}

接下来，逻辑代码
{% codeblock lang:java %}
  private GridLayout gridLayout;

/**
*课表界面实现
* @param data  课表数据的二维数组，与课表表格对齐
*
*/
    private void scheduleView(String[][] data) {

    //获取一些自定义颜色，用于填充课程信息子View的背景
        color[0] = getResources().getColor(R.color.Icon_half_oringe);
        color[1] = getResources().getColor(R.color.Icon_half_purple);
        color[2] = getResources().getColor(R.color.Icon_half_yellow);
        color[3] = getResources().getColor(R.color.Icon_half_green);
        color[4] = getResources().getColor(R.color.Icon_half_red);
        color[5] = getResources().getColor(R.color.Icon_half_blue);
        color[6] = getResources().getColor(R.color.Icon_half_cyan);

        //课表总字符串
        String schedulesStirng = "";
        //课程种类计数
        int scheduleCount = 0;
        //记录课程所对应的颜色值
        JSONObject scheduleToColor = new JSONObject();
        if (null != gridLayout) {
            //18为边栏固有子view数目，清空后添加子view
            gridLayout.removeViews(18, gridLayout.getChildCount() - 18);
            LayoutInflater layoutInflater = LayoutInflater.from(getContext());
            //确定每一项子view的宽度和高度，如果不进行这一步，内容将显示不正确
            WindowManager wm = (WindowManager) getContext().getSystemService(Context.WINDOW_SERVICE);
            Display display = wm.getDefaultDisplay();
            int width = display.getWidth();
            int height = display.getHeight();
            int item_width = width / 8;
            int item_height = height / 6;
            //填充子view
            for (int columnSpec = 1; columnSpec <= data.length; columnSpec++) {
                for (int rowSpec = 1; rowSpec <= data[columnSpec - 1].length; rowSpec++) {
                	//此LinearLayout为子View，只包含一个TextView
                    LinearLayout view = (LinearLayout) layoutInflater.inflate(R.layout.schedule_all_schedule_item_layout, null);
                    TextView item = (TextView) view.findViewById(R.id.schedule_content);
                    String content = data[columnSpec - 1][rowSpec - 1].trim();
                    //判断课表内容是否为空，若为空，则不填充子View
                    if (!content.equals("")) {
                        item.setText(content);
                        int position = content.indexOf("@");
                        String schedule = content.substring(0, position - 1);
                        //这里进行的是相同课程同背景色处理
                        try {
                            //判断课表名称里是否含有新进来的课
                            if (!schedulesStirng.contains(schedule)) {
                                schedulesStirng = schedulesStirng + schedule;
                                scheduleToColor.put(schedule, scheduleCount % 7);
                                view.setBackgroundColor(color[scheduleCount % 7]);
                                scheduleCount++;
                            } else {
                                view.setBackgroundColor(color[scheduleToColor.getInt(schedule)]);
                            }
                        } catch (JSONException e) {
                            e.printStackTrace();
                        }
                    }
                    GridLayout.LayoutParams param = new GridLayout.LayoutParams();
                    param.columnSpec = GridLayout.spec(columnSpec, 1);
                    param.rowSpec = GridLayout.spec(rowSpec * 2 - 1, 2);
                    param.setGravity(Gravity.FILL);
                    param.setMargins(1, 1, 1, 1);
                    param.width = item_width - 5;
                    param.height = item_height;
                    gridLayout.addView(view, param);
                }
            }
        }
    }
{% endcodeblock %}

至此，“乞丐版”超级课程表已完成，效果图：
![shuibiao](/images/schedule/shuibiao.png)

### 总结
看上去貌似很难做的样子，其实只要思路对了，做起来就不难了。本人美术功底有限，界面还有很多提升的空间，继续加油，哈哈
