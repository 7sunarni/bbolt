# cursor

search() 
传入 key 和 pageid 进行二分查找

首先通过 pageid 找到 pageid 对应的 page 和 node
首先返回在内存中的 page, 如果内存中不存在，从磁盘读取。
这里的 page 是在 tx 对象中的。

如果 page 是 leafPage, 直接比较其中的元素是否存在
如果 page 是 branchPage, 把其中的 inode 的key来拿做比较，如果key等于的话，直接返回，否则去搜索 inode 的下一个页面 id。
