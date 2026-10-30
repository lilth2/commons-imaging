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

Shuyang reported that the first two members had completed their assigned work before the implementation stage. The supplied `IMAGING-371_Yiran_Complete_Handoff.md` contains Yiran's diagnosis, reproduction sources, complete logs, input hashes, and the next-owner responsibilities. Earlier results below come from that handoff.

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
| 2026-10-30 | Received group confirmation before committing. | All four group members reviewed and approved the implementation and core tests. The user confirmed the approval status. |
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

## Qingyuan Zhu independent validation record

| Date | Work completed | Actual result and evidence |
| --- | --- | --- |
| 2026-10-31 | Fetched the latest group branch and created `imaging-371-verification` from `origin/fix/imaging-371`. Reviewed the implementation commit, `TiffReader.java`, and the existing `TiffGpsVersionIdTest.java`. | Confirmed that the fix preserves tag zero in GPS directories while retaining the original filter in other directories. The working tree was clean before validation. |
| 2026-10-31 | Independently ran the existing GPSVersionID regression tests and related TIFF/GPS tests. | All 10 existing GPSVersionID test invocations passed. Another 34 related test invocations passed. No production code changes were needed. |
| 2026-10-31 | Added two boundary regression tests to `TiffGpsVersionIdTest.java`. | Added 72 lines covering a GPS tag-zero entry with an invalid count and offset, and a GPS tag-zero entry with an unknown field type. Both tests cover little-endian and big-endian inputs with strict mode enabled and disabled. |
| 2026-10-31 | Checked how the reader handles malformed GPS fields and neighbouring entries. | Strict mode rejected the malformed count/offset case with an `IOException`. Non-strict mode discarded the malformed field and preserved the following GPSLatitudeRef. The unknown field type was skipped in both modes, while GPSLatitudeRef remained readable. |
| 2026-10-31 | Checked the new malformed-entry test against the original tag-zero behaviour. | Reverting the production fix locally caused the new strict-mode assertions to fail. The fix was then restored. No production changes were retained. |
| 2026-10-31 | Ran the updated GPSVersionID tests and the full Maven test suite. | All 18 focused test invocations passed. The full suite executed 1,277 tests, with zero failures, zero errors, and seven unrelated skips. |
| 2026-10-31 | Ran the Maven verification and quality checks in a temporary worktree at `C:\verif-imaging-371`. | Compilation, tests, Apache RAT, Checkstyle, SpotBugs, and PMD passed. The JavaDoc goal failed because Maven's Windows command invocation could not handle parentheses in the local JDK path. The same failure was reproduced on the unchanged baseline. The complete build therefore did not finish successfully. Evidence: `C:\verif-imaging-371-logs\full-build-2.log`. |
| 2026-10-31 | Reviewed the final changes and checked the working tree. | `git diff --check` passed. Only `TiffGpsVersionIdTest.java` was modified, with 72 insertions and no deletions. Existing tests and production code were unchanged. |
| 2026-10-31 | Submitted the two additional boundary tests for group review. | All four group members reviewed and approved the additional boundary tests. Committed as f33bcd40864e5554b7ee6fb362ad723677ded72f. |

### Validation scope and limitations

The additional tests check two malformed GPS tag-zero cases that were not covered by the original regression tests. They confirm that the reader still applies field-length checks and unknown-type handling after the GPSVersionID fix.

The tests use small, in-memory classic TIFF inputs. They do not establish separate coverage for JPEG EXIF integration, BigTIFF, or writer/read round trips. These paths were not added to the issue-specific tests.

The complete Maven test suite and the main quality checks passed. The JavaDoc failure was reproduced on the unchanged baseline and was not caused by the new tests. No global Java configuration was changed.

Independent validation and group review are complete. All four members approved the changes: Jiaao Liu, Yiran Zhang, Shuyang Feng and Qingyuan Zhu.

## Submission status

The implementation and core tests were committed by Shuyang Feng in `aededf8d4309e072b828d69383d874b956012cfc`. Qingyuan committed the additional boundary tests as f33bcd40864e5554b7ee6fb362ad723677ded72f. The validation record was committed as 763214c8. Both commits were verified on origin/imaging-371-verification on 2026-10-31.

The user authorised direct upstream PR submission on 2026-10-31 without an additional tutor confirmation. The clean PR branch retains both contributors' authorship and excludes the course log. Final build and publication status are recorded below.

This course record is kept separate from the technical contribution.

## Final integration and approval — 2026-10-31

All group members reviewed and approved the implementation, core tests and additional boundary tests: Jiaao Liu, Yiran Zhang, Shuyang Feng and Qingyuan Zhu. This approval status was confirmed by the user on 2026-10-31.

Verified remote implementation commit: aededf8d4309e072b828d69383d874b956012cfc. Verified remote boundary-test commit: f33bcd40864e5554b7ee6fb362ad723677ded72f. The validation log was already committed as 763214c8.

Prepared clean PR branch pr/imaging-371 with commits 5298282c and 2a10b267, preserving Shuyang's and Qingyuan's authorship. Its final diff contains only TiffReader.java and TiffGpsVersionIdTest.java. No coursework files are included.

Final default `mvn` completed successfully on the clean PR branch using OpenJDK 23.0.2 and Maven 3.9.16: 1,277 tests, zero failures, zero errors, seven skipped; 3 min 5 sec; finished 2026-10-31 09:41:41 AEDT; exit 0. Javadoc and the default quality checks passed. Evidence: [final build log](evidence/imaging-371-pr-final-mvn.log).

Upstream PR submitted on 2026-10-31: https://github.com/apache/commons-imaging/pull/569 (Open). Both technical commits and the updated course log were pushed to the group fork. The PR contains exactly two files: TiffReader.java and TiffGpsVersionIdTest.java. CI and maintainer review are pending; the PR is not merged.

## CI evidence-file correction — 2026-10-31

The verification-branch CI failed Apache RAT because assignment/evidence/imaging-371-pr-final-mvn.log had no license header. Added an Apache License 2.0 header while retaining the original Maven output. Local `mvn apache-rat:check` then passed (exit 0, 3.579 sec). This documentation correction does not change the production source or tests and is excluded from upstream PR #569. The push triggers a new verification-branch CI run; its outcome must be checked separately.
