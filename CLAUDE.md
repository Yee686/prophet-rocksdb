# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Prophet 是基于 RocksDB 的数据库优化项目，针对 ZNS (Zoned Namespace) SSD 设备进行优化。核心改进包括区域分配策略、预取优化等。

## 构建命令

```bash
# 构建带 ZenFS 插件的 release 版本
DISABLE_WARNING_AS_ERROR=1 ROCKSDB_PLUGINS=zenfs make -j db_bench install DEBUG_LEVEL=0

# 构建静态库
make static_lib

# 构建 debug 版本
make dbg

# 运行所有单元测试
make check

# 构建所有工具和测试（debug 模式）
make all
```

## 代码架构

```
prophet-rocksdb/
├── db/                    # 核心数据库实现
│   ├── db_impl/          # DBImpl 主类，数据库入口
│   ├── compaction/       # 压缩相关逻辑
│   └── blob/             # Blob Storage 实现
├── include/rocksdb/       # 公共 API 头文件
├── util/                  # 工具函数（缓存、日志、编码等）
├── table/                 # SST 文件格式实现
├── env/                   # 文件系统抽象层（包含 ZenFS）
├── port/                  # 平台相关代码
├── monitoring/            # 统计和监控
├── utilities/             # 高级功能（事务、TTL 等）
└── plugin/zenfs/          # ZNS 设备文件系统插件
```

## 关键文件

- `db/db_impl/db_impl.cc:1` - 数据库主实现类，处理读写请求
- `db/compaction/compaction_job.cc` - 压缩作业实现
- `db/builder.cc` - SST 文件构建
- `env/env.cc` - 环境抽象层，ZenFS 通过此集成
- `include/rocksdb/db.h` - 主数据库接口

## 运行测试

```bash
# 运行单个测试
make db_test
./db_test --gtest_filter=TestName

# 运行压缩相关测试
make db_compaction_test
./db_compaction_test

# 使用 gtest 过滤运行特定测试
./db_test --gtest_filter=*WriteTest*
```

## ZenFS ZNS 设备配置

```bash
# 1. 设置 I/O 调度器
echo deadline > /sys/class/block/<zoned_device>/queue/scheduler

# 2. 格式化设备
sudo ./plugin/zenfs/util/zenfs mkfs --zbd=<zoned_device> --aux_path=./temp --force

# 3. 使用 db_bench 测试
./db_bench --fs_uri=zenfs://dev:<zoned_device> ...
```

## 开发注意事项

- 项目依赖 C++17 编译器（GCC >= 7 或 Clang >= 5）
- 推荐安装压缩库：zlib, bzip2, lz4, snappy, zstandard
- 需要 gflags 库来运行工具程序
- ZenFS 插件需要 ZNS SSD 设备支持
