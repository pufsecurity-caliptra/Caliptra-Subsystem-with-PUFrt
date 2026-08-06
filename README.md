# Bring Up Your Own Caliptra

A pre-integrated Caliptra reference environment that enables developers to start software development on FPGA with confidence.

---

✅ Pre-integrated Caliptra + PUFrt  
✅ Official VCK190 FPGA Reference  
✅ Ready for Firmware and Software Development  

---

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture](#architecture)
3. [Project Roadmap](#project-roadmap)
4. [Build](#build)
5. [Usage](#usage)
6. [More Information & Contact](#more-information--contact)

---

## Introduction

Caliptra provides an open-source Hardware Root of Trust (RoT) RTL foundation for the industry. However, deploying Caliptra in a real SoC requires more than the core RTL — it requires additional hardware security components to be integrated, verified, and made production-ready.

A complete implementation typically requires the integration of a PUF-based UDS generation mechanism, a Noise Source for entropy, and OTP / secure non-volatile storage — all of which must be validated together with the Caliptra subsystem before silicon adoption can begin.

This project provides a **pre-integrated** Caliptra subsystem with PUFrt as a **verified reference design**, built on the official Caliptra FPGA platform (VCK190). The integration has been completed and validated on hardware, giving developers a trustworthy starting point to begin firmware development and system bring-up — without starting from scratch.

---

## Architecture

<p align="left">
  <img src="doc/architecture.png" width="500">
</p>

The Caliptra core connects to the Fuse Controller over the internal system bus. The Fuse Controller interfaces with PUFrt through the **OTI (One-Time Interface)**, enabling secure access to hardware root-of-trust primitives.

**PUFrt** provides the security foundation required for real silicon deployment:

- **PUF** — generates a unique device secret without storing it in non-volatile memory.
- **TRNG** — hardware entropy source for cryptographic operations.
- **OTP** — secure non-volatile storage for keys and configuration.

PUFrt exposes two interfaces: **APB/TCM/OTI** on the crypto side for Caliptra integration, and **APB/AHB** on the system side for host access.

Together, the Caliptra subsystem and PUFrt form a pre-integrated, verified Root of Trust foundation — ready for firmware development and system bring-up on the official Caliptra VCK190 FPGA platform.

---

## Project Roadmap

The goal of this integration effort is to demonstrate that the original Caliptra SDK continues to function correctly on a PUFrt-integrated platform, without requiring the community to adopt a divergent software stack. To validate this, the work is organized into three phases:

1. ✅ **Establish a baseline.** Build the official, unmodified Caliptra FPGA image and run the test suites provided by the Caliptra SDK, producing a set of reference results against known-good hardware.
2. ✅ **Reproduce on the integrated platform.** Build the PUFrt-integrated Caliptra FPGA image and attempt to reproduce the same test results obtained in Phase 1, using the same, unmodified Caliptra SDK.
3. 🔄 **Close the gaps.** For any test cases that do not pass on the PUFrt-integrated platform, identify the root cause the difference from the Phase 1 baseline and modify the software (caliptra-mcu-sw / caliptra-sw) only where necessary to adapt to the integrated platform — keeping such changes as minimal and local as possible.

This phased approach allows for an assessment at each step of how much the original Caliptra software needs to be changed and why, rather than presenting the integration as a signal, opaque branch.

### What Was Changed

Two targeted revisions to the Caliptra software stack were required to accommodate the PUFrt-integrated platform, both stemming from behavioral differences between the original FPGA reference model and a real fuse interface implementation:

1. **Lifecycle state fuse write with ECC.** The Caliptra specification defines lifecycle state fuses using a 16-bit data field paired with 6 bits of ECC protection. The data stored in fuses, including lifecycle state fuses will be written through a backdoor interface on the Caliptra FPGA platform. On the PUFrt-integrated platform, the OTI interface handles lifecycle state fuses as defined in the Caliptra specification. However, the backdoor interface requires the lifecycle state ECC bits to be provided manually. To accommodate this, the fuse write function was revised to compute and supply the ECC bits when writing lifecycle state fuses through the backdoor interface, ensuring correct behavior on the PUFrt-integrated platform.
2. **Extended OpenOCD TCL-ready timeout.** With a real PUFrt fuse interface in place, the JTAG interface may require additional time to become ready to accept a TCL connection compared to the original FPGA model. Without adjustment, this additional latency could cause JTAG-dependent test cases to report false-positive timeout failures. The OpenOCD TCL-ready timeout was therefore extended to account for this real-hardware initialization delay, ensuring JTAG-based tests reflect actual pass/fail behavior rather than timing artifacts.

### Test Case Status

| Test Case | Baseline (Original FPGA) | Reproduced (PUFrt-Integrated) | Remaining Gap / Status |
|---|---|---|---|
| `otp_provision::tests::test_decode_raw_state` | Pass | Pass |  |
| `otp_provision::tests::test_decode_roundtrip_all_counts` | Pass | Pass |  |
| `otp_provision::tests::test_decode_roundtrip_all_states` | Pass | Pass |  |
| `otp_provision::tests::test_lifecycle_manufacturing` | Pass | Pass |  |
| `otp_provision::tests::test_lifecycle_unlocked1` | Pass | Pass |  |
| `otp_provision::tests::test_otp_generate_lifecycle_tokens_mem` | Pass | Pass |  |
| `otp_provision::tests::test_otp_unscramble_token` | Pass | Pass |  |
| `tests::test_mailbox_execute` | Pass | Pass |  |
| `vmem::tests::test_pqc_key_type_doc_example` | Pass | Pass |  |
| `vmem::tests::test_read_write_vmem` | Pass | Pass |  |
| `vmem::tests::test_vendor_pk_hash_doc_example` | Pass | Pass |  |
| `jtag::test_bare_metal_jtag::test::test_bare_metal_jtag_sideload` | Pass | Pass |  |
| `jtag::test_jtag_taps::test::test_lcc_ta` | Pass | Pass |  |
| `jtag::test_lc_transitions::test::test_lc_walkthrough` | Pass | Pass |  |
| `jtag::test_lc_transitions::test::test_prod_rma_unlock` | Pass | Pass |  |
| `jtag::test_lc_transitions::test::test_raw_unlock` | Pass | Pass |  |
| `jtag::test_manuf_debug_unlock::test::test_manuf_debug_unlock` | Pass | Pass |  |
| `jtag::test_prod_debug_unlock::test::test_prod_debug_unlock` | Pass | Pass |  |
| `jtag::test_uds::test::test_uds` | Pass | Pass |  |
| `rom::test_bootfsm_timeout::test::test_bootfsm_timeout` | Pass | Pass |  |
| `rom::test_i3c_services::test::test_i3c_services_dot_error_then_continue` | Pass | Pass |  |
| `rom::test_i3c_services::test::test_i3c_services_dot_override_full_flow` | Pass | **Fail** | Fuse reset function is needed |
| `rom::test_i3c_services::test::test_i3c_services_dot_override_without_challenge` | Pass | Pass |  |
| `rom::test_i3c_services::test::test_i3c_services_dot_recovery_invalid_payload` | Pass | Pass |  |
| `rom::test_i3c_services::test::test_i3c_services_dot_status` | Pass | Pass |  |
| `rom::test_i3c_services::test::test_i3c_services_error_mid_packet_then_recover` | Pass | Pass |  |
| `rom::test_i3c_services::test::test_i3c_services_ping` | Pass | Pass |  |
| `rom::test_i3c_services::test::test_i3c_services_unknown_cmd` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_dev_to_prod` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_dev_to_rma` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_dev_to_scrap` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_invalid_transition_error` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_prod_end_to_scrap` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_prod_to_prod_end` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_prod_to_rma` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_prod_to_scrap` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_raw_to_scrap` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_raw_to_test_unlocked0` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_rma_to_scrap` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_test_locked0_to_scrap` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_test_locked0_to_test_unlocked1` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_test_unlocked0_to_scrap` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_test_unlocked0_to_test_locked0` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_test_unlocked7_to_dev` | Pass | Pass |  |
| `rom::test_lc_ctrl::test::test_lc_wrong_token_error` | Pass | Pass |  |
| `rom::test_otp_blank_check::test::test_otp_blank_check` | Pass | Pass |  |
| `rom::test_otp_scramble_check::test::test_otp_scramble_check` | Pass | Pass |  |
| `rom::test_rom_hooks::test::test_rom_hooks_fire_in_order` | Pass | Pass |  |
| `rom::test_sw_digest_lock::test::test_sw_digest_lock` | Pass | Pass |  |
| `test_raw_lifecycle_boot::test::test_raw_lifecycle_boot` | Pass | Pass |  |

### Known Limitation: Fuse Reset for Test Iteration

One function is currently missing from the PUFrt-integrated FPGA environment: a **fuse reset function** capable of resetting select fuse values from 1 back to 0. This is not a deviation from real-world fuse/OTP behavior — fuses are one-time-programmable by nature, and once a bit is written from 0 to 1, it cannot be reverted. Rather, the original Caliptra FPGA platform provides a backdoor interface that bypasses this restriction purely to support iterative testing, allowing the same FPGA image to be reused across multiple test runs with differing fuse configurations.

Without an equivalent reset mechanism on the PUFrt-integrated platform, one test case currently fails, and the FPGA environment cannot execute multiple test cases in sequence whenever their fuse value requirements conflict. To restore equivalent test-iteration support, we plan to add a corresponding backdoor interface for resetting fuse values on the PUFrt-integrated platform — mirroring the testing convenience of the original Caliptra FPGA environment without altering PUFrt's real-world, one-time-programmable fuse behavior in production use.

---

## Build

Because the PUFrt-integrated platform reuses the standard `caliptra-mcu-sw` toolchain and build flow without modification (aside from the software changes noted in the Project Roadmap above), the general build process follows the same steps as the original Caliptra project.

- **`caliptra-mcu-sw` — Building**
  General build instructions, toolchain setup (via `rustup`) and full clone of the repository:
  https://github.com/pufsecurity-caliptra/caliptra-mcu-sw/blob/pufrt/README.md#building

- **`caliptra-mcu-sw` — FPGA Build Instructions**
  FPGA-specific build and bring-up instructions:
  https://chipsalliance.github.io/caliptra-mcu-sw/fpga.html

[**TODO**]
- Specify https://github.com/pufsecurity-caliptra/caliptra-sw in `caliptra-mcu-sw`
- Branch `pufrt` from `main-2.1` in `caliptra-mcu-sw` and set it as default

---

## Usage

The PUFrt-integrated Caliptra FPGA image will be provided, developers can download and exercise the platform following the same workflow used for the original Caliptra FPGA reference:

1. **Set up the FPGA SD card and boot image.**
   Follow the `caliptra-mcu-sw` FPGA setup guide to prepare the SD card and load the FPGA boot image:
   https://github.com/pufsecurity-caliptra/caliptra-mcu-sw/blob/pufrt/hw/fpga/README.md

2. **Run tests or develop software against the platform.**
   Once the board is up, follow the test workflow described in the published `caliptra-mcu-sw` specification to run the reference test suite or begin developing new software against the platform:
   https://chipsalliance.github.io/caliptra-mcu-sw/fpga.html#test-workflow

This pre-integrated Caliptra subsystem with PUFrt provides the same environment as the original Caliptra subsystem for software and firmware developers. Any platform-specific behavior developers should be aware of — such as the fuse ECC handling and JTAG timeout adjustments — is documented in the Project Roadmap section. 

[**TODO**]
- Update the link to the FPGA image file boot1900.bin

---

## More Information & Contact

### Documentation

- Release Notes
- Integration Guide
- API Reference

### Hardware Integration Inquiry

For custom hardware security integration (Caliptra + PUFrt RTL integration, hardware security customization), please contact us at:

📧 [contact email placeholder]

---

## License

This repository is provided for evaluation and development purposes only. 
For commercial deployment, production use, or integration into commercial products, please contact PUFsecurity for licensing terms.
More: [License](LICENSE)
