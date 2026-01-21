# DIC Browser SDK

基于 Chromium(DIC) 的浏览器 SDK，提供指纹管理和进程控制功能，适用于浏览器自动化和反检测场景。

## 🚀 快速开始

### 安装

1.将 **dic-browser-sdk.zip** 解压在项目根目录下（或者你喜欢的其他任何地方），修改 package.json：

```bash
  "dependencies": {
    "dic-browser-sdk": "file:./dic-browser-sdk"
  }
```

2.安装 SDK 所需依赖（在你项目根目录中执行）

```bash
npm install
```

3.完成

### 基础使用

```javascript
const { createSDK } = require('dic-browser-sdk');
const path = require('path');

async function main() {
  // 1. 创建SDK实例
  const sdk = createSDK();

  // 2. 初始化SDK
  await sdk.initialize({
    key: 'Your usage sdk key', // sdk key 必须
    baseDir: path.join(__dirname, 'data'),
    chromiumPath: '/path/to/chromium.exe', // 指定浏览器内核路径
    logLevel: 'info',
  });

  // 3. 创建指纹配置
  const { instanceId, fingerprintConfig } = await sdk.createFingerprint({
    userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    proxy: {
      host: 'proxy.example.com',
      port: 8080,
      type: 'HTTP',
      username: 'user',
      password: 'pass',
    },
    fingerprint: {
      canvas: { type: 'noise' },
      rtc: { type: 'disable' },
    },
  });

  // 4. 启动浏览器实例
  const { id, wsUrl } = await sdk.launch({
    instanceId: instanceId,
    fingerprintConfig: fingerprintConfig.config,
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
  baseDir: './browser-data', // 数据目录
  chromiumPath: '/path/to/browser', // 浏览器路径
  logLevel: 'info', // 日志级别
});
```

### 2. 指纹配置

#### 指纹类型解释

