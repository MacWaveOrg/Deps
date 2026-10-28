# MacWaveOrg/Deps

MacWave 4.0 的依赖库存放处。与 Bottle 同源同理由，只是内容不同：
Bottle 放用户要装的软件（`jq`、`ollama`……），这里放它们链接的动态库
（`libidn2`、`libunistring`、`openssl`……）。

## 存放形态：一个依赖一个 Release

与 Bottle 完全对称 —— 资产放在 Release 里，因为 git 单文件 100 MiB 就拒绝推送，
而 Release 允许 2 GiB 且不限总量与带宽。

```
tag   = 依赖名
资产名 = <依赖名>@<版本>.<架构><扩展名>
```

客户端拼出来的地址：

```
https://github.com/MacWaveOrg/Deps/releases/download/libidn2/libidn2%402.3.7.arm64.tar.gz
```

镜像站保留域名之后那段相对路径即可：

```
默认：https://github.com/MacWaveOrg/Deps/releases/download/libidn2/libidn2%402.3.7.arm64.tar.gz
镜像：https://mirror.example.cn/MacWave/deps/libidn2/libidn2%402.3.7.arm64.tar.gz
```

`MacWave/` 这一层是为了让镜像域名能被多个产品共用（LinuxWave 将来用 `/LinuxWave`），
所以镜像目录与 `MACWAVE_DEPS_DOMAIN` 的值都带它。

## 三条规则（与 Bottle 相同）

**只增不删** —— 旧的依赖版本必须一直可取，否则已装包的重新安装会失败。
**同名不覆盖** —— 需要重建时换版本后缀（`2.3.7-1`）。
**字节原样搬运** —— 校验值因此与上游一致，可以直接用官方 SHA256；扩展名由索引的 `file`
字段决定，客户端只改名不改内容。

## 为什么依赖单独一个仓库

- 依赖比软件包变化慢、体积也大，混在一起会让「上一个包」变成拉一整个仓库的历史；
- 依赖是**共享**的（多个包链到同一份 `libidn2`），单独存放才方便看清谁引用了谁；
- 依赖出问题时影响面更大：一个被替换的 `libssl` 会同时影响所有用到它的包，
  单独一个仓库才好做审查与回滚。

## 缺域名就报错，不回落

`lib/domains.py` 里 `MACWAVE_DEPS_DOMAIN` 没有值时会直接停下并说明怎么设，**不会回落到上游**。
这是刻意的：静默回落等于把 4.0 想解决的问题又带回来，而且是以一种没人察觉的方式。

## 谁往里放东西

`MacWaveOrg/MacWave` 的 `.github/workflows/sync-payloads.yml`。手动跑单包验证：
Actions → Sync payloads → Run workflow，`only` 填 `libidn2`。

镜像站同步与完整步骤见 MacWave 仓库的 `docs/MIRROR-SETUP.md`。
