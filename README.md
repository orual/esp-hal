# esp-hal - ESP32 Hardware Abstraction Layer (Symltech Fork)

**Fork of [esp-rs/esp-hal](https://github.com/esp-rs/esp-hal) with critical PSRAM fixes**

This fork contains essential bug fixes for ESP32-S3 PSRAM memory management that are critical for the Symltech T-Deck hardware to function properly.

## Why This Fork Exists

The upstream ESP-HAL had a critical PSRAM memory mapping bug that didn't account for unmapped pages between allocated memory regions, causing memory corruption and device crashes on ESP32-S3 with external PSRAM.

## Critical Fix: PSRAM Memory Management

### Problem
The original PSRAM initialization code had incorrect memory mapping calculation that caused:
- Memory corruption when using PSRAM
- Device crashes and instability  
- Incorrect PSRAM start address calculation

### Solution
**Commit**: `f41a06f2` - "psram fix"

Fixed PSRAM initialization in `esp-hal/src/soc/esp32s3/psram.rs`:

```rust
// Before: Incorrect PSRAM start calculation
let psram_start = PSRAM_BASE + (page_count * PAGE_SIZE);

// After: Properly account for unmapped pages
let psram_start = calculate_psram_start_with_unmapped_pages();
```

### Technical Details

The fix addresses:
- **MMU Table Iteration**: Corrected page mapping calculation to skip unmapped regions
- **Memory Layout**: Fixed PSRAM base address calculation for ESP32-S3
- **Page Management**: Proper handling of memory page boundaries and unmapped areas

## Impact on Symltech

This fix is **essential** for the T-Deck hardware because:
- T-Deck uses ESP32-S3 with 8MB external PSRAM
- PSRAM is used for display buffers and image processing
- Without this fix, the device experiences crashes and memory corruption

## Files Changed

```
esp-hal/src/soc/esp32s3/psram.rs
├── Fixed PSRAM start address calculation
├── Corrected MMU table iteration
└── Proper unmapped page handling
```

## Status

- **Bug Severity**: Critical - prevents hardware from functioning
- **Upstream Status**: Should be contributed back to esp-rs/esp-hal
- **Maintenance**: Temporary fork until upstream accepts fix

## Building

```bash
cd esp-hal
cargo build --target xtensa-esp32s3-none-elf
```

## Integration

Used by the Symltech project via:

```toml
[dependencies]
esp-hal = { git = "https://github.com/orual/esp-hal", branch = "main" }
```

## Testing

The fix can be verified by:

1. **Memory Stability Tests**: Extended PSRAM allocation/deallocation cycles
2. **Hardware Verification**: Running on actual T-Deck hardware with PSRAM operations
3. **Stress Testing**: Large buffer operations and display rendering

```rust
// Test PSRAM allocation
fn test_psram_stability() {
    for _ in 0..1000 {
        let buffer = psram_allocate(1024 * 1024); // 1MB allocation
        // Use buffer for operations
        psram_deallocate(buffer);
    }
}
```

## Future Plans

1. **Upstream Contribution**: Submit PR to esp-rs/esp-hal with proper testing
2. **Additional Testing**: Ensure fix works across all ESP32-S3 variants
3. **Remove Fork**: Once upstream accepts the fix

## Verification

To verify the fix is working:

```bash
# Build and flash Symltech firmware
cd symltech-communicator
just build neema
just flash neema

# Monitor for PSRAM-related crashes (should not occur)
just monitor
```

## License

Maintains the same license as the original esp-hal project (MIT OR Apache-2.0).