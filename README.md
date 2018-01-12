### Table of Contents
- [1. kiss_android_issue](#1_kiss_android_issue)
- [2. Android_Threading_Performance](#2_Android_Threading_Performance)
- [3. ubuntu16.04_cannot start emulator](#3_ubuntu16.04_cannot_start_emulator)
- [4. 分析Messaging数据库](#4_分析Messaging数据库)
- [5. RecyclerView一个异常问题](#5_RecyclerView一个异常问题)

# 1. kiss_android_issue

拥抱android各种技术，再来一个深深的kiss

# 2. Android_Threading_Performance

[![alt text](https://vthumb.ykimg.com/0541040856CD0EEC6A0A490451CEE5A5)](http://player.youku.com/embed/XMTQ4MDU3Nzc3Mg==)

# 3. ubuntu16.04_cannot_start_emulator

升级到android8.1的sdk后，发现模拟器竟然无法启动了
解决方法如下：
```
用命令行启动avd发现是Could not launch './qemu/linux-x86_64/qemu-system-i386': No such file or directory错误

cd sdk-path/emulator/lib64/libstdc++
mv libstdc++.so.6 libstdc++.so.6.bak
ln -s /usr/lib64/libstdc++.so.6 sdk-path/emulator/lib64/libstdc++  


```

# 4. 分析Messaging数据库
``
参考http://www.sqlitetutorial.net/sqlite-index/
``
查看conversations表的索引
```
sqlite> .indices conversations
index_conversations_archive_status
index_conversations_sms_thread_id
index_conversations_sort_timestamp
```

查Messaging数据库里所有的索引
```
sqlite> SELECT * FROM sqlite_master WHERE type = 'index';
type        name                             tbl_name      rootpage    sql       
----------  -------------------------------  ------------  ----------  ----------
index       sqlite_autoindex_participants_1  participants  9                     
index       sqlite_autoindex_conversation_p  conversation  11                    
index       index_conversations_sms_thread_  conversation  12          CREATE IND
index       index_conversations_archive_sta  conversation  13          CREATE IND
index       index_conversations_sort_timest  conversation  14          CREATE IND
index       index_messages_sort              messages      15          CREATE IND
index       index_messages_status_seen       messages      16          CREATE IND
index       index_parts_message_id           parts         17          CREATE IND
index       index_conversation_participants  conversation  18          CREATE IND
Run Time: real 0.000 user 0.000000 sys 0.000000
```

数据库索引的思考
1. 索引有助于加快 SELECT 查询和 WHERE 子句，但它会减慢使用 UPDATE 和 INSERT 语句时的数据输入。索引可以创建或删除，但不会影响数据
2. 索引不应该使用在较小的表上
3. 索引不应该使用在有频繁的大批量的更新或插入操作的表上
4. 索引不应该使用在含有大量的 NULL 值的列上
5. 索引不应该使用在频繁操作的列上

..测试INDEX..
```
sqlite> EXPLAIN QUERY PLAN SELECT * FROM conversations where sms_thread_id='106';
0|0|0|SEARCH TABLE conversations USING INDEX index_conversations_sms_thread_id (sms_thread_id=?)
sqlite> SELECT * FROM conversations where sms_thread_id='106';
5|106|测试5|6|【格瓦拉】6月27日周五16:00国金百丽宫影院变形金刚ⅣH排9座等2张票已定，凭码2077777777777至影院自助取票机取票||||0|||||0|1512541613485|0|messaging://avatar/r?m=content%3A%2F%2Fcom.android.contacts%2Fcontacts%2F5%2Fphoto&f=messaging%3A%2F%2Favatar%2Fl%3Fn%3D%25E6%25B5%258B%25E8%25AF%25955%26i%3D3176r5-55A087031C|5|3176r5-55A087031C|10657109080335|1|1|1||1|0|
```

# 5. RecyclerView一个异常问题

调用notifyItemRemoved(0)后引起的问题 (ps:正常的使用无论是items数组 or cursor都是没有问题的，
因为在里面动态加入删除一些type及影响了itemcount才引起的问题)：
```
java.lang.IndexOutOfBoundsException: Inconsistency detected. Invalid item position 0(offset:-1).state:112 android.support.v7.widget.RecyclerView$Recycler.getViewForPosition(RecyclerView.java:4955)
android.support.v7.widget.RecyclerView$Recycler.getViewForPosition(RecyclerView.java:4913)
android.support.v7.widget.LinearLayoutManager$LayoutState.next(LinearLayoutManager.java:2029)
android.support.v7.widget.LinearLayoutManager.layoutChunk(LinearLayoutManager.java:1414)
android.support.v7.widget.LinearLayoutManager.fill(LinearLayoutManager.java:1377)
android.support.v7.widget.LinearLayoutManager.onLayoutChildren(LinearLayoutManager.java:588)
android.support.v7.widget.RecyclerView.dispatchLayoutStep1(RecyclerView.java:3211)
android.support.v7.widget.RecyclerView.dispatchLayout(RecyclerView.java:3067)
android.support.v7.widget.RecyclerView.consumePendingUpdateOperations(RecyclerView.java:1505)
```
分析后发现：
```
       View getViewForPosition(int position, boolean dryRun) {
            if (position < 0 || position >= mState.getItemCount()) {
                throw new IndexOutOfBoundsException("Invalid item position " + position
                        + "(" + position + "). Item count:" + mState.getItemCount());
            }
            boolean fromScrap = false;
            ViewHolder holder = null;
            // 0) If there is a changed scrap, try to find from there
            if (mState.isPreLayout()) {
                holder = getChangedScrapViewForPosition(position);
                fromScrap = holder != null;
            }
            // 1) Find from scrap by position
            if (holder == null) {
                holder = getScrapViewForPosition(position, INVALID_TYPE, dryRun);
                if (holder != null) {
                    if (!validateViewHolderForOffsetPosition(holder)) {
                        // recycle this scrap
                        if (!dryRun) {
                            // we would like to recycle this but need to make sure it is not used by
                            // animation logic etc.
                            holder.addFlags(ViewHolder.FLAG_INVALID);
                            if (holder.isScrap()) {
                                removeDetachedView(holder.itemView, false);
                                holder.unScrap();
                            } else if (holder.wasReturnedFromScrap()) {
                                holder.clearReturnedFromScrapFlag();
                            }
                            recycleViewHolderInternal(holder);
                        }
                        holder = null;
                    } else {
                        fromScrap = true;
                    }
                }
            }
            if (holder == null) {   //Look here
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                if (offsetPosition < 0 || offsetPosition >= mAdapter.getItemCount()) {
                    throw new IndexOutOfBoundsException("Inconsistency detected. Invalid item "
                            + "position " + position + "(offset:" + offsetPosition + ")."
                            + "state:" + mState.getItemCount());
                }
```
因为代码执行了Look here部分的代码， 才有问题。而holer为空， 说明validateViewHolderForOffsetPosition返回了false
查看validateViewHolderForOffsetPosition后发现
```
boolean validateViewHolderForOffsetPosition(ViewHolder holder) {
            // if it is a removed holder, nothing to verify since we cannot ask adapter anymore
            // if it is not removed, verify the type and id.
            if (holder.isRemoved()) {
                if (DEBUG && !mState.isPreLayout()) {
                    throw new IllegalStateException("should not receive a removed view unelss it"
                            + " is pre layout");
                }
                return mState.isPreLayout();
            }
            if (holder.mPosition < 0 || holder.mPosition >= mAdapter.getItemCount()) {
                throw new IndexOutOfBoundsException("Inconsistency detected. Invalid view holder "
                        + "adapter position" + holder);
            }
            if (!mState.isPreLayout()) {
                // don't check type if it is pre-layout.
                final int type = mAdapter.getItemViewType(holder.mPosition);
                if (type != holder.getItemViewType()) {
                    return false;
                }
            }
            if (mAdapter.hasStableIds()) {  //哦
                 return holder.getItemId() == mAdapter.getItemId(holder.mPosition);
            }
            return true;
        }
```
在哦处， 自定义的adapter确实设置了setHasStableIds(true);那么改成setHasStableIds(false)；就完事了。
下面就是深入理解setHasStableIds到底是干什么的.

```
  /**

     * Returns true if this adapter publishes a unique <code>long</code> value that can

     * act as a key for the item at a given position in the data set. If that item is relocated

     * in the data set, the ID returned for that item should be the same.

     * @return true if this adapter's items have stable IDs

     */

    public final boolean hasStableIds() {

        return mHasStableIds;

    }
```
大体意思是我们要重写getItemId方法给他一个不同位置的唯一标识，并且hasStableIds返回true的时候应该返回相同的数据集

那么用setHasStableIds(true)的好处就显而易见了。 
```
1. setHasStableIds(true)
       In RecyclerView.Adapter, we need to set setHasStableIds(true); 
       true means this adapter would publish a unique value as a key for item in data set.
       Adapter can use the key to indicate they are the same one or not after notifying data changed.

2. override getItemId(int position)
       Then we must override getItemId(int position), to return identified long for the item at position.
       We need to make sure there is no different item data with the same returned id.

3. After using stable Id, RecyclerView would try to use the same viewholder and view for the same id. 
       This would reduce blinking issue after data changed.
```