注意：指纹中提到的所有“基于 ip”指的是基于业务创建指纹时透传的 proxy.ipInfo 信息，并不是指本机 ip；目前当设置为基于 ip 的指纹项，会自动使用 proxy.ipInfo 信息作为指纹支撑（lang、acceptlang、timeZone、geo）。
关于 ipInfo 字段透传，[参考这里](http://ip-api.com/json) 检测结果中的字段即可（注意该站 https 会失败，改为 http 访问）。

```typescript
/**
 * 指纹相关类型定义
 */

// 指纹模式类型
// noise = 噪音（SDK内部处理）
// truth = 真实（SDK内部处理）
// custom = 自定义（部分指纹有特殊配置能力）
export type FingerprintMode = 'noise' | 'truth' | 'custom';

// ip指纹类型
// ip = 代表基于ip信息(proxy.ipInfo)
// custom = 代表自定义传入(此时传对应的value)
export type IpFingerprintType = 'ip' | 'custom';

// 界面语言类型
// for-acceptLang = 来自语言项（继承）
// truth = 真实
// custom = 代表自定义传入(此时传对应的value)
export type LangType = 'for-acceptLang' | 'truth' | 'custom';

// 分辨率类型
// truth = 真实
// random = 随机
// custom = 自定义（此时传value具体的分辨率，RatioValue）
export type RatioMode = 'custom' | 'truth' | 'random';

// 分辨率配置
export interface RatioValue {
  width: string;
  height: string;
}

// WebGpu类型
// inWebGL = 基于WebGL
// truth = 真实
// disable = 禁止WebGpu
export type WebGPUType = 'inWebGL' | 'truth' | 'disable';

// 地理位置类型
export type GeoType =
  | 'ask' // 询问
  | 'ask-ip' // 询问-基于ip
  | 'ask-custom' // 询问-自定义（此时建议传具体的地理位置）
  | 'allow' // 允许
  | 'allow-ip' // 允许-基于ip
  | 'allow-custom' // 允许-自定义（此时建议传具体的地理位置）
  | 'block'; // 禁止

// 时区类型
// ip = 基于ip
// custom = 自定义（传value为时区）
export type TimezoneType = 'ip' | 'custom';

// 字体类型
// auto = 自动
// custom = 自定义（此时需要传value为字体列表的数组，如["Abyssinica SIL","AnjaliOldLipi", "Apple Braille Pinpoint 6 Dot"]等）
export type FontType = 'auto' | 'custom';

// WebRTC类型
// disable = 禁止
// fake = 替代（此时需要声明value为内网ip）
// truth = 真实
export type RTCType = 'disable' | 'fake' | 'truth';

// 设备内存值
export type DeviceMemoryValue = '2' | '4' | '8';

// 硬件并发数值
export type HardwareConcurrencyValue =
  | '2'
  | '3'
  | '4'
  | '6'
  | '8'
  | '10'
  | '12'
  | '16'
  | '20'
  | '24';

// 额外的业务透传信息，基于用户提供的代理信息检测的结果。（非必填项可不传）
// 目前仅有必填项（acceptLang、lang、geo、timeZone）在满足一定条件下可为对应指纹项提供支撑的。
// 可参考：http://ip-api.com/json
export interface IpInfo {
  acceptLang: string[];
  lang: string;
  geo: {
    longitude: string;
    latitude: string;
    accuracy: string;
  };
  timeZone: string;
  city?: string;
  country?: string;
  countryCode?: string;
  ip?: string;
  region?: string;
  regionCode?: string;
}

// 用户的代理信息（直接交给内核代理转发）
export interface ProxyProps {
  host: string;
  port: number;
  type: 'HTTP' | 'HTTPS' | 'SOCKS5';
  username?: string;
  password?: string;
  ipInfo?: IpInfo;
}

export interface AdvancedConfig {
  /** 是否恢复上次会话 */
  restoreLast?: 'enable' | 'disable';
  /** URL过滤配置 */
  urls?: {
    black?: string; // 黑名单
    white?: string; // 白名单
  };
  /** 扩展配置 */
  extension?: {
    disable?: boolean;
    hideIds?: string;
  };
}

/**
 * 操作系统类型 (WINDOWS:Windows MAC:Mac ANDROID:Android IOS:iOS LINUX:Linux)
 */
export enum OsType {
  Windows = 'WINDOWS',
  Mac = 'MAC',
  Android = 'ANDROID',
  Ios = 'IOS',
  Linux = 'LINUX',
}

// 账号信息
export interface Account {
  username: string;
  password: string;
  platform: string;
}

export interface FingerprintParams {
  /** 操作系统类型（默认windows） */
  uaOs?: OsType;
  /** UA */
  userAgent?: string;
  /** 序号 */
  serialNumber?: number;
  /** 图标提示文本 */
  iconHintText?: string;
  /** 代理配置 */
  proxy?: ProxyProps;
  /** 账号配置 */
  accounts?: Account[];
  /** 高级配置 */
  advancedConfig?: AdvancedConfig;
  /** 详细指纹 */
  fingerprint?: FingerprintConfig;
}

// 指纹配置
export interface FingerprintConfig {
  /** 语言配置 */
  acceptLang?: {
    type: IpFingerprintType;
    value?: string[];
  };
  /** 界面语言配置 */
  lang?: {
    type: LangType;
    value?: string;
  };
  /** 时区配置 */
  timeZone?: {
    type: IpFingerprintType;
    value?: string;
  };
  /** 地理位置配置 */
  geo?: {
    type: GeoType;
    value?: string;
  };
  /** 音频配置 */
  audio?: {
    type: FingerprintMode;
    value?: string;
  };
  /** 电池配置 */
  battery?: {
    type: FingerprintMode;
    value?: string;
  };
  /** 客户端矩形配置 */
  clientRects?: {
    type: FingerprintMode;
    value?: string;
  };
  /** 设备内存配置 */
  deviceMemory?: {
    type: FingerprintMode;
    value?: DeviceMemoryValue;
  };
  /** 字体配置 */
  font?: {
    type: FontType;
    value?: string[]; // 当type为custom时的字体标识
  };
  /** WebRTC配置 */
  rtc?: {
    type: RTCType;
    value?: string;
  };
  /** Canvas指纹配置 */
  canvas?: {
    type: FingerprintMode;
    value?: string;
  };
  /** 剪贴板配置 */
  clipboard?: {
    type: 'enable' | 'disable';
  };
  /** 密码提示 */
  passwordHint?: boolean;
  /** WebGL配置 */
  webgl?: {
    type: 'custom' | 'truth';
    vendor?: string;
    renderer?: string;
  };
  /** WebGPU配置 */
  webGPU?: {
    type: WebGPUType;
    value?: string;
  };
  webGPUImage?: {
    type: FingerprintMode;
    value?: string;
  };
  /** 硬件并发数 */
  hardwareConcurrency?: {
    type: FingerprintMode;
    value?: HardwareConcurrencyValue;
  };
  /** 媒体设备配置 */
  mediadevice?: {
    type: FingerprintMode;
    value?: string;
  };
  /** 是否开启硬件加速 */
  accelerate?: boolean;
  /** 分辨率 */
  ratio?: {
    type: RatioMode;
    value?: RatioValue;
  };
  /** 窗口配置 */
  windowConfig?: {
    width: number;
    height: number;
  };
  /** 语音指纹 */
  speechvoices?: {
    type: FingerprintMode;
    value?: string;
  };
  /** 追踪 */
  track?: string;
  /** 启动参数 */
  startParams?: string;
  /** 端口扫描保护 */
  port?: string;
  /** 启动页面 */
  startUrl?: string[];
}
```

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
  // 详细指纹
  fingerprint: {
    canvas: { type: 'noise' }, // Canvas指纹
    rtc: { type: 'disable' }, // WebRTC
    audio: { type: 'noise' }, // 音频指纹
    font: { type: 'auto' }, // 字体
    deviceMemory: {
      type: 'custom',
      value: '8',
    },
    hardwareConcurrency: {
      type: 'custom',
      value: '8',
    },
    ratio: {
      type: 'custom',
      value: { width: '1920', height: '1080' },
    },
  },
  advancedConfig: {
    restoreLast: 'enable', // 恢复上次会话
  },
  accounts: [
    {
      // 账号信息
      platform: 'example.com',
      username: 'user123',
      password: 'pass123',
    },
  ],
});
```

### 3. 浏览器实例管理

#### 启动实例

```javascript
const { id, wsUrl } = await sdk.launch({
  instanceId: 'my-instance', // 可选，不提供则自动生成
  fingerprintConfig: fingerprint.config,
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
    fingerprintConfig: fingerprint.config,
  });

  instances.push(instance);
}

// 批量关闭
for (const instance of instances) {
  await sdk.close(instance.id);
}
```

### 环境 Cookie

```javascript
// 获取Cookie
const cookies = await sdk.getStandardCookies(instanceId);
// 设置（合并）标准格式Cookie
await sdk.setStandardCookies(instanceId, JSON.parse(cookies));
```

注意事项：实例必须为关闭状态时操作；

## 🛠️ 你需要准备

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

常见错误码参考：[ERROR_CODES.md](./docs/ERROR_CODES.md)`<br>`

## 🔍 调试

启用详细日志：

```javascript
await sdk.initialize({
  baseDir: './data',
  logLevel: 'debug', // error, warn, info, debug
});
```

## 📄 许可证

MIT License
