<!--
Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements. See the NOTICE file distributed with
this work for additional information regarding copyright ownership.
The ASF licenses this file to You under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with
the License. You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# IMAGING-371 Group Work Log

All dates use Australia/Sydney time. Completed work, reported handoff results, and pending work are distinguished below. Estimates and planned activities are not recorded as actual hours or completed checks.

## Project and group

- Upstream repository: https://github.com/apache/commons-imaging
- Group fork: https://github.com/lilth2/commons-imaging
- Issue: [IMAGING-371 - GPSVersionID gets skipped](https://issues.apache.org/jira/browse/IMAGING-371)
- Topic branch: `fix/imaging-371`
- Baseline: `04bdd7fbb96b36df3aa1ebb80aad0a468234e5de`
- Implementation commit: `aededf8d4309e072b828d69383d874b956012cfc`

| Member | GitHub username | Responsibility |
| --- | --- | --- |
| Jiaao Liu | lilth2 | Coordination, baseline preparation, and submission records |
| Yiran Zhang | ethan-Zhang-Happy | Reproduction, diagnosis, and implementation handoff |
| Shuyang Feng | shuyangyuzu-cmd | Focused repair, core regression tests, and implementation records |
| Qingyuan Zhu | Se-m1Ne | Independent boundary validation, final build verification, and findings |

## Incoming handoff

Shuyang reported that the first two members had completed their assigned work before the implementation stage. The supplied `IMAGING-371_Yiran_Complete_Handoff.md` contains Yiran's diagnosis, reproduction sources, complete logs, input hashes, and the next-owner responsibilities. Earlier results below are attributed to that handoff rather than described as commands rerun in Shuyang's environment.

| Owner | Work and evidence reported in the handoff | Result |
| --- | --- | --- |
| Jiaao Liu | Coordination and baseline preparation were reported complete by the group. | Implementation proceeded from the explicitly identified baseline; no additional unseen commands or review records are inferred. |
| Yiran Zhang | Generated two 56-byte classic TIFF metadata fixtures independently of the Imaging writer, with GPSVersionID `[2, 3, 0, 0]` and neighbouring GPSLatitudeRef `N`. | Version field missing for both byte orders, both strict settings, and the public metadata API; the neighbour remained readable. |
| Yiran Zhang | Traced the omission to the unconditional `tag == 0` filter in `TiffReader.readDirectory`. | GPSVersionID is valid tag `0x0000` in the GPS directory; the filter discarded it before field construction. |
| Yiran Zhang | Ran the baseline default Maven build using Java 23.0.2 and Maven 3.9.11 on macOS. | Reported BUILD SUCCESS: 1,259 tests, zero failures/errors, seven skipped. This baseline result does not establish that GPSVersionID was preserved. |

## Shuyang Feng implementation record

| Date | Work completed | Actual result and evidence |
| --- | --- | --- |
| 2026-10-30 | Read the supplied diagnosis and inspected the local checkout, contribution instructions, parser, constants, and existing tests. | Existing `fix/imaging-240` worktree was clean; the required IMAGING-371 commit was not yet available locally. |
| 2026-10-30 | Fetched the group fork and the exact upstream baseline. | Initial restricted-network attempts failed to resolve github.com. The repeated fetch succeeded with network access. No remote `fix/imaging-371` branch was exposed by the group-fork fetch. |
| 2026-10-30 | Created `fix/imaging-371` from the exact baseline. | HEAD matched `04bdd7fbb96b36df3aa1ebb80aad0a468234e5de`. The unrelated IMAGING-240 fix and log were not merged into this issue. |
| 2026-10-30 | Added `TiffGpsVersionIdTest` before changing production source. | Ten parameterized invocations cover both byte orders, strict mode on/off, public metadata access, exact version bytes, adjacent entries, and non-GPS tag-zero filtering. |
| 2026-10-30 | Ran the new regression and existing `TiffReadTest` and `TiffReadWriteTagsTest` against unchanged production source. | Exit 1: 14 tests, six assertion failures, zero errors/skips. All six GPS cases failed on the missing version field. Four non-GPS checks and four existing tests passed. Evidence: `imaging371-evidence/red.log` and JSON. |
| 2026-10-30 | Made the tag-zero guard conditional on directory type using `TiffDirectoryConstants.DIRECTORY_TYPE_GPS`. | GPS tag zero is retained. Other directories retain the original early filter. Existing type decoding and field range checks remain in place. |
| 2026-10-30 | Reran exactly the same targeted test selection. | Exit 0, BUILD SUCCESS: 14 tests, zero failures/errors/skips. Evidence: `imaging371-evidence/green.log` and JSON. |
| 2026-10-30 | Ran the full default Maven build in the restricted environment. | All 1,269 tests completed with zero failures/errors and seven skips. The build subsequently failed at Moditect's jdeps output-path step; SPDX network resolution also failed. This run was not reported as a successful full build. Evidence: `imaging371-evidence/full.log` and JSON. |
| 2026-10-30 | Repeated the full default build outside the restricted environment with identical source and goals. | Exit 0, BUILD SUCCESS, 4 min 39 sec, finished 19:44:27 AEDT. 1,269 tests, zero failures/errors, seven skipped. Coverage, module generation, RAT, Checkstyle, SpotBugs, PMD, and JavaDoc completed. Evidence: `imaging371-evidence/full-unrestricted.log` and JSON. |
| 2026-10-30 | Exported the two-file candidate patch, source hashes, and repository status. | Patch reverse-applicability and whitespace checks passed. Evidence: `IMAGING-371-candidate.patch`, `candidate-hashes.json`, and `candidate-status.txt` in the external evidence folder. |
| 2026-10-30 | Received group confirmation before committing. | Shuyang reported that group members had confirmed the changes and authorised submission. Independent boundary testing and final tutor viewing are separate activities; neither is inferred from this confirmation. |
| 2026-10-30 | Verified that source hashes still matched the validated candidate and committed the implementation and tests. | Commit `aededf8d4309e072b828d69383d874b956012cfc`, authored by Shuyang Feng, contains only `TiffReader.java` and `TiffGpsVersionIdTest.java`. The course log is maintained in a separate commit. |
| 2026-10-30 | Prepared this English work log, retained Qingyuan's pending responsibilities, and checked the added documentation. | English-text and whitespace checks passed. The offline Apache RAT check reported BUILD SUCCESS, exit 0, with no licence issues. Evidence: `imaging371-evidence/log-rat.log`. |

## Repair and core coverage

The production change is limited to the tag-zero condition and an explanatory comment. A GPS directory can contain GPSVersionID with tag zero, whereas the original filter continues to reject tag zero in other directories. This is a focused metadata-preservation fix, not a claim to validate every malformed GPS entry or TIFF variant.

The new tests construct inputs with `ByteBuffer` without using the Imaging writer or downloading images:

| Test group | Invocations | Required behaviour |
| --- | ---: | --- |
| Reader, both byte orders and strict settings | 4 | Traverse the GPS directory, preserve latitude reference `N`, and retain GPSVersionID with exact bytes `[2, 3, 0, 0]`. |
| Public `Imaging.getMetadata(byte[])`, both byte orders | 2 | Find the version in the exact GPS directory and preserve the version bytes and neighbour. |
| Non-GPS root tag zero, both byte orders and strict settings | 4 | Skip an invalid BYTE entry advertising count `0xffffffff` and retain the adjacent ImageDescription `X`, without allocating the advertised payload. |

These metadata-only classic TIFF inputs are not displayable photographs. The new tests do not by themselves establish JPEG EXIF, malformed GPS, or BigTIFF-specific coverage. Existing tests run during the full build are distinct from newly designed issue-specific checks.

## Reproduction commands and environment

Shuyang's validation environment was Windows 11, Oracle Java 17.0.18, and Maven 3.9.9. The project configured Java release 8. This is different from Yiran's recorded environment.

Run from the repository root with Maven available:

```sh
mvn -B test -Dtest=TiffGpsVersionIdTest,TiffReadTest,TiffReadWriteTagsTest
mvn -B
```

The second command supplies no goal and uses the project default:

```text
clean verify apache-rat:check checkstyle:check spotbugs:check pmd:check javadoc:javadoc
```

The recorded executions also supplied a task-local Maven cache through `-Dmaven.repo.local`. No check was disabled to obtain the final success. External local evidence is under `E:\Computing\6120\A5\imaging371-evidence`; the runner is `E:\Computing\6120\A5\Run-Imaging371Checks.ps1`. The before-repair evidence is retained in `red.log`; rerunning the test on the repaired source should pass.

## Qingyuan Zhu pending independent validation

This section is intentionally retained for Qingyuan to continue the work. It describes pending responsibilities, not completed results. Review the implementation commit and design independent tests based on the final change.

- [ ] Verify the repair and core regression assertions independently.
- [ ] Check non-GPS subdirectories as well as the root directory, including malformed tag-zero count and offset combinations with bounded inputs.
- [ ] Investigate malformed GPS field types, counts, and offsets against the existing parser contract, including strict and non-strict behaviour.
- [ ] Check missing GPS directories and preservation of adjacent entries where relevant.
- [ ] Assess whether the shared-reader change warrants focused JPEG EXIF, writer/read round-trip, or BigTIFF checks. Record which paths were actually tested.
- [ ] Add any justified boundary regression tests without disabling or weakening existing assertions.
- [ ] Run the final complete default Maven build after any additional changes; retain commands, environment, test counts, exit codes, and logs.
- [ ] Record findings, resolutions, and the final validation result below; obtain group review before committing new contributions.

### Qingyuan findings and completion record

Status: Pending. No independent execution, findings, additional tests, or final acceptance have been recorded yet.

| Date | Check or review scope | Finding | Resolution | Evidence and outcome |
| --- | --- | --- | --- | --- |
| Pending | To be completed by Qingyuan after actual review and execution | Pending | Pending | Pending |

## Submission status

The implementation has been committed locally following the group confirmation reported by Shuyang. This log accompanies that implementation in the group branch. Final independent validation, final tutor viewing, and an upstream pull request remain to be recorded after they occur.

This course record is kept separate from the implementation commit so a future upstream contribution can include the focused source and tests without the administrative log.
