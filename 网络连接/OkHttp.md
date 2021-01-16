# OkHttp

### 配置清单

- dispatcher 线程调度器
- connectionPool 连接池
- interceptors 自定义拦截器
- networkInterceptors 网络调试自定义拦截器
- eventListenerFactory 事件监听器
- retryOnConnectionFailure 连接失败或请求失败是否重试
- authenticator 授权Token
- followRedirects 是否跟着重定向
- followSslRedirects 当followRedirects打开的时候，发送协议切换是否跟着重定向
- cookieJar Cookie 存储器
- cache 缓存
- dns 域名服务器
- proxy 网络代理
- proxySelector 代理选择器
- proxyAuthenticator 代理授权
- socketFactory Socket连接
- sslSocketFactory SSL Socket连接
- x509TrustManager 证书验证器 x509是证书格式
- connectionSpecs 连接标准 各种协议版本
- protocols 支持的HTTP协议
- hostnameVerifier 主机名验证 验证主机名是否一样
- certificatePinner 证书固定者 指定证书
- certificateChainCleaner 证书验证操作员 验证是否合法
- callTimeoutMillis 回调超时时间
- readTimeoutMillis 读调超时时间
- writeTimeoutMillis 写调超时时间
- pingIntervalMillis 心跳间隔 WebSocket、HTTP2场景使用
- minWebSocketMessageToCompress 压缩WebSocket消息

### 责任链模式

- RetryAndFollowUpInterceptor 错误重试与重定向拦截器
  - 前置工作：连接的准备
  - 中置工作：传递
  - 后置工作：错误重试、重定向

- BridgeInterceptor 桥接拦截器
  - 前置工作：为请求报文做完整性补充
  - 中置工作：传递
  - 后置工作：初步解读响应报文

- CacheInterceptor 缓存拦截器
  - 前置工作：查找缓存
  - 中置工作：传递
  - 后置工作：保存缓存

- ConnectInterceptor 连接拦截器
  - 前置工作：获取可用的TCP连接和HTTPS连接，这个连接可能是当初建立，也可能是之前缓存
  - 中置工作：传递
  - 后置工作：无

- CallServerInterceptor 发送拦截器
  - 前置工作：通过Socket发送请求报文
  - 后置工作：通过Socket接收响应报文

