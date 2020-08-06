## 故障排查

#### 升级后监控失效

从旧版本升级时，出于安全原因禁用内置 `logstash_system` 用户。 要恢复监控：

1. 更改 `logstash_system` 密码：

```sh
PUT _xpack/security/user/logstash_system/_password
{
  "password": "newpassword"
}
```

2. 重新启用 `logstash_system` 用户：

```sh
PUT _xpack/security/user/logstash_system/_enable
```