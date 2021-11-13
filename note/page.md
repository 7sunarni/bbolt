# page
page 对应的是磁盘的数据库文件中的某一个部分，将磁盘文件按页的方式读到内存中。

page 本身可以理解为一块部分映射磁盘文件到内存的对象。page 有大量的 inode 数组对象组成。page 可以有不同的类型：branchPage 和 leafPage，对应的是b树中的叶节点和主干节点。

如果 page 是主干节点，那么里面的 inode 对象 key 存放的是索引值，inode 对象的 pageid 对应的是索引对应的数据存放的 pageid，可以通过 pageid 快速找到数据所在的 page。如果是 leafPage，那么 inode 的 key，value 对应的是真正存放的 key,value 数据，其中的 pageid 是没有意义的。可以通过 inode 的 flag 来区分 inode 存放的数据类型。 branchPage 是由叶节点分裂出来的，branchPage 的 key 是节点的第一个 inode 的key node.go L381

