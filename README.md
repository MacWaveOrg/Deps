# MacWaveOrg/Deps

MacWave 4.0 的依赖库存放处。与 Bottle 同源同理由，只是内容不同：
Bottle 放用户要装的软件（`jq`、`ollama`……），这里放它们链接的动态库
（`libidn2`、`libunistring`、`openssl@3`……）。

## 路径布局

与 Bottle 对称，也照抄 `infosource` 的目录形状，只去掉文件名前面那个下划线：

| `infosource` 的记录 | 这里的产物 |
| --- | --- |
| `deps/depsinfo_<架构>/<名字>/_<名字>@<版本>` | `deps/depsinfo_<架构>/<名字>/<名字>@<版本><扩展名>` |

客户端在 MacWave 仓库 `lib/domains.py` 的 `payload_path()` 里拼这个地址。

举例：

```
deps/depsinfo_arm64/libidn2/libidn2@2.3.7.tar.gz
deps/depsinfo_arm64/libcxx/libcxx@23.1.2.conda
```

文件名同样以索引（`deps/depsinfo_<架构>/<名字>/_<名字>@<版本>`）里的 `file:` 字段为准：
客户端取它的扩展名，拼成 `<名字>@<版本><扩展名>`。字节同样**原样搬运，不重打包**，
所以 `sha256` 通常就是上游公布的值。

## 为什么单独一个仓库

- 依赖比软件包变化慢、体积也大，混在一起会让「上一个包」变成拉一整个仓库的历史；
- 依赖是**共享**的（多个包链到同一份 `libidn2`），单独存放才方便看清谁引用了谁；
- 依赖出问题时影响面更大：一个被替换的 `libssl` 会同时影响所有用到它的包，
  单独一个仓库才好做审查与回滚。

## 缺域名就报错，不回落

`lib/domains.py` 里 `MACWAVE_DEPS_DOMAIN` 未设时直接停下并说明怎么设，
**不会回落到上游**。这是刻意的：静默回落等于把 4.0 想解决的问题又带了回来，
而且是以一种没人察觉的方式。

GHCR 与普通路径的差别见 [Bottle 的 README](../Bottle/README.md#一个待定的问题ghcr)，
两者要一起决定。

## 目录

```
deps/
    depsinfo_arm64/
        <依赖名>/
            <依赖名>@<版本><扩展名>
    depsinfo_amd64/
        ...
```

空目录用 `.gitkeep` 占位；真正的文件由 CI 放进来。
