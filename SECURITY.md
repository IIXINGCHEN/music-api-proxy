# 安全策略

## 支持的版本

我们为以下版本提供安全更新：

| 版本 | 支持状态 |
| --- | --- |
| 1.0.x | ✅ 完全支持 |
| 0.9.x | ❌ 不再支持 |
| < 0.9 | ❌ 不再支持 |

## 报告安全漏洞

### 🚨 紧急安全问题

如果您发现了可能影响用户安全的漏洞，请**不要**在公开的 GitHub Issues 中报告。

### 安全报告流程

1. **发送邮件**到 `security@example.com`
2. **包含以下信息**：
   - 漏洞的详细描述
   - 受影响的版本
   - 复现步骤
   - 潜在的影响
   - 建议的修复方案（如果有）

3. **等待确认**：我们会在 48 小时内确认收到您的报告
4. **协作修复**：我们会与您协作制定修复方案
5. **负责任披露**：在修复发布后，我们会公开致谢

### 报告模板

```
主题：[SECURITY] 安全漏洞报告 - [简要描述]

漏洞类型：[如：SQL注入、XSS、权限提升等]
影响版本：[如：v1.0.0 - v1.0.3]
严重程度：[低/中/高/严重]

详细描述：
[详细描述漏洞的技术细节]

复现步骤：
1. 
2. 
3. 

潜在影响：
[描述漏洞可能造成的影响]

建议修复：
[如果有修复建议，请提供]

联系信息：
姓名：
邮箱：
GitHub用户名：
```

## 安全最佳实践

### 部署安全

#### 1. 网络安全

```yaml
# 使用防火墙限制访问
# 只开放必要的端口
ports:
  - "5678:5678"  # API端口
  - "9090:9090"  # 监控端口（仅内网）

# 使用反向代理
nginx:
  - 启用 HTTPS
  - 设置安全头
  - 限制请求大小
  - 启用限流
```

#### 2. 容器安全

```dockerfile
# 使用非root用户
USER 1001:1001

# 只读文件系统
RUN mkdir -p /app/logs /app/cache
VOLUME ["/app/logs", "/app/cache"]

# 最小权限
COPY --chown=1001:1001 . /app
```

#### 3. Kubernetes 安全

```yaml
# 安全上下文
securityContext:
  runAsNonRoot: true
  runAsUser: 1001
  runAsGroup: 1001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
    - ALL

# 网络策略
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: unm-server-netpol
spec:
  podSelector:
    matchLabels:
      app: unm-server
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: nginx-ingress
    ports:
    - protocol: TCP
      port: 5678
```

### 配置安全

#### 1. 密钥管理

```yaml
# 不要在配置文件中硬编码密钥
security:
  jwt_secret: "${JWT_SECRET}"  # 使用环境变量
  api_key: "${API_KEY}"
  
# 使用 Kubernetes Secrets
apiVersion: v1
kind: Secret
metadata:
  name: unm-server-secrets
type: Opaque
data:
  jwt-secret: <base64-encoded-secret>
  api-key: <base64-encoded-key>
```

#### 2. CORS 配置

```yaml
# 生产环境不要使用通配符
security:
  cors_origins:
    - "https://your-domain.com"
    - "https://app.your-domain.com"
  # 不要使用: - "*"
```

#### 3. 限流配置

```yaml
performance:
  rate_limit: 100  # 每分钟请求数
  max_connections: 1000
  timeout_seconds: 30
```

### 应用安全

#### 1. 输入验证

```go
// 验证和清理用户输入
func validateMusicID(id string) error {
    if len(id) == 0 {
        return errors.New("音乐ID不能为空")
    }
    
    // 只允许数字和字母
    matched, _ := regexp.MatchString("^[a-zA-Z0-9]+$", id)
    if !matched {
        return errors.New("音乐ID格式无效")
    }
    
    if len(id) > 50 {
        return errors.New("音乐ID过长")
    }
    
    return nil
}
```

#### 2. SQL 注入防护

```go
// 使用参数化查询
func (r *Repository) FindMusic(id string) (*model.Music, error) {
    query := "SELECT * FROM music WHERE id = ?"
    row := r.db.QueryRow(query, id)
    // ...
}
```

#### 3. XSS 防护

```go
import "html"

// 转义用户输入
func sanitizeInput(input string) string {
    return html.EscapeString(input)
}
```

