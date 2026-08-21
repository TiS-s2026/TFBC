# TFBC - Tiny and Fast Bytecode

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Language: C](https://img.shields.io/badge/Language-C-blue.svg)]()

**TFBC** is a bytecode specification designed for **extreme compression** and **ultra-fast execution**.

---

## 🚀 Key Metrics

| Metric | Value |
|--------|-------|
| Instruction Density | **< 1.0 byte/instruction** |
| Decode Speed | **1 clock cycle** |
| Interrupt Response | **3-5 clock cycles** |
| Core VM Memory | **< 16 KB** |
| Registers | **16 + window switching** |
| Security | **MPU + 4 privilege levels** |
| Floating Point | **IEEE 754 single-precision** |
| Extensibility | **Infinite chained extensions** |

---

## ⚡ Why TFBC?

| Comparison | CPython | Java BC | WASM | **TFBC** |
|------------|---------|---------|------|----------|
| Instruction Density | ~2-3 bytes | ~2 bytes | ~3 bytes | **< 1 byte** 🏆 |
| Decode Speed | 3 cycles | 3 cycles | 5 cycles | **1 cycle** 🏆 |
| Interrupt Response | N/A | N/A | N/A | **3-5 cycles** 🏆 |
| VM Memory | ~5 MB | ~128 KB | ~256 KB | **< 16 KB** 🏆 |

---

## 📦 Quick Start

```bash
# Clone the repository
git clone https://github.com/yourusername/TFBC.git
cd TFBC

# Build the VM
gcc -O2 -o tfbc-vm src/v1.0/VM.c

# Run an example program
./tfbc-vm examples/hello.tfbc

# Python → TFBC conversion
py-to-tfbc app.py -o app.tfbc
./tfbc-vm app.tfbc
```

---

## 📖 Documentation

| Document | Description |
|----------|-------------|
| [Full Specification (English)](docs/v1.0/spec.md) | Complete TFBC 1.0 specification |
| [Full Specification (Chinese)](docs/v1.0/spec-CN.md) | Complete TFBC 1.0 specification (CN) |
| [Quick Reference](docs/v1.0/quick-ref.md) | Instruction set cheat sheet |

---

## 🛠️ Toolchain

| Tool | Description |
|------|-------------|
| `tfbc-as` | Assembler |
| `tfbc-ld` | Linker |
| `tfbc-vm` | Virtual Machine (with JIT) |
| `tfbc-cc` | AOT Compiler (.tfbc → native) |
| `py-to-tfbc` | Python → TFBC converter |

---

## 📜 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

Commercial use, modification, and distribution are all permitted, with the only requirement being to retain the copyright notice.

---

## 🤝 Contributing

Issues and Pull Requests are welcome!

---

⭐ **If you like this project, please give it a star!**

---

*TFBC 1.0 — Tiny, Fast, and Production Ready.* 🚀

---

