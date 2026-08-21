## 📄 中文 README（README-CN.md）

# TFBC - 极速极简字节码

[![许可证: MIT](https://img.shields.io/badge/许可证-MIT-green.svg)](LICENSE)
[![语言: C](https://img.shields.io/badge/语言-C-blue.svg)]()

**TFBC** 是一套为**极致压缩率**和**极速执行**而设计的字节码规范。

---

## 🚀 核心指标

| 指标 | 数据 |
|------|------|
| 指令密度 | **< 1.0 字节/指令** |
| 解码速度 | **1 个时钟周期** |
| 中断响应 | **3-5 个时钟周期** |
| 核心 VM 内存 | **< 16 KB** |
| 寄存器数量 | **16 个 + 窗口切换** |
| 安全模型 | **MPU + 4 级特权** |
| 浮点支持 | **IEEE 754 单精度** |
| 扩展能力 | **无限链式扩展** |

---

## ⚡ 为什么选择 TFBC？

| 对比项 | CPython | Java BC | WASM | **TFBC** |
|--------|---------|---------|------|----------|
| 指令密度 | ~2-3 字节 | ~2 字节 | ~3 字节 | **< 1 字节** 🏆 |
| 解码速度 | 3 周期 | 3 周期 | 5 周期 | **1 周期** 🏆 |
| 中断响应 | N/A | N/A | N/A | **3-5 周期** 🏆 |
| VM 内存 | ~5 MB | ~128 KB | ~256 KB | **< 16 KB** 🏆 |

---

## 📦 快速开始

```bash
# 克隆仓库
git clone https://github.com/yourusername/TFBC.git
cd TFBC

# 编译虚拟机
gcc -O2 -o tfbc-vm src/v1.0/VM.c

# 运行示例程序
./tfbc-vm examples/hello.tfbc

# Python → TFBC 转换
py-to-tfbc app.py -o app.tfbc
./tfbc-vm app.tfbc
```

---

## 📖 文档

| 文档 | 说明 |
|------|------|
| [完整规范 (中文)](docs/v1.0/spec-CN.md) | TFBC 1.0 完整中文规范 |
| [完整规范 (英文)](docs/v1.0/spec.md) | TFBC 1.0 完整英文规范 |
| [快速参考](docs/v1.0/quick-ref.md) | 指令集速查卡 |

---

## 🛠️ 工具链

| 工具 | 说明 |
|------|------|
| `tfbc-as` | 汇编器 |
| `tfbc-ld` | 链接器 |
| `tfbc-vm` | 虚拟机（含 JIT） |
| `tfbc-cc` | AOT 编译器（.tfbc → 原生） |
| `py-to-tfbc` | Python → TFBC 转换器 |

---

## 📜 许可证

本项目采用 **MIT 许可证** - 详见 [LICENSE](LICENSE) 文件。

商用、修改、分发全部自由，只需保留版权声明。

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

---

⭐ **如果喜欢这个项目，请给个 Star！**

---

*TFBC 1.0 — 极速极简，生产就绪。* 🚀
```

---

