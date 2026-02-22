---
description: "Repository-wide custom instructions for Copilot - Harvey OS"
applyTo: "**/*"
---

# Harvey OS - Plan 9 Derivative Operating System

Harvey OS is a distributed operating system based on Plan 9 from Bell Labs. It's designed for modern computing environments while maintaining the elegant simplicity and distributed nature of Plan 9.

## Project Overview

- **Type**: Distributed operating system kernel and userspace
- **Base**: Plan 9 4th Edition from Bell Labs
- **Language**: Primarily C, with some Go components
- **Architecture**: Multi-architecture support (x86, ARM, MIPS, etc.)
- **Focus**: Distributed computing, network transparency, and modern hardware support

## Key Characteristics

- Everything is a file (including network connections, processes)
- 9P protocol for distributed file systems
- Clean, minimal design philosophy
- QEMU-based development and testing environment

## Build System & Dependencies

**CRITICAL TIMING WARNING**: Harvey OS builds are EXTREMELY long-running (90-95 minutes). **NEVER CANCEL BUILD PROCESSES** that appear stuck.

### Required Dependencies
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y qemu-system-i386 qemu-utils expect rc go-dep

# Go tools
go get harvey-os.org/cmd/ufs

# Plan 9 ISO (automatically downloaded during build)
```

### Build Process Overview
1. **Dependency Download** (5-10 minutes)
2. **Plan 9 Boot** (5-10 minutes) 
3. **Source Mount via 9P** (2-3 minutes)
4. **Harvey Build** (60-70 minutes)
   - **GhostScript compilation**: 25+ minutes (appears frozen - THIS IS NORMAL)
   - Kernel compilation: 15-20 minutes
   - Userspace compilation: 20-25 minutes
5. **ISO Creation** (5-10 minutes)

### CRITICAL Build Timeouts
- **Total build time**: 90-95 minutes minimum
- **GhostScript phase**: 25+ minutes (appears completely stuck - DO NOT CANCEL)
- **Initial wait for any build command**: 120+ seconds minimum
- **Long compilation phases**: Use 300+ second delays when checking progress

## Development Environment

### Architecture Directories
```
386/     - x86 32-bit architecture files
amd64/   - x86 64-bit architecture files  
arm/     - ARM architecture files
mips/    - MIPS architecture files
power/   - PowerPC architecture files
riscv/   - RISC-V architecture files
sparc/   - SPARC architecture files
```

### Core System Directories
```
sys/     - Core system files, drivers, kernel
lib/     - System libraries and utilities
usr/     - User space applications
rc/      - RC shell and scripts
mail/    - Mail system components
acme/    - Acme text editor
```

### Key Files
- `BUILD_ON_UNIX` - Unix build instructions
- `cfg/` - System configuration files
- `build/` - Build scripts and tooling
- `dist/` - Distribution and packaging

## Coding Standards

### C Code Style
- Follow Plan 9 C style conventions
- Use tabs for indentation (width 8)
- K&R brace style
- Clear, descriptive variable names
- Minimal use of preprocessor macros

### Example C Code Pattern:
```c
void
function(int arg)
{
	char *buf;
	
	buf = malloc(256);
	if(buf == nil){
		fprint(2, "malloc failed\n");
		exits("malloc");
	}
	/* ... */
	free(buf);
}
```

### Go Code Style
- Standard Go formatting (gofmt)
- Follow Go naming conventions
- Use Go modules for dependency management

### Assembly Code
- Architecture-specific assembly in respective arch directories
- Follow Plan 9 assembler syntax
- Clear commenting for hardware-specific operations

## File System Structure

### Everything is a File
- `/proc` - Process information
- `/net` - Network interfaces
- `/dev` - Device files
- `/mnt` - Mount points for remote file systems

### 9P Protocol
- Network-transparent file access
- Used for distributed computing
- All system resources accessible via file operations

## Build Commands

### Basic Build
```bash
cd build
bash command.bash  # Basic build command
```

### Development Build with Monitoring
```bash
cd build
# Start build (will take 90+ minutes)
timeout 7200 bash command.bash || echo "Build completed or timed out"
```

### Testing
```bash
# Boot test in QEMU
qemu-system-i386 -cdrom harvey.iso
```

## Common Development Workflows

### Adding New System Calls
1. Define in `sys/src/9/port/` or architecture-specific directory
2. Add to system call table
3. Update user-space headers in `sys/include/`
4. Test with both kernel and user-space components

### Device Driver Development
1. Create driver in `sys/src/9/port/` or `sys/src/9/pc/`
2. Follow existing driver patterns
3. Register with device manager
4. Test with QEMU hardware emulation

### User-Space Applications
1. Create in appropriate `usr/` subdirectory
2. Use Plan 9 APIs and conventions
3. Link against system libraries
4. Test in Harvey environment

## Troubleshooting Guide

### Build Issues

**Problem**: Build appears frozen for 20+ minutes
**Solution**: This is NORMAL during GhostScript compilation. Wait 90+ minutes total.

**Problem**: "Plan 9 ISO not found"
**Solution**: Ensure internet connection for automatic download, or manually place `plan9.iso` in build directory.

**Problem**: QEMU acceleration warnings
**Solution**: In sandboxed environments, KVM may not be available. Build will work but be slower.

**Problem**: Go compilation errors
**Solution**: Ensure Go 1.16+ is installed and `GOPATH` is set correctly.

### Runtime Issues

**Problem**: Boot hangs at Plan 9 prompt
**Solution**: Check ISO integrity and QEMU configuration.

**Problem**: Network not working in QEMU
**Solution**: Verify QEMU network configuration and 9P mount points.

## Testing and Validation

### Build Success Indicators
- "Harvey ISO created successfully" message
- `harvey.iso` file present in build directory
- No fatal errors in build log

### Boot Testing
```bash
# Test boot sequence
qemu-system-i386 -cdrom harvey.iso -nographic
```

### System Validation
- Check core Plan 9 commands work (`ls`, `cat`, `rc`)
- Verify 9P mounts are accessible
- Test network connectivity if configured

## Security Considerations

### Memory Safety
- Harvey inherits Plan 9's memory model
- Be cautious with pointer arithmetic
- Use appropriate bounds checking

### Network Security
- 9P protocol has authentication mechanisms
- Implement proper access controls for network resources
- Validate file system permissions

## Distributed OS Development Best Practices

### Network Transparency
- Design components to work across network boundaries
- Use 9P for all file system operations
- Consider network latency in performance critical code

### Resource Management
- Everything-is-a-file paradigm
- Proper cleanup of file descriptors
- Efficient handling of distributed resources

### Cross-Architecture Support
- Write portable C code
- Use architecture-specific code sparingly
- Test across multiple target architectures

## Environment Limitations

### Sandboxed Development
- KVM acceleration may not be available
- Builds will be slower but functional
- Network access may be limited for QEMU

### Long Build Times
- **Always allow 90+ minutes for full builds**
- Use incremental build techniques when possible
- Plan development time accordingly

## Key Resources

- [Harvey OS Official Site](https://harvey-os.org/)
- [Plan 9 Documentation](http://plan9.bell-labs.com/plan9/)
- [9P Protocol Specification](http://plan9.bell-labs.com/magic/man2html/5/intro)
- [Build System Documentation](BUILD_ON_UNIX)

## Important Notes for AI Assistants

1. **NEVER CANCEL LONG-RUNNING BUILDS** - 90+ minute build times are normal
2. **GhostScript compilation appears frozen** - Wait 25+ minutes, this is expected
3. **Use long timeouts** - Minimum 120s initial wait, 300s+ for compilation phases
4. **Harvey is NOT Linux** - Different syscalls, file system, and development model
5. **Test in QEMU** - Primary development and testing environment
6. **Respect Plan 9 conventions** - Clean, minimal design philosophy

## Validation Scenarios

### New Feature Development
1. Understand Plan 9 design principles
2. Implement following existing patterns
3. Test in QEMU environment
4. Validate cross-architecture compatibility
5. Document network transparency aspects

### Bug Fixes
1. Reproduce in QEMU environment
2. Follow Plan 9 debugging practices
3. Test fix across affected architectures
4. Verify no regression in core functionality
5. Validate distributed system behavior

### Performance Optimization
1. Profile in QEMU environment
2. Consider distributed system implications
3. Maintain network transparency
4. Test across different architectures
5. Benchmark against Plan 9 reference implementation