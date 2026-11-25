# mowan-city-mcp 服务介绍

`mowan-city-mcp` 模玩时空内置的 **City 地图** MCP 服务。它提供 7 个工具，其中 1 个按地名或地址查询两点间的直线距离，另外 6 个用于编辑当前地图上的点、线和面。

## 客户端接入

对于接受 `mcpServers` 和 `streamableHttp` 配置的客户端，可使用以下 JSON。将 `url` 填为纯文本地址即可。

```json
{
  "mcpServers": {
    "mowan-city-mcp": {
      "type": "streamableHttp",
      "url": "http://mowan-city-mcp.yuniverse-work.svc:8080/mcp"
    }
  }
}
```

## 距离查询

`city_query_distance` 接受两个必填字符串参数，`startPlace` 表示起点，`endPlace` 表示终点。两者均可填写地名或地址。根据工具说明，结果包含两端匹配到的地址、坐标，以及以公里计的地表直线距离。解读距离前，可以先用返回地址核对同名地点是否匹配正确。

查询示例

```json
{
  "startPlace": "上海虹桥站",
  "endPlace": "上海浦东国际机场"
}
```

## 地图编辑工具

| 工具 | 用途 | 主要业务参数 |
| --- | --- | --- |
| `city_add_point` | 按地址搜索位置，在当前地图上添加点位。 | `place` 指定地点，`color` 指定颜色。 |
| `city_connect_points` | 连接当前画布上两个已存在的点。 | `startName` 和 `endName` 对应两个已有点的名称。 |
| `city_rename_feature` | 按画布上的实际名称定位要素并改名。 | `currentName` 为当前名称，`newName` 为新名称。 |
| `city_update_feature_properties` | 更新名称、类别、说明或自定义属性，也可新增自定义字段。未指定的字段会保留。 | 用 `currentName` 定位。可设置 `category`、`description`、`properties` 或 `newProperties` 等字段。 |
| `city_edit_feature` | 按要素类型编辑点、线或面的指定属性。例如可修改颜色，线要素还支持宽度和线型。 | 用 `currentName` 定位，通过 `attributes` 提交对应类型的属性。 |
| `city_delete_feature` | 从当前地图草稿中删除一个按名称唯一定位的点、线或面。 | `currentName` 指定当前名称，参数结构也包含 `featureType`。 |

六个编辑工具均要求提供以下四个顶层参数。上表中的业务参数位于 `request` 内。

| 顶层参数 | 含义 |
| --- | --- |
| `map_id` | 目标地图标识 |
| `operation_id` | 本次编辑操作的标识 |
| `write_token` | 写入令牌 |
| `request` | 对应工具的编辑参数对象 |
