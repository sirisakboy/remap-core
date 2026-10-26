# Deep review — ECU Tuner Core Pack

**Review date:** 2026-10-26  
**Disposition:** research/reference bundle only; **not ready for live ECU write/flash or public/commercial release as a unified product**.

## Executive summary

The pack has a sensible separation between transport, protocol references and exact ECU profiles. The two included XDFs are structurally readable, contain no `XDFPATCH`/`XDFFLAG` blocks or password fields, and no stock firmware file was found. The CAN/IP examples fail closed for transmit by default. The package is nevertheless **not a validated motorcycle tuning tool**: no included map profile has an independently verified ECU/HW/SW match, verified checksum vectors or vehicle test. Several optional references expose low-level send/programming operations, one XDF parser evaluates formulas from input, licensing is not cleared at pack level, and selected source snapshots have incomplete build/test collateral.

**Recommended use today:** offline research, source review and hardware-free transport tests only. Do not connect this bundle to a live ECU for writes, programming-voltage operations, periodic sends or guessed diagnostic requests.

## Review scope and method

Reviewed the current pack structure and manifest/ZIP consistency; the two engine XDFs and profile/coverage/picker records; local CAN/IP/BLE sidecars; the J2534 references; the XDF parser and KWP reference; selected source/license notes; local Markdown links and build references. Ran the included hardware-free tests and exercised a few pure parser functions with benign synthetic inputs. No ECU, vehicle, J2534 cable, BLE device or physical CAN adapter was connected. This is a static engineering review—not a formal security audit, legal opinion, exhaustive line-by-line review of every vendored upstream file, or CVE/SCA scan.

## Verified checks

- The two included XDFs parse as XML: CBR1000RR has 39 tables, one external checksum block and no flags/patches; CRF110F has 11 tables and no checksum/flags/patches. Neither includes `modifypassword` or `openpassword` fields. The CRF110F region/start-offset discrepancy and CBR checksum-plugin limitation remain unresolved as documented.
- No `.bin`, `.rom`, `.hex`, `.srec`, `.mot`, `.img`, `.eep` or `.flash` file was found in the pack. The source archives were also checked for firmware-like paths.
- The manifest matched all files and sizes at review time; the ZIP passed `testzip`. The included tests passed **6/6**: virtual CAN plus localhost TCP/UDP only.
- The CAN JSON Schema was checked against representative requests. Three requests pass schema validation but are rejected by the runtime implementation (details below).
- A local-link scan found **25 relative links to omitted files** across five upstream/reference documents; most are deliberate omissions from selected source snapshots, but they make some embedded upstream instructions incomplete.

## Findings

### DR-01 — HIGH, conditional: XDF parser evaluates formulas from input and has correctness defects

**Evidence:** `references/xdf_parser/xdf_merge.py:54–60` uses Python `eval()` on formula text. Setting `__builtins__` to `None` is not a robust sandbox. If this reference parser is ever run on untrusted XDF formulas, treat it as unsafe until expression parsing is replaced with a strict AST allowlist. No exploit was executed in this review; the parser is currently marked reference-only and is not used by the included profiles.

The same file also has concrete output defects:

- `find_size()` at lines 186–194 multiplies `Rowcount * Rowcount`; a synthetic 2×3 axis returned 4 instead of 6.
- `find_result_table()` at lines 220–232 uses `==` where it needs assignment for the single-equation case; a synthetic input returned `result: None`.
- Formula callers uppercase equations before evaluation (for example lines 215–216), while the allowed `math` names are case-sensitive; a benign `sin(X)` example through `find_result()` failed. The parser should not be used as an authoritative map decoder.

**Action:** keep it isolated/reference-only; before any reuse, obtain a clear license, replace `eval()` with an allowlisted parser, fix the calculation bugs, and add tests for scalar/axis/table expressions and malicious input.

### DR-02 — HIGH, safety: transport/J2534 interfaces can transmit raw data when explicitly enabled

**Evidence:**

