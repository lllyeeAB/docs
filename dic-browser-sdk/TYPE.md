```typescript
/**
 * 指纹相关类型定义
 */

// 指纹模式类型
export type FingerprintMode = 'noise' | 'truth' | 'custom';

// 分辨率类型
export type RatioMode = 'custom' | 'truth' | 'random';

// 分辨率配置
export interface RatioValue {
  width: string;
  height: string;
}

export type WebGPUType = 'inWebGL' | 'truth' | 'disable';

// 字体类型
export type FontType = 'auto' | 'custom';

// WebRTC类型
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

/** ip信息（解析后）；访问参考：http://ip-api.com/json */
export interface IpInfo {
  /** 界面语言 */
  acceptLang: string[];
  /** 语言 */
  lang: string;
  /** 地理位置经纬度 */
  geo: {
    longitude: string;
    latitude: string;
  };
  /** 时区 */
  timeZone: string;
  /** 城市 */
  city?: string;
  /** 国家 */
  country?: string;
  /** 国家代码 */
  countryCode?: string;
  /** 出口ip */
  ip?: string;
  /** 地区 */
  region?: string;
  /** 地区代码 */
  regionCode?: string;
}

/** 代理信息 */
export interface ProxyProps {
  /** host */
  host: string;
  /** 端口 */
  port: number;
  /** 代理协议 */
  type: 'HTTP' | 'HTTPS' | 'SOCKS5';
  /** 用户名 */
  username?: string;
  /** 密码 */
  password?: string;
  /** 解析后的ip详细信息（对指纹真实度有帮助） */
  ipInfo?: IpInfo;
}

export interface AdvancedConfig {
  /** 是否恢复上次会话 */
  restoreLast?: 'enable' | 'disable';
}

export interface Account {
  /** 账号 */
  username: string;
  /** 密码 */
  password: string;
  /** 平台（如tiktok.com） */
  platform: string;
}

export interface FingerprintParams {
  /** UA */
  userAgent?: string;
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
    value?: string; // 当type为custom时的字体标识
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
  speechvoices?: {
    type: FingerprintMode;
    value?: string;
  };
  track?: string;
  /** 启动参数 */
  startParams?: string;
  /** 端口扫描保护 */
  port?: string;
  /** 启动页面 */
  startUrl?: string;
}


export interface ProcessedFingerprint {
  /** 原始指纹配置 */
  config: FingerprintParams;
  /** 创建时间 */
  createdAt: Date;
}

```
