# Protobuf 21.11 (3.21.11) on loongarch64

## 前置依赖

Protobuf 21.x 起强依赖 abseil，而发布包 `protobuf-all-21.11.tar.gz` **不自带 `third_party/abseil-cpp`**，
所以必须先编好 abseil 再用 `-Dprotobuf_ABSL_PROVIDER=package`。

## 构建

    cd protobuf-21.11 && mkdir -p cmake/build && cd cmake/build
    CXXFLAGS="-O2" cmake -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_INSTALL_PREFIX=$PREFIX \
        -Dprotobuf_USE_EXTERNAL_GTEST=ON -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
        -Dprotobuf_BUILD_SHARED_LIBS=OFF -Dprotobuf_BUILD_TESTS=OFF \
        -Dprotobuf_ABSL_PROVIDER=package ../..
    make -j4 && make install

零补丁，0 error。静态库落在 `installed/lib64`。

## 验证

    $ protoc --version
    libprotoc 3.21.11

功能验证：编译一个 proto3 生成 `.pb.cc/.pb.h`，再用 `libprotobuf.a` 静态链接成可执行文件并运行，返回值 0（LoongArch ELF）。

## 注意

- `make install` 若被链式命令里的 `grep -c`（0 匹配时退出码 1）截断，记得单独补跑。
- brpc 需要 `PROTOBUF_PROTOC_EXECUTABLE` 指向这个 protoc。

## 更新（2026-11-04）

最终 BE 用本 protoc 生成 gencode，`installed/lib64/libprotobuf.a` 已链接进 `doris_be`，0 错误。
brpc 的 `PROTOBUF_PROTOC_EXECUTABLE` 指向 `$PREFIX/bin/protoc`；注意产物在 `lib64` 而非 `lib`。