- `adapters/can_jsonl_sidecar/can_jsonl_sidecar.py:183–233` can send arbitrary valid CAN IDs/payloads after process launch with `--allow-tx`. It has bounds checks, but no ECU-ID allowlist or rate limit. Default transmit is off.
- `transports/ip_socket/raw_socket_transport.py` gates TCP/UDP send behind `tx_enabled=True`, but after opt-in it sends caller-supplied bytes to a caller-supplied host/port; it does not verify the adapter or protocol.
- The included J2534 references expose `PassThruWriteMsgs`, periodic messages and programming voltage. The OpenPort API note describes 5–20 V/ground operations and states periodic messages may continue in the cable after the client process exits until a later reset. These upstream claims were not independently tested here.
- `ble-serial` defaults to `--permit rw`; its optional TCP server has no authentication. The default bind is loopback (`127.0.0.1`), but the CLI permits changing the host.

These are capabilities of low-level adapters, not a profile-driven safe write workflow. Opening hardware can still initialize a device even when no CAN frame is sent.

**Action:** leave transmit disabled; use only isolated, authorized benches until a product-level capability gate, explicit target/ID allowlists, rate/time limits, audit logging and emergency-stop behavior are designed and tested. Keep the BLE-serial TCP bridge on loopback; do not bind it to a LAN interface.

### DR-03 — HIGH, readiness: no included ECU profile is verified for operational use

**Evidence:** the CBR XDF has no verified checksum implementation/vectors and its exact ECU/software applicability is not established. The CRF110F XDF has an unresolved region/start-offset mismatch. Neither profile has a known-good binary comparison, bench test or vehicle test. The two diagnostic references also remain unverified. `PROFILE_PICKER_TH.csv` correctly marks them as candidates/references, not supported write targets.

**Action:** keep all map definitions offline/read-only. Before a future profile can support editing, require exact ECU part/HW/SW/calibration identity, address-origin/length evidence, known-good checksum vectors and controlled bench validation; vehicle testing is a separate gate.

### DR-04 — HIGH for redistribution: the bundle has no pack-level license and some included code has unclear terms

**Evidence:** there is no top-level `LICENSE` covering locally authored sidecars/contracts/docs. `references/xdf_parser/` has no tracked upstream LICENSE/COPYING in the snapshot. `references/kwp2000/` contains an MIT-style source header while its repository-level LICENSE is GPL-3.0; the relationship is unresolved. The rest of the bundle also carries distinct GPL, AGPL, LGPL, MIT and Apache/BSD terms.

**Action:** do not treat the ZIP as one uniformly licensed SDK or redistribute it commercially without review. Decide a license for local code, retain per-component notices, and remove or clear the XDF parser/KWP items before claiming a distribution right. “Reference-only” is not itself a redistribution license.

### DR-05 — MEDIUM: OpenPort J2534 snapshot is not self-contained for the documented test workflow

**Evidence:** `adapters/openport2_j2534/Makefile` declares tests under `tests/unit`, `tests/mock`, fuzz sources under `tests/fuzz`, simulator files under `tests/sim`, and examples/tools that are absent from this curated directory. The copied README/PROTOCOL/API/TESTING documents link to additional omitted files. The full `make check-all` workflow therefore cannot run from this snapshot even after installing libusb. The ordinary build was not verified; the environment also lacked libusb development files.

**Action:** either include the specific test/build collateral needed under the selected-source rule, or clearly label this as a non-build-verified code/documentation reference and replace instructions that imply the full upstream checkout is present. Distinguish upstream hardware claims from tests actually run on this pack.

### DR-06 — MEDIUM: CAN JSON Schema and runtime accept different request sets

**Evidence:** `core_contracts/CAN_SIDECAR_SCHEMA.json` accepts all three of these, but runtime checks reject them:

