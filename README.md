# 医疗：疾病常见知识搜索

[该 MCP 服务提供疾病、疫苗、检查和治疗常见知识的检索与详情查询能力，适用于健康知识查询和医疗概念辅助理解。](https://www.handaas.com/)

## 主要功能

- 🩺 疾病病因、症状、预防和治疗知识
- 💉 疫苗接种对象、功效、禁忌和注意事项
- 🔬 医疗检查目的、准备、方法和风险
- 💊 治疗适用疾病、准备、风险和康复知识
- 🔗 相关疾病、医院、科室、检查和治疗信息

## 推荐调用流程

1. 根据用户问题提取资料关键词、资料分类和就诊科室。
2. 调用 `medical_resource_search` 查找候选知识条目。
3. 从结果中取得 `medicalInfoId` 和 `medicalType`。
4. 调用 `medical_resource_detail` 查询完整知识内容。

详情查询时必须同时传入 ID 和正确的资料分类；不要根据中文名称自行构造 ID。

## 服务设计说明

- 服务提供 2 个只读 Tool，统一覆盖疾病、疫苗、检查和治疗四类资料。
- 中文分类会映射为 Product API 使用的 `Disease`、`Vaccine`、`Check`、`Treatment`。
- 所有成功响应统一使用 `total/resultList` 顶层；详情命中时通常 `total=1`。
- 内容仅用于常见知识查询，不替代医生诊断、处方或紧急医疗服务。

## 环境要求

- Python 3.10+
- 依赖：`mcp`、`python-dotenv`、`requests`

## 本地快速启动

```bash
cd medical-knowledge-mcp-server
python3 -m venv mcp_env
source mcp_env/bin/activate
pip install -r requirements.txt
cp .env.example .env
python server/mcp_server.py streamable-http
```

服务默认地址：`http://127.0.0.1:8000/mcp`。支持 `stdio`、`sse` 和 `streamable-http`。

## 环境变量

```env
INTEGRATOR_ID=your_integrator_id
SECRET_ID=your_secret_id
SECRET_KEY=your_secret_key
HANDAAS_REQUEST_TIMEOUT=30
```

## 可用 Tool 与 Product ID

| MCP Tool | 使用场景 | Product ID | 关键输入 |
| --- | --- | --- | --- |
| `medical_resource_search` | 检索疾病、疫苗、检查和治疗知识 | `6a7b0faadfa4a2f37b191166` | 关键词、资料分类、科室、分页 |
| `medical_resource_detail` | 查询知识条目的完整内容和关联资料 | `6a7b12b08c8d379dd486bb59` | `medicalId`、`medicalType` |

## Tool 详细说明

### 1. medical_resource_search

**何时使用**：用户询问某种疾病、疫苗、医疗检查或治疗方式时使用，也是详情查询的入口。

**关键参数**：

- `keyword`：资料名称或内容关键词，例如“流感”“乙肝疫苗”。
- `medicalType`：可选值为“疾病”“疫苗”“检查”“治疗”。
- `medicalDepartmentList`：就诊科室筛选值，多级科室可用逗号分隔。
- `pageIndex`：从 1 开始；`pageSize` 默认 10、最大 50。

**返回说明**：返回资料名称、分类、别名、科室和简介等摘要，并提供 `medicalInfoId` 与 `medicalType` 供详情查询。

### 2. medical_resource_detail

**何时使用**：已经确认知识条目，需要查看疾病病因症状、疫苗禁忌、检查准备或治疗风险等完整内容时使用。

**关键参数**：

- `medicalId`：必填，使用搜索结果中的 `medicalInfoId`。
- `medicalType`：必填，必须与搜索结果一致；支持疾病/疫苗/检查/治疗及对应英文值。

**返回说明**：不同分类返回对应的完整知识字段以及相关疾病、医院、检查或治疗信息。

## 使用示例

### 示例一：查询疾病知识

用户问题：

> 搜索流感相关的疾病知识，先看 5 条。

建议调用：

```json
{
  "tool": "medical_resource_search",
  "arguments": {
    "keyword": "流感",
    "medicalType": "疾病",
    "pageIndex": 1,
    "pageSize": 5
  }
}
```

### 示例二：查询疾病详情

用户问题：

> 查看刚才疾病的病因、症状、预防和治疗方法。

建议调用：

```json
{
  "tool": "medical_resource_detail",
  "arguments": {
    "medicalId": "从知识搜索结果取得",
    "medicalType": "疾病"
  }
}
```

### 示例三：查询疫苗知识

用户问题：

> 查询乙肝疫苗的接种对象、禁忌和注意事项。

先调用 `medical_resource_search` 并设置 `medicalType="疫苗"`，确认条目后再调用详情 Tool。

## 使用注意事项

1. `medicalId` 和 `medicalType` 必须来自同一条搜索结果。
2. 搜索词过宽时优先增加 `medicalType` 或科室条件。
3. 返回内容是知识资料，不用于自动诊断或替代专业医疗意见。

## 测试验证

```bash
python -m py_compile server/mcp_server.py
python -m unittest discover -s tests -v
```

单元测试使用 Mock 响应；真实验证通过本地 `.env` 调用 HandaaS Integrator Gateway。
