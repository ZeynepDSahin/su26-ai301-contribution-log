# Contribution [2]: [Allow GetFirmwareHash while locked
 #5040]

**Contribution Number:** [2]  
**Student:** [Zeynep Sahin]  
**Issue:** [[GitHub issue link]([https://github.com/trinodb/trino/issues/6190](https://github.com/trezor/trezor-firmware/issues/5040))]  
**Status:** [Phase II] [Complete]
**Branch Link:** (https://github.com/ZeynepDSahin/trezor-firmware) 

---

## Why I Chose This Issue

I chose this issue because it is an open, straightforward task labeled as a "good first issue," making it a perfect opportunity to get familiar with the trezor-firmware codebase. The goal is to allow the GetFirmwareHash command to run even when the device is locked, which aligns with how the legacy firmware behaves and has no negative security implications. Working on this will help me understand the core firmware architecture, specifically how command access levels are controlled and enforced, while allowing me to contribute a meaningful and practical improvement to the project.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

Reproduction branch URL:
https://github.com/ZeynepDSahin/trezor-firmware/tree/fix/5040-allow-getfirmwarehash-while-locked

Environment Setup

Repository: fork of trezor/trezor-firmware (ZeynepDSahin/trezor-firmware), branched from main.
Component: Trezor Core firmware (MicroPython), specifically the wire message dispatch and lock-state gating logic.
Build/run target: Trezor emulator (core/), which runs the same message-handling code paths as physical hardware.
Toolchain: the repo's standard Core dev environment (Nix / poetry shell); device tests are driven via trezorlib over the emulator debug link.

Relevant files:
core/src/trezor/workflow.py — the ALLOW_WHILE_LOCKED allowlist
core/src/apps/common/lock_manager.py — _pinlock_filter, which forces unlocking for messages not on the allowlist
core/src/apps/misc/get_firmware_hash.py — the GetFirmwareHash handler

Steps to Reproduce:

Run the firmware and initialize a device with a PIN set so it can be locked.
Lock the device (or use a fresh session where features.unlocked is False).
Send a GetFirmwareHash message while the device is locked.
Original behavior (the bug): because GetFirmwareHash was not listed in ALLOW_WHILE_LOCKED, _pinlock_filter wraps the handler and calls unlock_device() first, forcing a PIN prompt before the hash is returned — even though computing the firmware hash never accesses the seed or any secret.
After the fix: GetFirmwareHash is added to the allowlist, so the handler runs directly, returns the hash without any unlock prompt, and the device stays locked.
Reproduction Evidence

Branch: https://github.com/ZeynepDSahin/trezor-firmware/tree/fix/5040-allow-getfirmwarehash-while-locked
Commit: https://github.com/ZeynepDSahin/trezor-firmware/commit/c6801c969

Changes:

core/src/trezor/workflow.py — added MessageType.GetFirmwareHash to both the THP and non-THP ALLOW_WHILE_LOCKED tuples.
tests/device_tests/test_firmware_hash.py — new test test_firmware_hash_while_locked: sets a PIN, asserts unlocked is False, calls firmware.get_hash(...), asserts it returns the correct hash without a PIN prompt, and asserts the device remains locked afterward.
core/.changelog.d/5040.added — changelog fragment referencing issue #5040.


---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan


---

## Testing Strategy




### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [3] Progress



### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [[GitHub PR URL when submitted]]
**PR Description:** 
**Maintainer Feedback:**
- *Have not received feedback yet.
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