#### 4. 日志安全

```go
// 不要记录敏感信息
logger.Info("用户登录",
    "user_id", userID,
    // "password", password,  // 不要记录密码
    "ip", clientIP,
)

// 脱敏处理
func maskSensitiveData(data string) string {
    if len(data) <= 4 {
        return "****"
    }
    return data[:2] + "****" + data[len(data)-2:]
}
```

## 安全检查清单

### 部署前检查

- [ ] 更新所有依赖到最新版本
- [ ] 扫描容器镜像漏洞
- [ ] 配置安全头
- [ ] 启用 HTTPS
- [ ] 设置防火墙规则
- [ ] 配置限流和监控
- [ ] 使用强密码和密钥
- [ ] 禁用不必要的服务

### 运行时监控

- [ ] 监控异常访问模式
- [ ] 检查错误日志
- [ ] 监控资源使用
- [ ] 检查安全告警
- [ ] 定期安全扫描
- [ ] 备份重要数据

### 定期维护

- [ ] 更新依赖包
- [ ] 轮换密钥
- [ ] 审查访问日志
- [ ] 更新安全策略
- [ ] 进行渗透测试
- [ ] 培训团队成员

## 安全工具

### 1. 依赖扫描

```bash
# 使用 go mod 检查漏洞
go list -json -m all | nancy sleuth

# 使用 Snyk
snyk test

# 使用 OWASP Dependency Check
dependency-check --project "Music-API-Proxy" --scan .
```

### 2. 代码扫描

```bash
# 使用 gosec
gosec ./...

# 使用 staticcheck
staticcheck ./...

# 使用 CodeQL
codeql database create music-api-proxy-db --language=go
codeql database analyze music-api-proxy-db --format=csv --output=results.csv
```

### 3. 容器扫描

```bash
# 使用 Trivy
trivy image unm-server:latest

# 使用 Clair
clairctl analyze unm-server:latest

# 使用 Anchore
anchore-cli image add unm-server:latest
anchore-cli image vuln unm-server:latest os
```

### 4. 运行时保护

```yaml
# Falco 规则示例
- rule: Unexpected network connection
  desc: Detect unexpected network connections
  condition: >
    spawned_process and
    proc.name = "unm-server" and
    fd.typechar = 4 and
    fd.ip != "0.0.0.0" and
    not fd.ip in (allowed_ips)
  output: >
    Unexpected network connection
    (command=%proc.cmdline connection=%fd.name)
  priority: WARNING
```

## 事件响应

### 安全事件分类

#### 严重 (Critical)
- 远程代码执行
- 权限提升
- 数据泄露
- 服务完全不可用

#### 高 (High)
- 拒绝服务攻击
- 认证绕过
- 敏感信息泄露
- 配置错误导致的安全问题

#### 中 (Medium)
- 信息泄露
- 会话管理问题
- 输入验证问题

#### 低 (Low)
- 信息收集
- 配置建议
- 最佳实践偏差

### 响应流程

1. **检测和报告** (0-1小时)
   - 确认安全事件
   - 评估影响范围
   - 通知相关人员

2. **遏制** (1-4小时)
   - 隔离受影响系统
   - 阻止攻击继续
   - 保护证据

3. **根除** (4-24小时)
   - 识别根本原因
   - 清除恶意代码
   - 修复漏洞

4. **恢复** (24-72小时)
   - 恢复正常服务
   - 监控异常活动
   - 验证修复效果

5. **总结** (1周内)
   - 事后分析
   - 更新安全策略
   - 改进防护措施

## 联系信息

### 安全团队

- **安全邮箱**: security@example.com
- **PGP 公钥**: [如果有的话]
- **响应时间**: 48小时内确认，7天内初步响应

### 紧急联系

对于严重安全问题，请直接联系：

- **主要联系人**: security-lead@example.com
- **备用联系人**: security-backup@example.com

## 致谢

我们感谢以下安全研究人员的贡献：

- [研究人员姓名] - 发现并报告了 [漏洞类型]
- [研究人员姓名] - 改进了 [安全功能]

如果您报告了安全漏洞并希望被公开致谢，请在报告中说明。

## 法律声明

本安全策略不构成法律承诺。我们保留根据实际情况调整响应时间和处理方式的权利。

---

**最后更新**: 2025-06-28
**下次审查**: 2025-12-28
