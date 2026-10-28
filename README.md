# MacWaveOrg/Deps

MacWave 4.0 的依赖库二进制存放处。与 Bottle 同源同理由，只是内容不同：
Bottle 放用户要装的软件（`jq`、`ollama`……），这里放它们链接的动态库
（`libidn2`、`libunistring`、`openssl@3`……）。

## 路径布局

与 Bottle 对称，客户端在 MacWave 仓库的 `lib/domains.py`：

```python
def dep_url(dep_name, dep_version, arch, filename):
    return f"{require_domain('deps')}/{dep_name}/{dep_version}/{arch}/{filename}"
```

```
<MACWAVE_DEPS_DOMAIN>/<依赖名>/<版本>/<架构>/<文件名>
```

例如：

```
deps/libidn2/2.3.7/arm64/libidn2-2.3.7-darwin-arm64.tar.gz
```

文件名同样以索引（`infosource` 的 `deps/depsinfo_<架构>/<名字>/_<名字>@<版本>`）里的
`file:` 字段为准，`sha256` 是这里那份文件的校验值。

## 为什么单独一个仓库

- 依赖比软件包变化慢、体积也大，混在一起会让「上一个包」变成拉一整个仓库的历史；
- 依赖是**共享**的（多个包链到同一份 `libidn2`），单独存放才方便看清谁引用了谁；
- 依赖出问题时影响面更大：一个被替换的 `libssl` 会同时影响所有用到它的包，
  单独一个仓库才好做审查与回滚。

## 目录

```
deps/
    <依赖名>/
        <版本>/
            <架构>/           arm64 或 amd64
                <文件名>      与索引 file 字段完全一致
```

`lib/domains.py` 里还有 `MACWAVE_DEPS_DOMAIN` 未设时的报错路径：4.0 没有「回落到上游」
这一说，缺域名就直接停下并说明怎么设。这是刻意的——静默回落到上游等于把 4.0 想解决的
三个问题又带了回来，而且是以一种没人察觉的方式。

GHCR 与普通路径的差别见 [Bottle 的 README](../Bottle/README.md#一个待定的问题ghcr)，两者一起决定。
