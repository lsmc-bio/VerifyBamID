# LSMC VerifyBamID distribution

Release **2.0.4** imports Tim Fennell's
[upstream PR #99](https://github.com/Griffan/VerifyBamID/pull/99) at
`c2b66f6f6ea85e7d88ea407c16c29bf46749d0fd`. The PR was open and unmerged at
import. This is an LSMC fork release, not an upstream release or an assertion
of scientific acceptance. `LSMC_RELEASE.json` retains the source identity.
The executable banner identifies the fork as `2.0.4-lsmc-pr99`.

The fork retains that commit's scientific implementation, including one-pass
pileup, explicit `--ReadType`, near-indel filtering and its htslib 1.25 minimum.
DayOA selects this distribution separately from official VerifyBamID 2.0.3's
pileup-input recipe. Preserve frozen Bjuice 1.0; do not replace its dependencies.
Stage 1 and stage 2 results require separate provenance and paired evaluation
before changing the modern default. No QC thresholds change with this release.

## Native build and deployment

Build on the designated Linux EC2 compute allocation using local NVMe for
source extraction, the CMake build tree and temporary files. Retain logs at
their final FSx path. Select exact absolute compiler and CMake paths and a
published dependency prefix containing htslib 1.25, zlib, bzip2, xz and libcurl.
Build only the `VerifyBamID` target; building the executable does not run tests.

```bash
"${CMAKE_EXE:?}" -S "${SOURCE_DIR:?}" -B "${BUILD_DIR:?}" \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_COMPILER="${CC:?}" -DCMAKE_CXX_COMPILER="${CXX:?}" \
  -DCMAKE_C_FLAGS=-fsigned-char -DCMAKE_CXX_FLAGS=-fsigned-char \
  -DCMAKE_PREFIX_PATH="${DEPENDENCY_PREFIX:?}" \
  -DHTS_INCLUDE_DIRS="${DEPENDENCY_PREFIX}/include" \
  -DHTS_LIBRARIES="${DEPENDENCY_PREFIX}/lib/libhts.so" \
  -DCMAKE_BUILD_RPATH="${DEPENDENCY_PREFIX}/lib"
"${CMAKE_EXE}" --build "${BUILD_DIR}" --target VerifyBamID --parallel "${BUILD_JOBS:?}"
```

The upstream CMake project writes the binary to `SOURCE_DIR/bin/VerifyBamID`.
Publish that completed binary at
`/fsx/references/runtime_assets/prodv3/tools/verifybamid-lsmc/2.0.4/bin/VerifyBamID`.
Place its native build receipt beside the `bin` directory as
`build_receipt.json`, recording the exact fork commit, upstream PR head,
dependency prefix, compiler/CMake identities, command, host and native result.
Retain original source/package payloads and publish the runtime prefix through
DayOA's POSIX-preserving resource publisher to
`s3://lsmc-dayoa-references-usw2/runtime_assets/prodv3/tools/verifybamid-lsmc/2.0.4/`.
The dependency prefix must also be published at its canonical FSx/S3 path.
An unexported cluster-local installation is not a durable release artifact.

The retained upstream Dockerfile is historical upstream build infrastructure:
it clones an upstream tag and runs tests. It is not the LSMC native build path.
Do not use it to build this fork's release. Container builds, if later selected,
must run only on EC2 and use an explicit fork source revision.

Implementation and comparison tracking: [BFX-866](https://linear.app/lsmc/issue/BFX-866).
