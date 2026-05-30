# lsquic-prebuilt

预编译 [lsquic](https://github.com/litespeedtech/lsquic)（LiteSpeed QUIC，纯 C，自带 TLS）
静态库 + 其依赖 BoringSSL 的 `libssl/libcrypto`，供 `hook` 主项目（`System/3rd/lsquic`）下载使用。

主项目无需安装任何编译工具链（Go / NASM / cmake）；所有编译脏活都在本仓库的
GitHub Actions 多平台矩阵里完成，产物以 Release artifact 形式发布。

## 组成

- `lsquic/` — lsquic 源码（git submodule，已固定 **v4.7.1**）
- `.github/workflows/build.yml` — 多平台 CI：clone+编 BoringSSL → 编 lsquic → 打包 → 发 Release
- BoringSSL 不入库，由 CI 按固定版本 `0.20250807.0`（见 workflow `BORINGSSL_REF`）单独 clone 编译

## 产物

每个 `v*` tag 触发 CI，生成并发布以下平台包（含 `lib/` 静态库 + `include/` 头 + `VERSION.txt`）：

| 平台 | 包名 | 库 |
|------|------|----|
| Linux x64    | `lsquic-linux-x64.tar.gz`   | `liblsquic.a libssl.a libcrypto.a` |
| macOS arm64  | `lsquic-macos-arm64.tar.gz` | 同上 |
| Windows x64  | `lsquic-windows-x64.zip`    | `lsquic.lib ssl.lib crypto.lib` |

## 发布新版本

```bash
# 1. （可选）升级 lsquic 子模块到新 tag
cd lsquic && git fetch --tags && git checkout vX.Y.Z && cd ..
git add lsquic && git commit -m "bump lsquic to vX.Y.Z"

# 2. 打 tag 触发 CI 构建 + 发 Release
git tag v4.7.1      # 与 lsquic 版本对应即可
git push origin main --tags
```

CI 跑完后在 Releases 页可见四平台产物。

## 主项目（hook）如何消费

`hook/System/3rd/lsquic/fetch.sh` 按平台 + 锁定版本从本仓库 Release 下载解压到
`System/3rd/lsquic/<platform>/`。主项目构建脚本只 `-I include` + 链接 `lib/*.a`。

## 本地手动构建（调试用，平时不需要）

需要 Go + cmake（Windows 另需 NASM + Perl）：

```bash
git clone https://boringssl.googlesource.com/boringssl
cd boringssl && git checkout 0.20250807.0
cmake -DCMAKE_BUILD_TYPE=Release -B build && cmake --build build -j
BORINGSSL=$PWD && cd ..

cd lsquic
cmake -DCMAKE_BUILD_TYPE=Release -DLSQUIC_SHARED_LIB=OFF \
      -DLSQUIC_BIN=OFF -DLSQUIC_TESTS=OFF -DLIBSSL_DIR=$BORINGSSL -B build
cmake --build build -j
```
