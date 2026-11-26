# FreeCAD 参数化建模 MCP

`freecad-design` 让 MCP 客户端通过长、宽、高三个参数创建和调整长方体模型，并将同一设计导出为 PNG 预览图、可继续编辑的 FreeCAD 文档（FCStd）或用于 CAD 软件间交换的 STEP 文件。

## 客户端接入

```json
{
  "mcpServers": {
    "freecad-design": {
      "type": "streamableHttp",
      "url": "http://freecad-mcp.yuniverse-work.svc.cluster.local:8081/mcp"
    }
  }
}
```

## 五个工具

| 工具 | 必填参数 | 用途 |
| --- | --- | --- |
| `get_rpc_status` | 无，传 `{}` | 检查 FreeCAD 的 RPC 服务是否运行、图形界面是否就绪。 |
| `create_box` | `length`、`width`、`height` | 创建独立的长方体设计，返回供后续操作使用的 `design_id`。 |
| `resize_box` | `design_id`、`length`、`width`、`height` | 调整已有长方体。即使只改变一个尺寸，也要传入全部三个目标尺寸。 |
| `export_design` | `design_id`、`format` | 导出指定格式，返回有效期为 **10 分钟**的 `download_url`。 |
| `close_design` | `design_id` | 关闭设计。已签发的下载链接在原有效期内仍可使用。 |

三个尺寸均为数值，单位是**毫米**，每个值须大于 0 且不超过 100,000。导出格式参数使用小写值 `png`、`fcstd` 或 `step`。`design_id` 的格式是 `Market_` 加 24 位小写十六进制字符；调用方应私下保存服务实际返回的值。

## 示例：创建长方体、调整高度并导出

以下 JSON 为各工具的 `arguments`。示例中的 `design_id` 为虚构值，使用时替换成第一步返回的标识。

**1. 创建模型。** 调用 `create_box`，创建长 80 mm、宽 50 mm、高 20 mm 的长方体。

```json
{ "length": 80, "width": 50, "height": 20 }
```

**2. 调整高度。** 调用 `resize_box`，把高度改为 25 mm，并传入原来的长和宽。

```json
{ "design_id": "Market_0123456789abcdef01234567", "length": 80, "width": 50, "height": 25 }
```

**3. 导出并下载。** 调用 `export_design` 导出 STEP 文件，然后从项目工作区访问返回的 `download_url`，在 10 分钟内保存文件。需要预览图时，用同一个 `design_id` 再调用一次，将 `format` 改为 `png`。

```json
{ "design_id": "Market_0123456789abcdef01234567", "format": "step" }
```

**4. 关闭设计。** 完成所需导出后调用 `close_design`。

```json
{ "design_id": "Market_0123456789abcdef01234567" }
```
