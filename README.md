# qgis-mcp · QGIS 矢量处理能力

`qgis-mcp` 接收 GeoJSON 或 GeoPackage 文件，提供格式转换、缓冲区分析、几何简化和坐标重投影四项工具。每次处理生成的文件标识可传给下一项工具，例如先把道路投影到以米为单位的坐标系，再生成道路两侧的缓冲区。

## 客户端接入

```json
{
  "mcpServers": {
    "qgis-mcp": {
      "type": "streamableHttp",
      "url": "http://qgis-mcp.yuniverse-work.svc.cluster.local:8080/mcp"
    }
  }
}
```

## 四项工具

| 工具 | 用途 | 专用参数 |
| --- | --- | --- |
| `convert_vector` | 将单个矢量图层转换为 GeoJSON 或 GeoPackage，保留源坐标参考系（CRS）和属性。 | 通过公共参数 `output_format` 指定目标格式。 |
| `buffer_vector` | 按指定距离生成缓冲区面。输入图层须使用以米为单位的投影坐标系。 | `distance_m` 必填，为大于 0 的距离，单位为米。`dissolve` 可选，默认 `false`，设为 `true` 可融合缓冲区面。 |
| `simplify_vector` | 用 Douglas–Peucker 算法简化线或面几何。输入图层须使用以米为单位的投影坐标系。 | `tolerance_m` 必填，为大于 0 的简化容差，单位为米。 |
| `reproject_vector` | 将图层坐标转换到指定坐标参考系。 | `target_crs` 必填，格式为 `EPSG:` 加 1 至 7 位数字，首位为 1 至 9，例如 `EPSG:4326`。 |

四个工具均接受以下公共参数。

| 参数 | 类型 | 用法 |
| --- | --- | --- |
| `file_id` | 字符串，必填 | 上传或上一步处理返回的文件标识，由 48 个小写十六进制字符组成。 |
| `output_format` | 字符串，必填 | 可取 `geojson` 或 `gpkg`。即使只做缓冲、简化或重投影，也需要指定结果格式。 |
| `layer` | 字符串或 `null`，可选 | 输入图层名，默认 `null`。对多图层 GeoPackage，可传入要处理的图层名。名称长度为 1 至 256 个字符。 |

## 文件上传、传递与下载

服务说明给出的文件接口与 MCP 端点位于同一主机。下列路径均从主机根路径开始。

1. **上传文件。** 将 GeoJSON 原始文件字节作为请求体发往 `POST /files/geojson`，或将 GeoPackage 发往 `POST /files/gpkg`，取得 `file_id`。
2. **调用工具。** 将 `file_id`、`output_format` 和专用参数传给对应的 MCP 工具。连续处理时，把上一步结果中的 `file_id` 作为下一步的输入。
3. **下载结果。** 使用同一主机对结果中的 `download_path` 发起 `GET` 请求，将文件保存到工作区。服务声明临时文件在 **600 秒（10 分钟）** 后过期，需在有效期内完成后续调用或下载。

### 示例：生成道路的 100 米缓冲区

假设道路图层使用经纬度坐标。上传文件后，先调用 `reproject_vector`，把输入图层转到适合道路所在区域、以米为单位的投影坐标系，输出格式可选 `gpkg`。再调用 `buffer_vector`，传入重投影结果的文件标识。下面的 `file_id` 是格式示例，实际调用时请替换为服务返回值。

```json
{
  "file_id": "0123456789abcdef0123456789abcdef0123456789abcdef",
  "output_format": "gpkg",
  "distance_m": 100,
  "dissolve": true
}
```

这组参数会按 100 米距离建立缓冲区，并融合缓冲区面。如果目的是仅减少道路线条的顶点数，可对米制投影后的道路调用 `simplify_vector`，用 `tolerance_m` 指定简化容差。
