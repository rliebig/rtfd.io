---
title: Architectural Registers
---

Architectural registers may be accessed through the `ql.arch.regs` object as described below. For more info please refer to [qiling/arch/register.py](https://github.com/qilingframework/qiling/blob/master/qiling/arch/register.py)

Registers may be denoted either by their architectural names (e.g. `'eax'` in x86, `'r0'` in ARM, etc.) or by their corresponding Unicorn enumeration value (e.g. `UC_X86_REG_EAX`)

### Read
There are two equivalent methods to read architectural registers: using the `read` method, or access them as properties. Though both may be use interchangebly, it is recommended to maintain consistency and pick one for the course of the emulation program.

#### Using the read method
The read method has a single argument to indicate the register to be read. The argument may be either the register architectural name (e.g. `'eax'`) or Unicorn enumeartion constant (e.g. `UC_X86_REG_EAX`).

The following are identical in functionality:
```python
value = ql.arch.regs.read('eax')           # reading eax using register architectural name
value = ql.arch.regs.read(UC_X86_REG_EAX)  # reading eax using Unicorn enumeration constant
```

#### Accessing register as a property
```python
value = ql.arch.regs.eax
```

### Write
There are two equivalent methods to write architectural registers: using the `write` method, or access them as properties. Though both may be use interchangebly, it is recommended to maintain consistency and pick one for the course of the emulation program.

#### Using the write method
The write method has two arguments to indicate the register and value to write. The argument may be either the register architectural name (e.g. `'ecx'`) or Unicorn enumeartion constant (e.g. `UC_X86_REG_ECX`).

The following are identical in functionality:
```python
value = 0xdeadface

ql.arch.regs.write('ecx', value)           # modifying ecx using register architectural name
ql.arch.regs.write(UC_X86_REG_ECX, value)  # modifying ecx using Unicorn enumeration constant
```

#### Accessing register as a property
```python
ql.arch.regs.ecx = 0xdeadface
```

This access method has an advantage as it enables more readable code when manipulating a register value, e.g.:
```python
ql.arch.regs.eflags |= (0b1 << 6)  # set the zero flag
```

### Generic aliases
To allow code to be generic across multiple architectures, a few common registers may be accessed by their generic aliases:

```python
pc = ql.arch.regs.arch_pc  # read the architectural program counter ('eip' on x86, 'pc' on ARM, etc.)
sp = ql.arch.regs.arch_sp  # read the architectural stack pointer ('esp' on x86, 'sp' on ARM, etc.)
```

The generic aliases may be used to modify the content of their respective registers in a similar manner:

```python
ql.arch.regs.arch_pc = 0x00400000  # setting the architectural program counter
ql.arch.regs.arch_sp = 0x7ffff000  # setting the architectural stack pointer
```

### Architecture-specific resources
Some architectures may have their own specific resources which are available in slightly different ways.

#### Intel Model Specific Registers (MSR)
Intel MSR are accessed through `ql.arch.msr`, which has its own `read` and `write` methods. For example:

```python
IA32_BIOS_SIGN_ID = 0x8b

# read bios signature value
bios_sig = ql.arch.msr.read(IA32_BIOS_SIGN_ID)
```

```python
IA32_APIC_BASE_MSR = 0x1b
apic_base = 0xfee00000

# set apic base while setting BSP and APIC global enable
ql.arch.msr.write(IA32_APIC_BASE_MSR, apic_base | (0b1 << 8) | (0b1 << 11))
```

#### AArch / AArch64 Coprocessors
ARM coprocessors are accessed through `ql.arch.cpr` and `ql.arch.cpr64` respectively, with their own `read` and `write` methods. For example:

```python
from qiling.arch.arm_const import CPACR

# set full access to cp10 and cp11
cpacr = ql.arch.cpr.read(*CPACR)
ql.arch.cpr.write(*CPACR, cpacr | (0b11 << 20) | (0b11 << 22))
```
