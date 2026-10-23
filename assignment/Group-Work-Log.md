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

# IMAGING-240 Group Work Log

All dates use Australia/Sydney time. Entries describe the status at the time of each activity; later entries supersede earlier pending statuses. Estimates are not actual hours worked. Failed attempts are retained.

## Group Members

| Name | GitHub username |
| --- | --- |
| Shuyang Feng | shuyangyuzu-cmd |
| Jiaao Liu | lilth2 |
| Qingyuan Zhu | Se-m1Ne |
| Yiran Zhang | ethan-Zhang-Happy |

Repository: [Apache Commons Imaging](https://github.com/apache/commons-imaging).
Group fork: [lilth2/commons-imaging](https://github.com/lilth2/commons-imaging).
Selected issue: [IMAGING-240 - JPEG: Not a Valid TIFF File](https://issues.apache.org/jira/browse/IMAGING-240).
Assignment: [COMP2120/COMP6120 Assignment 5](https://comp.anu.edu.au/courses/comp2120/assignments/assignment5/).

Evidence paths below refer to the group's local A5 handoff folder unless an explicit repository path or URL is given. Local evidence and planning documents are not included in this repository log commit. Earlier members' original files were not supplied with the merged log; their results are recorded as handoff statements, not independently verified results from Shuyang's environment. References to unavailable files are identified explicitly.

## Jiaao Liu - Environment and Baseline

| Date | Member | Work completed | Result | Evidence |
| --- | --- | --- | --- | --- |
| 2026-10-23 | Jiaao Liu | Read CONTRIBUTING.md, README.md, pom.xml, and CI configuration. | Identified requirements for a topic branch, issue ID in commit messages, regression tests, and default Maven checks. | Original CONTRIBUTING.md, pom.xml, and .github/workflows/maven.yml references; original handoff copies not supplied. |
| 2026-10-23 | Jiaao Liu | Cloned Apache upstream and created fix/imaging-240. | Completed; a personal fork had not yet been connected. | [Baseline 4ee55f32](https://github.com/apache/commons-imaging/commit/4ee55f32c6a8c35504a70ee708d6de6c34a1ae41). |
| 2026-10-23 | Jiaao Liu | Checked Java and configured local Maven. | OpenJDK 23.0.2 and Maven 3.9.16; Maven SHA-512 verified; no global environment changes. | Original Enter-Imaging.ps1 and environment README.md; not supplied. |
| 2026-10-23 | Jiaao Liu | Ran the first default Maven baseline build. | 1,259 tests, zero failures/errors, seven skipped. Full build failed with 646 CRLF Checkstyle violations. | Original baseline-mvn.log and baseline-result.json; not supplied. |
| 2026-10-23 | Jiaao Liu | Corrected repository-local line-ending settings. | Set core.autocrlf=false and core.eol=lf; restored LF and confirmed a clean tree with no source logic or staged changes. | Original CI configuration and restore_lf.py; not supplied. |
| 2026-10-23 | Jiaao Liu | Reran the default Maven build. | BUILD SUCCESS, exit 0, 3 min 04 sec; 1,259 tests, zero failures/errors, seven skipped; all default checks passed. | Original baseline-lf-mvn.log and baseline-lf-result.json; not supplied. |
| 2026-10-23 | Jiaao Liu | Verified the group fork, connected origin, retained upstream, and fetched origin. | origin/master matched local HEAD. Local fix/imaging-240 had not been pushed at this stage. | [Group fork](https://github.com/lilth2/commons-imaging); baseline 4ee55f32. |

## Yiran Zhang - Reproduction

Yiran used a separate environment with Java 23.0.2 and Maven 3.9.11. These entries do not imply changes to Jiaao's environment or local branch.

| Date | Member | Work completed | Result | Evidence |
| --- | --- | --- | --- | --- |
| 2026-10-23 | Yiran Zhang | Confirmed the issue and cloned the group fork at a fixed version. | Selected IMAGING-240 and baseline 4ee55f32. | [Issue](https://issues.apache.org/jira/browse/IMAGING-240); [baseline](https://github.com/lilth2/commons-imaging/commit/4ee55f32c6a8c35504a70ee708d6de6c34a1ae41). |
| 2026-10-23 | Yiran Zhang | Prepared Java, local Maven, and the original attachment; built the current version. | BUILD SUCCESS with tests, Checkstyle, and RAT skipped; not a full validation. | Original build-current.log and img.jpg; handoff package not supplied. |
| 2026-10-23 | Yiran Zhang | Wrote a minimal Java reproducer and tested 1.0-alpha1. | Reproduced IOException: Not a Valid TIFF File; ImageIO and getImageSize succeeded. | Original Reproduce240.java and alpha1.log; not supplied. |
| 2026-10-23 | Yiran Zhang | Ran the same reproducer against master, 1.0.0-alpha7-SNAPSHOT. | Reproduced the same exception on the current version. | Original current.log; not supplied. |
| 2026-10-23 | Yiran Zhang | Inspected EXIF bytes and TIFF parsing. | Identified a missing four-byte offset at the end of an EXIF subdirectory; TiffReader encountered EOF. No library source fix at this stage. | Original analysis README.md and current.log; not supplied. |
| 2026-10-23 | Yiran Zhang | Organized scripts, dependencies, and notes; reran both versions and condensed the work log. | Both runs exited 1 with the expected exception. Detailed records retained separately by Yiran. | Original run.sh, README.md, recheck-alpha1.log, recheck-current.log, and detailed operation log; not supplied. |

## Initial Handoff Plan

At this point the issue had been reported as reproduced on both versions, but no production fix or repository regression test had been completed. The merged handoff preserved seven Jiaao entries and six Yiran entries. The original source-log attachment was not available locally.

| Priority | Planned task | Acceptance criteria |
| --- | --- | --- |
| 1 | Receive the reproducer and verify actual branch and baseline state; run the current-version reproducer. | Preserve the exception and actual repository state; do not assume an unpushed local branch is available remotely. |
| 2 | Add repository regression tests and define expected getImageInfo behaviour. | Demonstrate failure before the fix; verify 731 by 727 dimensions and expected error handling. |
| 3 | Evaluate JPEG/EXIF tolerance and TIFF strict parsing, then implement a focused fix. | Regression passes without indiscriminately suppressing IOException; check valid and damaged inputs. |
| 4 | Run relevant tests and the complete default Maven build. | Preserve commands, exit codes, test counts, and output; skipped checks do not establish full success. |
| 5 | Document, review, commit, and hand off the fix. | Use an IMAGING-240 commit message and real evidence; record a PR URL only after creation. |

## Shuyang Feng - Planning and Handoff Review

| Date | Member | Work completed | Result | Evidence |
| --- | --- | --- | --- | --- |
| 2026-10-23 | Shuyang Feng | Read the merged log and inventoried the local A5 folder. | Only the merged log was initially present; source, sample, reproducer, and original test outputs were absent. Earlier work was treated as handoff information. | Local file inventory and supplied merged log. |
| 2026-10-23 | Shuyang Feng | Checked the assignment page, issue, and upstream contribution instructions. | Confirmed personal contribution records, peer review, regression tests, and default Maven validation requirements. Issue displayed Unresolved/Unassigned. Tutor approval was initially unconfirmed. | Assignment and issue URLs above; repository CONTRIBUTING.md. |
| 2026-10-23 | Shuyang Feng | Planned root-cause validation, focused implementation, and core regression tests. | Estimated 8-12 person-hours; proposed independent testing and review for Qingyuan. This was a plan, not completed work or measured time. | Local personal task plan, maintained separately. |

Recording practice: append completed activities with actual results, including failures; distinguish plans from results and estimates from actual time. Do not prefill commits, review outcomes, or successful checks. Operations outside the recorded session require evidence before being added.

## Shuyang Feng - Clone, Investigation, Implementation, and Validation

| Date / time | Member | Work completed | Result | Evidence |
| --- | --- | --- | --- | --- |
| 2026-10-23 12:32 AEDT | Shuyang Feng | Attempted to clone https://github.com/lilth2/commons-imaging.git into A5/commons-imaging. | First attempt failed, exit 1: Could not resolve host: github.com. | Recorded clone command output. |
| 2026-10-23 12:32 AEDT | Shuyang Feng | Retried the same clone with network access. | Success, exit 0; 877 files checked out at 4ee55f32c6a8c35504a70ee708d6de6c34a1ae41. | Local repository and clone output. |
| 2026-10-23 12:32 AEDT | Shuyang Feng | Added Apache upstream and checked status, remotes, HEAD, and branches. | Clean master tracking origin/master; origin was the group fork. upstream configured but not fetched. No remote fix/imaging-240 branch existed then; baseline matched the handoff. | Local clone-status.txt. |
| 2026-10-23 | Shuyang Feng | Confirmed tutor approval and inspected contribution guidance, attributes, JPEG code, and TiffReader. | Tutor approval was confirmed by Shuyang in the project discussion; no email copy supplied. Unconditional nextDirectoryOffset reading matched the root-cause hypothesis, pending testing. | Recorded approval confirmation; CONTRIBUTING.md; TiffReader.java. |
| 2026-10-23 | Shuyang Feng | Checked Java/Maven, configured local LF settings, and created fix/imaging-240. | Java 17.0.18; Maven absent from PATH, but an existing Maven 3.9.9 was usable. Version output included an Access is denied message. No global environment changes. Initial checkout command did not ultimately normalize all files, as discovered later. | Version output; core.autocrlf=false; core.eol=lf; git switch -c fix/imaging-240. |
| 2026-10-23 | Shuyang Feng | Downloaded the original public JIRA attachment. | Download complete; reproduction had not yet been independently established. | Local evidence/img.jpg; [original attachment](https://issues.apache.org/jira/secure/attachment/12985928/img.jpg). |
| 2026-10-23 | Shuyang Feng | Inspected EXIF structure using a read-only Python script. | JPEG 114,885 bytes; little-endian TIFF 193 bytes. EXIF subdirectory at 155 contains three entries ending at 193, leaving no footer bytes. Root next offset 7,864,320 is outside the EXIF data. Confirmed missing footer without extending the fix to other corruption. | Local evidence/inspect_exif.py and evidence/exif-structure.txt. |
| 2026-10-23 | Shuyang Feng | Ran Maven test -Dtest=JpegReadTest using a task-local dependency cache. | BUILD SUCCESS, exit 0, 2 min 16 sec; selected JPEG baseline only, not the full default build. | Local evidence/baseline-jpeg.log and baseline-jpeg.exit. |
| 2026-10-23 | Shuyang Feng | Added JpegImaging240Test and the original attachment as a test resource; started pre-fix tests. | Covered dimensions, retained EXIF entries, strict rejection, a completed footer, and root truncation. Corrected an extra closing brace during editing; production source unchanged at this stage. | src/test/java/org/apache/commons/imaging/formats/jpeg/JpegImaging240Test.java; local evidence/imaging240-red.log. |
| 2026-10-23 | Shuyang Feng | Completed pre-fix regression execution. | Five tests: zero assertion failures, two errors, zero skipped; image-info and metadata tests raised the target IOException. Three boundary tests passed. Exit 1 established the actual defect rather than a compile or resource error. | Local evidence/imaging240-red.log and imaging240-red.exit. |
| 2026-10-23 | Shuyang Feng | Implemented focused handling of a missing subdirectory footer in TiffReader.readDirectory. | For classic TIFF with ignoreNextDirectory=true, non-strict mode, and fewer than four footer bytes, preserve entries, record a compliance comment, and use offset zero. Root, strict, and BigTIFF reading retained existing behaviour. No general IOException catch. | src/main/java/org/apache/commons/imaging/formats/tiff/TiffReader.java. |
| 2026-10-23 | Shuyang Feng | Ran JpegImaging240Test, JpegReadTest, TiffReadTest, and TiffReadWriteTagsTest. | BUILD SUCCESS, 23 tests, zero failures/errors/skipped, exit 0, 52.975 seconds. Image dimensions 731 by 727; three EXIF fields retained; strict rejection and root-truncation rejection preserved; completed footer accepted. | Local evidence/imaging240-green.log and imaging240-green.exit. |
| 2026-10-23 | Shuyang Feng | Created a repeatable validation script, normalized the two edited Java files, checked the diff, and started the full build. | git diff --check passed. Full mode supplies no Maven goal, using the default clean verify, RAT, Checkstyle, SpotBugs, PMD, and Javadoc sequence. Result pending at this stage. | Local Run-ImagingChecks.ps1 and evidence/full-*.log with JSON metadata. |
| 2026-10-23 | Shuyang Feng | Prepared implementation notes, handoff material, and a PR draft. | Documented root cause, scope, contribution boundaries, five core cases, 23 related tests, and pending review. No PR published. | Local handoff document, PR-description-draft.md, and Run-ImagingChecks.ps1. |
| 2026-10-23 | Shuyang Feng | Checked the full test summary, source diff, and sample hash. | 1,264 tests, zero failures/errors, seven skipped; five more tests than the earlier baseline. Diff check passed and resource matched the downloaded sample. Later default checks were still running. | Local evidence/full-20261023-124135.log; SHA-256 below. |
| 2026-10-23 | Shuyang Feng | Analyzed the first full-build failure. | Exit 1, 4 min 17 sec. Tests passed, but Checkstyle reported 645 CRLF violations. Earlier checkout-index had not rewritten existing line endings. This was not a successful full build. | Local evidence/full-20261023-124135.log and matching .log.json. |
| 2026-10-23 | Shuyang Feng | Wrote and ran index-aware LF normalization, then restarted the full build. | Converted only tracked LF text with CRLF/mixed working-tree endings; binary data and source logic preserved. | Local evidence/normalize_tracked_lf.py and lf-normalization.txt. |
| 2026-10-23 | Shuyang Feng | Verified normalization and index state. | All 645 violations were line-ending issues. Normalized 715 text files. Refreshed the index with git add --renormalize, then unstaged the source change. Actual changes remained only the fix, test, and sample; no unrelated staged changes. Diff check passed. | Local normalization list; Git status and staged-diff output. |
| 2026-10-23 | Shuyang Feng | Analyzed the second full-build failure. | 1,264 tests, zero failures/errors, seven skipped; Checkstyle zero violations and SpotBugs zero bugs/errors. PMD found one TooManyStaticImports violation caused by three new static imports; full build exit 1. | Local evidence/full-20261023-124720.log, matching JSON, and pmd-static-import-failure.xml. |
| 2026-10-23 | Shuyang Feng | Replaced the three new static imports with TiffConstants-qualified references; started a third full build. | Parsing behaviour unchanged; no rule disabled or skipped. Restored LF for the edited file; diff check passed. | TiffReader.java; final full-build log. |
| 2026-10-23 | Shuyang Feng | Exported the source, test, and sample into a transferable patch. | Patch 148,313 bytes. git apply --reverse --check passed without applying or reverting changes. | Local evidence/IMAGING-240.patch and export_patch.py. |
| 2026-10-23 13:01 AEDT | Shuyang Feng | Completed the final full default Maven build. | BUILD SUCCESS, exit 0, Maven duration 3 min 53 sec. 1,264 tests, zero failures/errors, seven skipped. RAT, coverage, Checkstyle, SpotBugs, PMD, and Javadoc passed. Only batch mode and local cache options supplied; no checks skipped. | Local evidence/full-20261023-125734.log and matching .log.json. |
| 2026-10-23 | Shuyang Feng | Archived final reports, updated handoff notes and the PR draft, and performed final Git checks. | Saved Checkstyle, PMD, SpotBugs, and regression XML reports. Diff check passed; staging empty. Implementation phase complete; independent review remained pending at that time. No commit, push, PR, or measured personal hours claimed then. | Local evidence/final-reports, final-source-diff.txt, handoff notes, PR draft, and patch. |

Attachment SHA-256: `a8a525bc05d2ed014e77806189fd2d97437e9f768e799e224015ff0761dcc985`.

## Group Confirmation

| Date | Members | Activity | Result | Evidence |
| --- | --- | --- | --- | --- |
| 2026-10-23 | Shuyang Feng, Jiaao Liu, Qingyuan Zhu, Yiran Zhang | Shuyang reported that all members had confirmed all his IMAGING-240 changes, including source, regression tests, and sample. | Group confirmation complete, superseding the earlier pending confirmation status. This does not establish independent test execution by each member or tutor review of the final changes. | Shuyang's explicit statement in the project discussion; no individual review comments or screenshots supplied. |

## Commit and Push

| Date | Member | Work completed | Result | Evidence |
| --- | --- | --- | --- | --- |
| 2026-10-23 | Shuyang Feng | Checked Git identity, branch, remotes, and changes; staged only the fix, regression test, and original sample. | Author Shuyang Feng; branch fix/imaging-240. Staged diff check passed: three files, 92 insertions, one deletion, and a 114,885-byte sample. External work notes, dependency cache, and build outputs excluded. | Staged diff and existing final-build evidence. |
| 2026-10-23 | Shuyang Feng | Committed and ran git push -u origin fix/imaging-240. | Commit and push succeeded, exit 0; remote branch created and tracking configured. Clean working tree. No source changes since validation and no redundant test rerun. No PR created. | [Commit 90e3ce1e](https://github.com/lilth2/commons-imaging/commit/90e3ce1ed2c2e15d4ae10368f0f3ca29b0ec4133); [branch](https://github.com/lilth2/commons-imaging/tree/fix/imaging-240). |

## English Log Revision

| Date | Member | Work completed | Result | Evidence |
| --- | --- | --- | --- | --- |
| 2026-10-23 | Shuyang Feng | Requested an English filename and English text throughout the merged work log, and a new submission. | Renamed the local merged log to Group-Work-Log.md, translated the historical entries, retained failures and evidence limitations, and prepared a repository copy at assignment/Group-Work-Log.md. This is a documentation revision; source and tests unchanged. English-text and annotation checks passed; the offline Apache RAT check passed (exit 0, 16.051 seconds). Submission status is available in Git history and the remote branch. | This document and its Git history. |

## Next Work

Qingyuan's proposed independent verification and additional boundary tests remain future work, separate from the reported group confirmation. Suggested cases include one to three remaining footer bytes, big-endian EXIF, truncated entries, and fail-on-error compliance behaviour. Record actual results and reviewer feedback when available.

The implementation commit is available remotely. Tutor review of final changes and creation of an upstream PR have not been recorded. The 8-12 person-hour estimate remains a plan, not measured time.
