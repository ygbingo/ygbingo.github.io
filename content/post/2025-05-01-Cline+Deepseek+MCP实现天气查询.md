---
title: "Cline+Deepseek+MCP实现天气查询"
date: 2025-05-01T13:54:44+08:00
draft: false
tags:
    - llm
    - MCP
    - deepseek
---

> 听说CoT模型架构过时了？现在流行MCP了？来亲自试试，看看MCP到底是个什么东西

## 什么是MCP服务
轻量级程序，通过标准的 Model Context Protocol 提供特定能力。比如下面我们要实现的查天气能力，相当于给大模型提供一个额外的技能！

MCP 服务器可以提供三种主要类型的能力：

- Resources: 可以被 clients 读取的类文件数据（如 API 响应或文件内容）
- Tools: 可以被 LLM 调用的函数（需要用户批准）
- Prompts: 预先编写的模板，帮助用户完成特定任务

## 开发一个查天气的MCP服务
我们将构建一个服务器，该服务器会公开两个工具：get-alerts（获取警报）和 get-forecast（获取天气预报）。然后我们会将该服务器连接到一个 MCP 主机。由于我在大陆，所以我是用Cline作为MCP的客户端。

### 1. 准备条件
- VS code
- Cline (因为需要一个支持MCP的大模型客户端)
- Deepseek API Key (建议，deepseek真的很便宜，也可以用别的大模型，Cline支持很多)

### 2. 开发第一个mcp server
- Node: 使用node环境
``` shell
# node需要大于16版本
node --version
npm --version

# 为我们的项目创建一个新 directory
md weather
cd weather

# 初始化一个新的 npm 项目
npm init -y

# 安装 dependencies
npm install @modelcontextprotocol/sdk zod
npm install -D @types/node typescript

# 创建我们的 files
md src
new-item src\index.ts
```

更新你的 package.json 以添加 type: “module” 和一个 build script

```json
{
  "type": "module",
  "bin": {
    "weather": "./build/index.js"
  },
  "scripts": {
    "build": "tsc && chmod 755 build/index.js"
  },
  "files": [
    "build"
  ],
}
```


在你的项目的根目录下创建一个 tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./build",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

### 3. 构建MCP服务器
编辑src\index.ts

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const NWS_API_BASE = "https://api.weather.gov";
const USER_AGENT = "weather-app/1.0";

// 创建 server instance
const server = new McpServer({
  name: "weather",
  version: "1.0.0",
  capabilities: {
    resources: {},
    tools: {},
  },
});

// Helper function 用于发送 NWS API 请求
async function makeNWSRequest<T>(url: string): Promise<T | null> {
  const headers = {
    "User-Agent": USER_AGENT,
    Accept: "application/geo+json",
  };

  try {
    const response = await fetch(url, { headers });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return (await response.json()) as T;
  } catch (error) {
    console.error("Error making NWS request:", error);
    return null;
  }
}

interface AlertFeature {
  properties: {
    event?: string;
    areaDesc?: string;
    severity?: string;
    status?: string;
    headline?: string;
  };
}

// 格式化警报数据
function formatAlert(feature: AlertFeature): string {
  const props = feature.properties;
  return [
    `Event: ${props.event || "Unknown"}`,
    `Area: ${props.areaDesc || "Unknown"}`,
    `Severity: ${props.severity || "Unknown"}`,
    `Status: ${props.status || "Unknown"}`,
    `Headline: ${props.headline || "No headline"}`,
    "---",
  ].join("\n");
}

interface ForecastPeriod {
  name?: string;
  temperature?: number;
  temperatureUnit?: string;
  windSpeed?: string;
  windDirection?: string;
  shortForecast?: string;
}

interface AlertsResponse {
  features: AlertFeature[];
}

interface PointsResponse {
  properties: {
    forecast?: string;
  };
}

interface ForecastResponse {
  properties: {
    periods: ForecastPeriod[];
  };
}


// 注册天气 tools
server.tool(
  "get-alerts",
  "获取某个州的天气警报",
  {
    state: z.string().length(2).describe("两个字母的州代码（例如 CA、NY）"),
  },
  async ({ state }) => {
    const stateCode = state.toUpperCase();
    const alertsUrl = `${NWS_API_BASE}/alerts?area=${stateCode}`;
    const alertsData = await makeNWSRequest<AlertsResponse>(alertsUrl);

    if (!alertsData) {
      return {
        content: [
          {
            type: "text",
            text: "未能检索警报数据",
          },
        ],
      };
    }

    const features = alertsData.features || [];
    if (features.length === 0) {
      return {
        content: [
          {
            type: "text",
            text: `No active alerts for ${stateCode}`,
          },
        ],
      };
    }

    const formattedAlerts = features.map(formatAlert);
    const alertsText = `Active alerts for ${stateCode}:\n\n${formattedAlerts.join("\n")}`;

    return {
      content: [
        {
          type: "text",
          text: alertsText,
        },
      ],
    };
  },
);

server.tool(
  "get-forecast",
  "获取某个位置的天气预报",
  {
    latitude: z.number().min(-90).max(90).describe("位置的纬度"),
    longitude: z.number().min(-180).max(180).describe("位置的经度"),
  },
  async ({ latitude, longitude }) => {
    // 获取网格点数据
    const pointsUrl = `${NWS_API_BASE}/points/${latitude.toFixed(4)},${longitude.toFixed(4)}`;
    const pointsData = await makeNWSRequest<PointsResponse>(pointsUrl);

    if (!pointsData) {
      return {
        content: [
          {
            type: "text",
            text: `Failed to retrieve grid point data for coordinates: ${latitude}, ${longitude}. This location may not be supported by the NWS API (only US locations are supported).`,
          },
        ],
      };
    }

    const forecastUrl = pointsData.properties?.forecast;
    if (!forecastUrl) {
      return {
        content: [
          {
            type: "text",
            text: "Failed to get forecast URL from grid point data",
          },
        ],
      };
    }

    // 获取预报数据
    const forecastData = await makeNWSRequest<ForecastResponse>(forecastUrl);
    if (!forecastData) {
      return {
        content: [
          {
            type: "text",
            text: "Failed to retrieve forecast data",
          },
        ],
      };
    }

    const periods = forecastData.properties?.periods || [];
    if (periods.length === 0) {
      return {
        content: [
          {
            type: "text",
            text: "No forecast periods available",
          },
        ],
      };
    }

    // 格式化预报 periods
    const formattedForecast = periods.map((period: ForecastPeriod) =>
      [
        `${period.name || "Unknown"}:`,
        `Temperature: ${period.temperature || "Unknown"}°${period.temperatureUnit || "F"}`,
        `Wind: ${period.windSpeed || "Unknown"} ${period.windDirection || ""}`,
        `${period.shortForecast || "No forecast available"}`,
        "---",
      ].join("\n"),
    );

    const forecastText = `Forecast for ${latitude}, ${longitude}:\n\n${formattedForecast.join("\n")}`;

    return {
      content: [
        {
          type: "text",
          text: forecastText,
        },
      ],
    };
  },
);


async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Weather MCP Server running on stdio");
}

main().catch((error) => {
  console.error("Fatal error in main():", error);
  process.exit(1);
});
```
说明: 这里会调用一个天气查询服务

### 4. 测试

先确保下面这个命令可以在本地运行
``` shell
cd weather
npm run build
```

### 5. 在vscode的Cline插件中添加MCP服务

```json

{
  "mcpServers": {
      "weather": {
          "command": "node",
          "args": [
              "<YOUR_PATH>\\weather\\build\\index.js"
          ]
      }
  }
}

```

