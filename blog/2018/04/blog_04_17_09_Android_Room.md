# Android Room 理解

官网地址：https://developer.android.google.cn/training/data-storage/room?hl=zh-cn

处理大量结构化数据的应用可极大地受益于在本地保留这些数据。最常见的使用场景是缓存相关的数据，这样一来，当设备无法访问网络时，用户仍然可以在离线状态下浏览该内容。

Room 持久性库在 SQLite 上提供了一个抽象层，以便在充分利用 SQLite 的强大功能的同时，能够流畅地访问数据库。具体来说，Room 具有以下优势：

针对 SQL 查询的编译时验证。
可最大限度减少重复和容易出错的样板代码的方便注解。
简化了数据库迁移路径。
出于这些方面的考虑，我们强烈建议您使用 Room，而不是直接使用 SQLite API。


 Android 2017 IO大会推出了官方数据库框架：Room。Room其实就只是对原生的SQLite API进行了一层封装。
