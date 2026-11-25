# search-place-mcp · 上海地名地址查询

提供一个只读工具 `search_shanghai_places`。输入门牌地址、地名、道路名或历史地名，工具会按原词检索，并按上游顺序返回候选。道路查询还可以请求线形几何。

## 客户端接入

```json
{
  "mcpServers": {
    "search-place-mcp": {
      "type": "streamableHttp",
      "url": "http://search-place-mcp.yuniverse-work.svc.cluster.local:8080/mcp"
    }
  }
}
```

## 查询参数

| 参数 | 类型 | 用法 |
| --- | --- | --- |
| `query` | 字符串，必填 | 原始检索词，长度为 1 至 256 个字符。 |
| `limit` | 整数，可选 | 本次最多返回的候选条数，范围为 1 至 20，默认 5。 |
| `include_geometry` | 布尔值，可选 | 默认 `false`。需要道路的线形几何时传 `true`。 |

## 实测示例

向 `search_shanghai_places` 传入以下参数可查询“南京西路”，并取首条候选。

```json
{
  "query": "南京西路",
  "limit": 1
}
```

MCP 响应中的 `content[0].text` 是 JSON 字符串，解析后的业务结果如下。

```json
{
  "query": "南京西路",
  "count": 1,
  "results": [
    {
      "name": "南京西路",
      "address_code": "310106012011N009477N002887N9999",
      "type": "道路",
      "match_score": "1.0",
      "city": "上海市",
      "historical_names": "静安寺路",
      "roads": [
        { "name": "南京西路", "length_m": 3857.39 }
      ]
    }
  ]
}
```

`count` 表示上游候选总数，`results` 包含本次按 `limit` 截取的候选。首条也可能是模糊匹配，选取记录时可结合名称、类型、城市和匹配分值核对。本例中的 `historical_names` 和 `roads` 来自道路候选，解析其他类型时应按实际返回字段取值。
