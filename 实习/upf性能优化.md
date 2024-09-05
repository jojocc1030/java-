
| 字段描述                           | 新增字段                         |
| ------------------------------ | ---------------------------- |
| 新增 SegGrpMatchResult 资源对饮的资源索引 | UNIT32 segMatchRsltBitmapIdx |
| 新增 当前匹配结果对应的号段组版本号             | USHORT segMagicNum           |
| 新增 标记位标识当前用户是否命中了号段组           | Bit segMatchFlag             |
| 新增 标记位标识当前资源状态：空闲，在被读，在被写      | UCHAR segMatchRsltState      |