1. `arbitration_id: "ABC"` (schema permits unprefixed hex; runtime parses strings with `int(value, 0)`).
2. `arbitration_id: 2048` with `is_extended_id: false` (schema does not condition the ID maximum on the flag; runtime enforces the 11-bit limit).
3. A 9-byte payload with `is_fd: false` (schema allows up to 64 items without a CAN-FD condition; runtime limits classic CAN to 8 bytes).

This is a client-generation/validation contract defect, not an observed unauthorized send; the runtime rejects these cases.

**Action:** make the schema conditional on `is_extended_id` and `is_fd`, align accepted hex-string syntax, and add tests that validate schema and implementation against the same request corpus.

### DR-07 — MEDIUM: BLE bridge network exposure is easy to misconfigure

**Evidence:** `dependencies/ble-serial-3.0.0/ble_serial/cli.py:37–57` sets permission mode to `rw` by default and can enable a TCP listener. `ble_serial/ports/tcp_socket.py:24–37` accepts a peer without authentication. Loopback is the CLI default, but binding to a non-loopback interface can expose the BLE data path to other hosts.

**Action:** do not expose the TCP bridge beyond loopback; set a narrower permission mode when appropriate; do not use it with an ECU until its GATT UUIDs, permissions and transport semantics are independently verified. Do not disable Secure Boot to install its Windows virtual-COM driver.

### DR-08 — MEDIUM: BLE sidecar is uncompiled and dependency resolution is not fully pinned

**Evidence:** `adapters/btleplug_scan/` has not been compiled or tested with a BLE adapter. Its manifest uses broad `tokio = "1"` and `serde_json = "1"` requirements and has no sidecar `Cargo.lock`; the local btleplug crate is pinned, but the wrapper's full dependency graph is not.

**Action:** add a lockfile and target-platform CI builds/tests before treating this as a client adapter. Keep its current scan/list-only capability boundary.

### DR-09 — MEDIUM, scope: OpenPort protocol documentation includes adapter firmware-update material

**Evidence:** `adapters/openport2_j2534/docs/PROTOCOL.md` describes an OpenPort cable bootloader/update handshake and a cable-side update-seed exchange. No firmware image or ECU immobilizer seed-key algorithm was found in that section, and the driver documentation says it does not issue the bootloader transition. This is separate from the motorcycle ECU profiles, but it exceeds a narrowly defined transport-only bundle.

**Action:** decide whether this cable-firmware subsection belongs in the client pack. If the rule is “no seed/update material of any kind,” remove or replace that subsection with a short non-operational note; do not confuse it with ECU immobilizer data.

### DR-10 — LOW: selected snapshots contain broken local links and inherited instructions

**Evidence:** 25 local relative links point to omitted OpenPort/KWP files. These mostly reflect repository filtering rather than missing core source, but they can mislead a user who tries to follow upstream build or test directions.

**Action:** keep upstream documents clearly marked as unmodified, add local snapshot-status notes, or rewrite the entry-point guide to point only to files actually shipped.

## Recommended release gates

1. **Safety:** research/read-only label remains; no profile-backed write/flash; keep raw transmit disabled outside an isolated bench.
2. **Licensing:** resolve the pack-level/local-code license, XDF parser license and KWP license ambiguity before external redistribution.
3. **Parser:** remove unsafe `eval()` and fix/test the demonstrated XDF parser defects, or remove it from the distributable pack.
4. **Contracts:** reconcile CAN schema with runtime and validate both from shared test vectors.
5. **Reproducibility:** either restore OpenPort's test collateral or document the curated subset accurately; pin the BLE sidecar dependency graph and compile/test on supported platforms.
6. **Profiles:** obtain exact ECU/HW/SW match evidence and known-good checksum vectors; validate each profile on a controlled bench before any vehicle trial.

## Bottom line

**Suitable today:** curated offline research and the included virtual/localhost examples, subject to component licenses.  
**Not suitable today:** a production ECU tuning client, a vehicle-tested protocol stack, checksum-approved map editing, or any write/flash workflow.  
**Runtime code changes in this review:** none; findings and recommended gates only.
