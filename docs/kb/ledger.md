# Pattern ledger

One row per cross-repository pattern ([shared/patterns](shared/patterns)): has this repository been checked for it,
at which commit, and what was found. `kb.py sync` adds an `unchecked` row whenever a new pattern arrives; checking it
means following the pattern's "How to check a repository" section and filling in the row. Status values:
[shared/vocabulary.md](shared/vocabulary.md#ledger). A row that is `affected` or `fixed` names the local issue, and that
issue names the pattern in its `pattern:` field.

When code that a row describes changes a lot, check the pattern again and update Checked, Commit and Evidence.

| Pattern | Status | Checked | Commit | Issue | Evidence |
|---|---|---|---|---|---|
| PAT-001 | affected | 2026-10-27 | 3f6c1b7 | PLA-001 | src/main_planodia_boinc.cpp:194-196: called in the condition and again in the body, one boinc_checkpoint_completed(); +1 critical-section level whenever the timer fires on idx % 20 != 0 |
| PAT-002 | not-affected | 2026-10-27 | 3f6c1b7 |  | main:153-157 with trim_output() main:57-82: out cut back to header + checkpoint count, torn tail dropped, restart from 0 if out is short; every line fflushed before the checkpoint that counts it (main:190-195); no end marker. In-place rewrite: PAT-003 |
| PAT-003 | affected | 2026-10-27 | 3f6c1b7 | PLA-004 | (a) trim_output() rewrites out in place (main:77-80); (b) fprintf/fwrite/fclose and boinc_rename results unchecked (main:51-53, 79-80); (c) boinc_rename used; (d) boinc_checkpoint_completed() after a failed write (main:196) |
| PAT-004 | affected | 2026-10-27 | 3f6c1b7 | PLA-014 | main:194: written every 20 light curves regardless of the timer, outside a critical section: every ~1.6 s on a CPU, ~80 ms at 4 ms per light curve (GPU); tiny file, tmp + boinc_rename |
| PAT-005 | affected | 2026-10-27 | 3f6c1b7 | PLA-013 | read_checkpoint() main:37-38 and trim_output() main:58-59 treat any open failure as a missing file; main:158-160 then opens out with "wb" |
| PAT-006 | affected | 2026-10-27 | 3f6c1b7 | PLA-012 | main:161, 190-191, 200: fprintf/fflush/fclose on out unchecked before boinc_finish(0) at main:204 |
| PAT-007 | affected | 2026-10-27 | 3f6c1b7 | PLA-005 | boinc_fraction_done() only after each light curve (main:193), nothing at resume. Format 1: about 0.5 s per light curve on an M1 core (1.3 s with 3 passes); a heavy format-2 setting 19 s on an M1 core, so over a minute on CPUs 4x slower |
| PAT-008 | n/a | 2026-10-27 | 3f6c1b7 |  | no boinc_get_status, quit_request, abort_request or direct_process_action in src/; default options (main:90-95), the library handles quit and abort |
| PAT-009 | n/a | 2026-10-27 | 3f6c1b7 |  | no trickle messages: no boinc_send_trickle_up in the repository |
| PAT-010 | n/a | 2026-10-27 | 3f6c1b7 |  | the BOINC worker only computes and reads regular files (boinc_fopen, fread in skt.h); the PAT-010 grep has no hits |
| PAT-011 | affected | 2026-10-27 | 3f6c1b7 | PLA-002 | main:85-86: device only from --device (default 0); no boinc_get_init_data or gpu_opencl_dev_index in src/ |
| PAT-012 | affected | 2026-10-27 | 3f6c1b7 | PLA-003 | planodia_opencl.cpp:126 counts CL_DEVICE_TYPE_GPU, :137 fetches CL_DEVICE_TYPE_ALL; :128-134 CL_DEVICE_TYPE_ALL fallback; platform matched to the build's EXPECTED_GPU_VENDOR, not gpu_type |
| PAT-013 | affected | 2026-10-27 | 3f6c1b7 | PLA-006, PLA-019 | main:130 (gpu_init failure) and main:187 (buffer allocation, kernel launch, clFinish failures) end in boinc_finish(1); no boinc_temporary_exit in src/ |
| PAT-014 | n/a | 2026-10-27 | 3f6c1b7 |  | kernel is float only (planodia_opencl.cpp:23-91; planodia_opencl.h:2); no double, no cl_khr_fp64 |
| PAT-015 | affected | 2026-10-27 | 3f6c1b7 | PLA-022 | kernel needs exactly 64 (reqd_work_group_size, planodia_opencl.cpp:26, 225); device limit checked with a message (:142-150), CL_KERNEL_WORK_GROUP_SIZE not checked (:171-173) |
| PAT-016 | not-affected | 2026-10-27 | 3f6c1b7 |  | at most 4096 periods per launch (kChunk, planodia_opencl.cpp:102). Estimate: README 4 ms per light curve on an RTX 5070 Ti over 6 launches (about 23,400 periods) = at most 0.7 ms per launch. Measured on an Apple M1 2026-10-27 (synthetic): format 1 at most 9 ms per launch; format-2 setting at the M1 local-memory limit (11,146 points, 3,500 bins, 8 durations, 35 launches) at most 36 ms |
| PAT-017 | fixed | 2026-10-27 | 3f6c1b7 | PLA-007 | src/cl_dynload.c:38-84, 119-121: android_dlopen_ext through the sphal, vndk, default and a private namespace, added in 3f6c1b7 |
| PAT-018 | affected | 2026-10-27 | 3f6c1b7 | PLA-009, PLA-010 | GPU against CPU measured to differ (45/50 lines, 5/50 peak lists, M1, PLA-009); no -ffp-contract in tools/; planodia_cli -O2 arm64: GCC 13.2 5 fmadd, Apple clang 14 none, outputs identical on 50 synthetic light curves; validator unknown (PLA-010) |
| PAT-019 | not-affected | 2026-10-27 | 3f6c1b7 |  | out written "wb"/"ab" and trimmed "rb"/"wb" (main:58, 77, 157, 160); checkpoint is text mode (main:49) but one integer read with fscanf (main:40); in parsed with fscanf; shards "rb" (skt.h:51) |
| PAT-020 | n/a | 2026-10-27 | 3f6c1b7 |  | single-threaded: no threads and no thread_local in src/ |
| PAT-021 | affected | 2026-10-27 | 3f6c1b7 | PLA-008 | tools/build_android.sh:33-35 -idirafter /mnt/c/temp and :9, 12-13 /root/ paths, none overridable; also build_mac.sh:29-30, build_cpu.sh:8, 12, build_gpu_server.sh:13, 41, 53-54, ingest.py:23 |
| PAT-022 | not-affected | 2026-10-27 | 3f6c1b7 |  | CPU Linux builds fully static (build_cpu.sh:17); OpenCL Linux builds in ubuntu:20.04 (glibc 2.31), newest GLIBC version printed (build_gpu_server.sh:36-38) but not enforced |
| PAT-023 | affected | 2026-10-27 | 3f6c1b7 | PLA-020 | tools/ingest.py:58-70, 160-169: a light curve that failed 4 times is dropped like a 404 and its shard written, never retried; :141-142 target list downloaded to its final name and reused if cut. Shards themselves tmp + os.replace (:112-118) |
| PAT-024 | n/a | 2026-10-27 | 3f6c1b7 |  | no CUDA build (only OpenCL, compiled at run time by the driver); -arch hits are macOS CPU architectures |
| PAT-025 | n/a | 2026-10-27 | 3f6c1b7 |  | the BOINC application starts no threads (no std::thread, pthread_create, OpenMP in src/) |
| PAT-026 | affected | 2026-10-27 | 3f6c1b7 | PLA-011 | planodia_core.h:230 std::partial_sort by power without a tie-break decides the reported peaks; tie order measured to differ between libc++ and libstdc++ (M1). nth_element (:49) and sort (:200) use values only |
| PAT-027 | affected | 2026-10-27 | 3f6c1b7 | PLA-015 | planodia_opencl.cpp:197-198, 205, 212 clEnqueueWriteBuffer, :214-224 clSetKernelArg, :230-232 clEnqueueReadBuffer unchecked; only the launch and clFinish are checked (:226-229) |
| PAT-028 | not-affected | 2026-10-27 | 3f6c1b7 |  | guards: PL_OPENCL (main:23-31, 127, 180) and EXPECTED_GPU_VENDOR (planodia_opencl.cpp:12-14, default empty = no vendor filter); all four scripts pass -DPL_OPENCL -DCL_TARGET_OPENCL_VERSION=120 -DEXPECTED_GPU_VENDOR to every OpenCL object; CPU builds omit PL_OPENCL by design |
| PAT-029 | affected | 2026-10-27 | 3f6c1b7 | PLA-021 | build_gpu_server.sh:36 ubuntu:20.04 by tag, :19 apt-get packages unversioned (compiler, static libstdc++, OpenCL headers end up in the Linux binaries); BOINC trees on the build machines not recorded |
| PAT-030 | n/a | 2026-10-27 | 3f6c1b7 |  | no boinc_temporary_exit() and no deliberate exit without boinc_finish() in src/ (git grep) |
