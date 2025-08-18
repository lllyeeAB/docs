# DIC Browser SDK

基于 Chromium(DIC) 的浏览器 SDK，提供指纹管理和进程控制功能，适用于浏览器自动化和反检测场景。

## 🚀 快速开始

### 安装

```bash
npm install dic-browser-sdk
```

### 基础使用

```javascript
const { createSDK } = require('dic-browser-sdk');
const path = require('path');

async function main() {
  // 1. 创建SDK实例
  const sdk = createSDK();
  
  // 2. 初始化SDK
  await sdk.initialize({
    baseDir: path.join(__dirname, 'data'),
    chromiumPath: '/path/to/chromium.exe', // 指定浏览器内核路径
    logLevel: 'info'
  });
  
  // 3. 创建指纹配置
  const { instanceId, fingerprintConfig } = await sdk.createFingerprint({
    userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    proxy: {
      host: 'proxy.example.com',
      port: 8080,
      type: 'HTTP',
      username: 'user',
      password: 'pass'
    },
    fingerprint: {
      canvas: { type: 'noise' },
      rtc: { type: 'disable' }
    }
  });
  
  // 4. 启动浏览器实例
  const { id, wsUrl } = await sdk.launch({
    instanceId: instanceId,
    fingerprintConfig: fingerprintConfig.config
  });
  
  console.log(`浏览器启动成功: ${id}`);
  console.log(`WebSocket URL: ${wsUrl}`);
  
  // 5. 关闭实例
  await sdk.close(id);
  
  // 6. 清理资源
  await sdk.cleanup();
}

main().catch(console.error);
```

## 📚 主要功能

### 1. SDK 初始化

```javascript
await sdk.initialize({
  baseDir: './browser-data',           // 数据目录
  chromiumPath: '/path/to/browser',    // 浏览器路径
  logLevel: 'info'                     // 日志级别
});
```

### 2. 指纹配置

#### 基础指纹配置
```javascript
const fingerprint = await sdk.createFingerprint({
  userAgent: 'Mozilla/5.0 ...',
  iconHintText: '1',
  proxy: {
    host: 'proxy.com',
    port: 8080,
    type: 'HTTP',
    username: 'user',
    password: 'pass',
	...,
    ipInfo: {
      timeZone: 'Asia/Shanghai',
      ip: '255.23.41.3',
      ...
    }
  }
});
```

#### 高级指纹配置
```javascript
const fingerprint = await sdk.createFingerprint({
  userAgent: 'Mozilla/5.0 ...',
  fingerprint: {
    canvas: { type: 'noise' },          // Canvas指纹
    rtc: { type: 'disable' },           // WebRTC
    audio: { type: 'noise' },           // 音频指纹
    font: { type: 'auto' },             // 字体
    deviceMemory: { 
      type: 'custom', 
      value: '8' 
    },
    hardwareConcurrency: { 
      type: 'custom', 
      value: '8' 
    },
    ratio: {
      type: 'custom',
      value: { width: '1920', height: '1080' }
    }
  },
  advancedConfig: {
    restoreLast: 'enable',              // 恢复上次会话
  },
  accounts: [{                          // 账号信息
    platform: 'example.com',
    username: 'user123',
    password: 'pass123'
  }]
});
```

### 3. 浏览器实例管理

#### 启动实例
```javascript
const { id, wsUrl } = await sdk.launch({
  instanceId: 'my-instance',           // 可选，不提供则自动生成
  fingerprintConfig: fingerprint.config
});
```

#### 实例状态查询
```javascript
// 获取单个实例信息
const instance = sdk.getInstance('instance-id');
console.log(instance.status); // running, stopped, error 等

// 获取所有实例
const allInstances = sdk.getAllInstances();

// 检查实例是否存活
const isAlive = sdk.isInstanceAlive('instance-id');

// 获取SDK状态
const status = sdk.getStatus();
console.log(status.totalInstances, status.runningInstances);
```

#### 关闭实例
```javascript
// 普通关闭
await sdk.close('instance-id');
```

## 🔧 高级用法

### 代理配置
```javascript
// HTTP代理
proxy: {
  host: 'proxy.com',
  port: 8080,
  type: 'HTTP',
  username: 'user',
  password: 'pass'
}

// SOCKS5代理
proxy: {
  host: 'socks.com',
  port: 1080,
  type: 'SOCKS5',
  username: 'user',
  password: 'pass'
}
```

### 事件监听
```javascript
sdk.on('instanceLaunched', (instance) => {
  console.log('实例启动:', instance.id);
});

sdk.on('instanceClosed', ({ instanceId }) => {
  console.log('实例关闭:', instanceId);
});

sdk.on('processExit', (data) => {
  console.log('进程退出:', data.pid);
});
```

### 批量管理
```javascript
// 启动多个实例
const instances = [];
for (let i = 0; i < 5; i++) {
  const fingerprint = await sdk.createFingerprint({
    userAgent: `UserAgent-${i}`,
    // ... 其他配置
  });
  
  const instance = await sdk.launch({
    fingerprintConfig: fingerprint.config
  });
  
  instances.push(instance);
}

// 批量关闭
for (const instance of instances) {
  await sdk.close(instance.id);
}
```

## 🛠️ 环境要求

- Node.js >= 18.0.0
- Windows/macOS
- 可用的 Chromium 内核

## 📖 错误处理

```javascript
try {
  await sdk.launch(options);
} catch (error) {
  console.error('启动失败:', error.message);
  console.error('错误码:', error.code);
}
```

常见错误码参考：[ERROR_CODES.md](./ERROR_CODES.md)<br>
指纹类型定义：[TYPE.md](./TYPE.md)

## 🔍 调试

启用详细日志：
```javascript
await sdk.initialize({
  baseDir: './data',
  logLevel: 'debug'  // error, warn, info, debug
});
```

## 📄 许可证

MIT License
