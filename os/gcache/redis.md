
# `Redis`缓存

`GF`框架通过社区模块提供了`gcache`的`Redis`缓存适配实现，仓库地址：https://github.com/gogf/gcache-adapter 使用方式请参考仓库`README`介绍。


`Redis`缓存在多节点保证缓存的数据一致性时非常有用，特别是`Session`共享, 数据库查询缓存等场景中。其中，数据库ORM也内置支持了缓存适配功能，具体示例请参考：[ORM查询缓存](database/gdb/model/cache.md) 章节。















