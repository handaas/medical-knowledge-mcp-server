# 医疗：疾病常见知识搜索

[该MCP服务提供疾病、疫苗、医疗检查和治疗方式等常见医疗知识的搜索与详情查询功能。](https://www.handaas.com/)


## 主要功能

- 🩺 疾病病因、症状、预防和治疗知识
- 💉 疫苗接种对象、功效、禁忌和注意事项
- 🔬 医疗检查目的、准备、方法和风险
- 💊 治疗适用疾病、准备、风险和康复知识
- 🔗 相关疾病、医院、检查和治疗信息

## 环境要求

- Python 3.10+
- 依赖包：python-dotenv, requests, mcp

## 本地快速启动

### 1. 克隆项目
```bash
git clone https://github.com/handaas/medical-knowledge-mcp-server
cd medical-knowledge-mcp-server
```

### 2. 创建虚拟环境&安装依赖

```bash
python -m venv mcp_env && source mcp_env/bin/activate
pip install -r requirements.txt
```

### 3. 环境配置

复制环境变量模板并配置：

```bash
cp .env.example .env
```

编辑 `.env` 文件，配置以下环境变量：

```env
INTEGRATOR_ID=your_integrator_id
SECRET_ID=your_secret_id
SECRET_KEY=your_secret_key
HANDAAS_REQUEST_TIMEOUT=30
```

### 4. streamable-http启动服务

```bash
python server/mcp_server.py streamable-http
```

服务将在 `http://localhost:8000` 启动，MCP 地址为 `http://127.0.0.1:8000/mcp`。

#### 支持启动方式 stdio 或 sse 或 streamable-http

### 5. Cursor / Cherry Studio MCP配置

```json
{
  "mcpServers": {
    "medical-knowledge-mcp-server": {
      "type": "streamableHttp",
      "url": "http://127.0.0.1:8000/mcp"
    }
  }
}
```

## STDIO版安装部署

### 设置Cursor / Cherry Studio MCP配置

```json
{
  "mcpServers": {
    "medical-knowledge-mcp-server": {
      "command": "uv",
      "args": ["run", "mcp", "run", "{workdir}/server/mcp_server.py"],
      "env": {
        "PATH": "{workdir}/mcp_env/bin:$PATH",
        "PYTHONPATH": "{workdir}/mcp_env",
        "INTEGRATOR_ID": "your_integrator_id",
        "SECRET_ID": "your_secret_id",
        "SECRET_KEY": "your_secret_key"
      }
    }
  }
}
```

## 使用官方Remote服务

当前服务暂未开放官方 Remote 地址。请使用本地 `stdio`、`sse` 或 `streamable-http` 方式运行；官方 Remote 地址开放后将在本节补充配置。

### 注意：integrator_id、secret_id、secret_key及Remote token需要登录 https://www.handaas.com/ 进行注册开通平台获取


## 可用工具

### 1. medical_resource_search
**功能**: 疾病及常见医疗知识搜索

**参数**:
根据关键词、资料分类和就诊科室分页搜索疾病、疫苗、检查或治疗知识。

- `keyword` (可选): 资料名称或内容关键词
- `medicalType` (可选): 资料分类，可选疾病、疫苗、检查、治疗
- `medicalDepartmentList` (可选): 就诊科室，多级科室用逗号分隔
- `pageIndex` (可选): 页码，从1开始，默认1
- `pageSize` (可选): 每页条数，默认10，最大50

**返回值**:
- `total`: 符合条件的知识条目总数
- `resultList`: 知识条目列表
  - `medicalInfoId`: 医疗资料ID，供详情查询使用
  - `medicalName`: 资料名称
  - `medicalType`: 资料分类
  - `medicalAlias`: 资料别名
  - `medicalDepartmentList`: 就诊科室
  - `medicalIntroduction`: 资料简介
  - `medicalComplications`: 并发症
  - `diseaseProneGroup`: 高发人群
  - `relatedDisease`: 相关疾病
  - `highlight`: 关键词命中高亮信息

### 2. medical_resource_detail
**功能**: 疾病及常见医疗知识详情查询

**参数**:
通过医疗资料ID和资料分类查询完整知识内容。

- `medicalId` (必需): 使用搜索结果中的 `medicalInfoId`
- `medicalType` (必需): 必须与搜索结果分类一致，支持疾病、疫苗、检查、治疗及对应英文值

**返回值**:
- `total`: 命中数量，通常为1
- `resultList`: 分类对应的完整知识详情
  - 疾病类：病因、症状、预防、检查、治疗及相关疾病
  - 疫苗类：接种对象、功效、指导、反应、禁忌和注意事项
  - 检查类：检查目的、准备、部位、方法、禁忌和风险
  - 治疗类：适用疾病、治疗准备、并发症、风险、饮食和康复
  - 关联信息：相关医院、疾病、检查和治疗列表

## 使用注意事项

1. **ID与分类要求**: `medicalId` 和 `medicalType` 必须来自同一条搜索结果。
2. **分类映射**: 中文分类会映射为 Product API 使用的 Disease、Vaccine、Check、Treatment。
3. **API限制**: 分页查询一页最多获取50条数据。
4. **合法空结果**: 未命中时返回 `total=0` 和 `resultList=[]`。
5. **医疗边界**: 返回内容用于常见知识查询，不替代医生诊断、处方或紧急医疗服务。

## 使用提问示例

### medical_resource_search (疾病及常见医疗知识搜索)
1. 搜索流感相关的疾病知识
2. 查询乙肝疫苗的常见知识
3. 查找心内科相关的医疗检查资料

### medical_resource_detail (疾病及常见医疗知识详情查询)
1. 查看刚才疾病的病因、症状和预防方法
2. 查询这项检查需要做哪些准备
3. 查看该疫苗的接种禁忌和注意事项

## 测试验证

```bash
python -m py_compile server/mcp_server.py
python -m unittest discover -s tests -v
```

当前服务包含 5 个离线单元测试。真实接口验证需在本地 `.env` 配置有效凭据后执行，测试输出不得提交真实业务响应或凭据。
