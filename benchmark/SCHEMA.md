# DCTaint data schema

Schema version: 1.0. Data audit date: 2026-11-21.

This schema describes the four worksheets in `ground_truth.xlsx` and `dctaint_analysis.xlsx`. The term ground truth distinguishes study records and authorization annotations from analyzer-specific annotations. It does not certify every retained label, vulnerability identity, or reproduction claim. Evidence qualifications remain in the data.

## Files, record units, and identifiers

| Workbook and sheet | Record unit | Primary identifier | Release size |
| --- | --- | --- | --- |
| `ground_truth.xlsx` / `Vulnerabilities` | One retained row of the 497-record study corpus. | `case_id` | 497 rows |
| `ground_truth.xlsx` / `Authorization annotations` | One retained row of the separate authorization annotation workbook. | `annotation_id` | 188 rows |
| `dctaint_analysis.xlsx` / `Legacy analysis` | The analyzer-related legacy values associated with one authorization annotation. | `annotation_id` | 188 rows |
| `dctaint_analysis.xlsx` / `Reported evaluation` | One paper identifier/category/outcome group, potentially representing several reported instances. | `group_id` | 68 groups; `sum(bug_count) = 96` |

`case_id` values `DCT-0001` through `DCT-0497` are assigned in the original enriched corpus's worksheet order and then increasing source row. The 16 worksheets, in that order, are HDFS, YARN, MapReduce, HBASE, Zookeeper, CLOUDSTACK, SPARK, KAFKA, MESOS, MONGODB, memcached, Redis, ceph, etcd, Kubernetes, and OpenStack. Their record counts are 24, 28, 11, 46, 12, 44, 29, 23, 38, 41, 14, 33, 32, 12, 55, and 55. Once assigned, identifiers should remain stable when views are sorted or filtered.

`annotation_id` values `AUTH-0001` through `AUTH-0188` identify original authorization rows in source worksheet and row order. The `Legacy analysis.annotation_id` join is one-to-one with `Authorization annotations.annotation_id`. Neither an original issue ID nor an API name is a primary key. Repeated IDs and apparent duplicates are retained. Matching an identifier does not establish that two source rows are independent vulnerabilities.

The 188 authorization annotations are a separate historical population. They must not be equated automatically with the 190 rows classified as authorization in the 497-record corpus. Joins and discrepancies are made explicit rather than repaired through undocumented row deletion or relabeling.

## Common value conventions

- Blank cells mean that the source or the audit does not provide a value. A blank is not zero, false, a safe result, or evidence that a vulnerability is absent.
- Literal `-` values in original fields are preserved. They are historical placeholders whose precise meaning is not uniformly documented; they must not be silently converted into `no`, zero, or a negative result.
- Original text, capitalization, spelling, whitespace, labels, URLs, and numeric annotations are retained unless a correction is explicitly recorded. New provenance fields do not rewrite the original labels.
- Commit identifiers are full 40-character Git SHAs stored as text. They are interpreted within `repository_url`; the repository may differ from the original study-system label.
- `evidence_urls` contains newline-separated source URLs. A reference may identify a commit, review, issue, advisory, source artifact, or release manifest. Reference links are not all executable patches.
- `candidate_case_ids` contains newline-separated `case_id` values. It retains all candidates, including the selected identifier when `case_id` is assigned. Candidate membership is not a claim of a verified one-to-one correspondence. No candidates are represented by a blank cell.
- `original_row` is the one-based Excel row number, including the source header row in the numbering. `original_sheet` preserves the source worksheet spelling.

## `ground_truth.xlsx`: `Vulnerabilities`

This sheet contains all 17 fields from `dtaintRawdata-with-commits-20261121.xlsx` plus four identity/provenance fields. The table lists the published column order.

| Field | Type and meaning |
| --- | --- |
| `case_id` | Unique text identifier for this retained study row. |
| `study_system` | Original study-system grouping. This is a corpus category, not a guarantee of the actual affected repository or product. |
| `ID` | Original vulnerability, CVE, issue, or pull-request label. Preserved even when a mismatch is documented. Not unique. |
| `type` | Original high-level study classification: `input validation`, `fault tolerance`, `authorization`, `authentication`, or `sensitive information leak`. |
| `root cause` | Original finer-grained annotation. Values include MPC and other codes and free text; punctuation does not define a machine-readable hierarchy. |
| `impact` | Original impact annotation, such as denial of service, privilege escalation, information disclosure, or remote code execution. It is not a standardized severity score. |
| `component` | Original numeric component-count annotation. Component granularity is a process. Counts are preserved, not recomputed or newly verified by this release. |
| `source` | Original source/data-entry annotation, often an API or input label. It is not necessarily a complete file/signature/line location or a DCTaint configuration entry. |
| `sink` | Original sink/operation annotation, with the same location and configuration limitations as `source`. |
| `communication type` | Original communication annotation. Observed values are `local`, `rpc`, `RPC`, `servlet`, `rpc+event`, `socket`, `rest`, and `rpc+rest`. Case variants are preserved, including three `RPC` entries. |
| `repository_url` | Repository to which the two SHA fields refer. This is the authoritative code-attribution field when the original study label is inaccurate. |
| `vulnerable_commit` | Inspected pre-fix or affected-release source snapshot. It is not generally the vulnerability-introducing commit and is not evidence of the historical evaluated revision. |
| `fixed_commit` | Recorded repair, mitigation, follow-up, or fixed-release source snapshot, within the scope stated by the status and notes. It is not uniformly the first fixing commit. |
| `verification_status` | Audit classification from the complete vocabulary below. It describes the evidence and its limits, not a dynamic reproduction result or confidence score. |
| `link` | Original issue or advisory reference. Its retention does not resolve a conflicting original identity. |
| `patch` | Current repair or mitigation reference. It may be a commit, merged review, attachment, or vendor advisory; consult `verification_status`. |
| `evidence_urls` | Newline-separated URLs supporting the mapping or explaining its limitations. |
| `commit_notes` | Row-specific rationale, baseline and series choices, configuration requirements, original-link corrections, attribution conflicts, and unverified aspects. |
| `original_patch` | Previous patch-cell value retained when it was replaced or expanded. A blank does not mean no original patch existed; the current patch may have been retained unchanged. |
| `original_sheet` | Worksheet in the enriched 16-sheet corpus from which the row was copied. |
| `original_row` | Source Excel row number. |

The release has 496 nonblank vulnerable SHAs and 495 rows with both SHAs. The missing values are intentional and described under known issues. A source pair does not imply a successful build, an executed PoC, exploitability in every configuration, or a minimal patch interval. A merge pair generally uses the pre-merge first parent; a multi-part repair may span multiple commits and unrelated changes. Vendor release snapshots may be on divergent histories. The detailed row notes take precedence over a generic parent/child assumption.

### `verification_status` vocabulary

All 31 values occurring in this release are defined here. The names are retained from the source audit. `confirmed` and `verified` refer to the recorded source/reference checks, not to newly executed vulnerability tests.

| Value | Rows | Interpretation |
| --- | ---: | --- |
| `confirmed` | 206 | The audit identified the relevant repair commit and baseline from the available issue, review, or source evidence. Read the row notes for the exact checks. |
| `verified_source_pair` | 94 | The repair and pre-fix baseline were inspected in source. |
| `verified_fix_code_review` | 104 | Production changes were reviewed to establish the repair and the affected baseline. |
| `verified_commit` | 5 | The linked commit and its pre-fix parent were verified. |
| `verified_commit_and_advisory` | 16 | Commit/source evidence was checked against an official advisory. |
| `verified_commit_and_cve` | 2 | Commit/source evidence was checked against the CVE record. |
| `verified_merged_review` | 30 | The repair was resolved to an official merged review revision or repair series. |
| `confirmed_dependency_upgrade` | 1 | The project changes a vulnerable dependency version. The pair is a project build/dependency update, not a local repair to the dependency's source. |
| `verified_downstream_packaging_fix` | 1 | The pair is a verified downstream packaging/configuration repair, not an upstream application-code pair. |
| `verified_source_pair_configuration_required` | 8 | The source change requires the configuration or deployment conditions described in the notes to provide the mitigation. |
| `verified_source_pair_secure_default` | 1 | The change makes an existing protection enabled by default; explicit disablement can remove the protection. |
| `limited_mitigation` | 1 | A related configurable mitigation is verified, with remaining coverage limitations. It must not be counted as a universal repair. |
| `optional_mitigation_no_default_complete_fix` | 1 | Optional mitigations exist, but no fixed SHA satisfying the documented default-complete criterion is established. `fixed_commit` remains blank. |
| `verified_source_pair_disclosure_fix_ssrf_remains` | 1 | The repair closes the disclosed-information channel while the underlying outbound-request behavior remains. |
| `verified_historical_patch_known_later_bypass` | 4 | The historical repair is verified, but a later bypass or related vulnerability limits its scope. |
| `verified_source_pair_completed_by_followup` | 1 | The fixed endpoint includes a required later repair; the vulnerable endpoint remains before the original fix. |
| `verified_fix_with_advisory_errata` | 1 | The fixed endpoint includes the official advisory's subsequent erratum, while the baseline precedes the initial repair. |
| `verified_source_pair_integrated_upstream_fix` | 1 | An upstream integrated implementation or combined backport replaces the originally linked proposal. |
| `verified_source_pair_merge_first_parent` | 3 | The full merge contains the repair; its first parent is the pre-merge baseline. The repaired topic-branch parent is not used as vulnerable. |
| `verified_source_pair_missing_merge_sha_resolved` | 4 | A missing API merge-SHA field was resolved through the merged PR's actual implementation history. |
| `verified_source_pair_replaced_unmerged_pr` | 1 | The original unmerged PR was replaced by evidence of the actual integrated repair. |
| `verified_source_pair_replaced_dead_commit` | 1 | An unavailable original commit link was replaced by a verified relevant repair. |
| `verified_source_pair_replaced_wrong_pr` | 1 | The original unrelated PR was replaced by the verified repair PR. |
| `verified_source_pair_corrected_issue_as_pr` | 1 | An issue URL incorrectly presented as a PR was corrected to the actual repair PR. |
| `verified_source_pair_component_mismatch` | 1 | The pair belongs to the actual affected dependency/component, which differs from the original study-system attribution. |
| `verified_source_pair_component_mismatch_duplicate` | 2 | The actual affected component differs from the study label, and duplicate source rows are explicitly retained. |
| `verified_linked_issue_original_id_mismatch` | 1 | The linked security issue and its repair are verified, but the original row ID denotes a different issue. The original ID remains visible. |
| `verified_source_pair_later_repair_cve_attribution_inferred` | 1 | The later source repair matches the described defect, but its association with the original CVE is an inference rather than an explicit vendor identity/alias statement. |
| `verified_vendor_release_source_snapshots` | 1 | Official affected/fixed vendor sources were mapped to Git snapshots. These may be on different branches and include documented build/productization differences. No first-repair claim is made. |
| `vendor_release_snapshots_private_source` | 1 | Official vendor manifests anchor affected/fixed private-source release SHAs. The corresponding private patch and public Git ancestry are not available. |
| `issue_conflict` | 1 | The original identity/labels conflict with the available issue and code evidence. No security pair is asserted. |

## `ground_truth.xlsx`: `Authorization annotations`

This sheet retains the tool-independent fields of `access-control-vulnerabilities.xlsx`, together with provenance and explicit links to study rows. The original `model` field describes permission-checking architecture and belongs here.

| Field | Type and meaning |
| --- | --- |
| `annotation_id` | Unique authorization-row identifier, also used by `Legacy analysis`. |
| `case_id` | A study-row identifier only when the mapping establishes a unique eligible match. Blank for ambiguous or unresolved mappings. |
| `study_system` | Original authorization worksheet's study grouping and spelling. This may differ from the spelling in `Vulnerabilities`; use the explicit identifiers for joins. |
| `ID` | Original authorization-record identifier, preserved verbatim and not unique. |
| `API` | Original API or operation annotation. A method name without its path/signature is not a complete independent code location. |
| `root cause` | Original authorization classification. It may differ from the corpus or reported evaluation. |
| `impact` | Original impact annotation. |
| `metadata` | Original security-relevant variable or metadata annotation. It is not an executable source/sink or sanitizer configuration. |
| `model` | Original permission-checking mechanism label, M1–M4, as defined below. |
| `cross-component` | Original annotation of whether the relevant behavior crosses process components. Values and blanks are retained without reclassification. |
| `URL` | Original issue/advisory URL. |
| `patch` | Original authorization-workbook patch/reference value. It is preserved separately from the enriched corpus's corrected patch reference. |
| `candidate_case_ids` | Possible study-row identifiers, including the selected row when one is assigned. This list does not itself select an individual row. |
| `mapping_status` | How the authorization annotation was associated with the corpus. It is distinct from commit `verification_status`. |
| `mapping_notes` | The evidence and restrictions behind the row mapping, including missing API information and unresolved identities. |
| `original_sheet` | Worksheet of the original authorization workbook. |
| `original_row` | One-based row in that original worksheet. |

The manuscript defines M1–M4 in its Permission Checking Mechanisms section:

| Code | Mechanism |
| --- | --- |
| `M1` | The server node receiving the request performs its permission check locally before responding. |
| `M2` | The receiving server asks a separate authorization node to check permission and uses its accept/deny response. |
| `M3` | The user obtains a token containing authentication and authorization information; the receiving server checks the token and user. |
| `M4` | A centralized authorization node checks the request and, if permitted, forwards it to the server, then relays the response. |

These labels describe mechanisms, not analyzer versions, models, configurations, or run results. Preserve compound or otherwise original values as supplied. Process granularity is the authors' component definition; it does not by itself validate every historical count or cross-component label.

### Authorization mapping statuses

Only an exact original ID and exactly one candidate with an equal, nonmissing `source`/`API` value permit a `case_id` assignment. Equality trims surrounding whitespace only. It does not infer method aliases, signatures, overloads, namespace equivalence, or independent code sites. All same-ID candidates remain visible even when one is selected.

| Value | Rows | Interpretation |
| --- | ---: | --- |
| `exact_id_unique_api` | 96 | The original ID is identical and exactly one candidate's nonmissing `source` equals the annotation's `API`. `case_id` is assigned. |
| `exact_id_ambiguous` | 11 | Multiple rows share the exact ID and source/API information does not select exactly one. `case_id` is blank. |
| `exact_id_api_unresolved` | 81 | One row shares the exact ID, but source/API equivalence is missing or unverified. It remains a candidate and `case_id` is blank. |
| `alias_candidate_only` | 0 | No exact ID matches; candidates come only from explicitly documented case/punctuation or CVE-token normalization. `case_id` is blank. Reserved by the mapping procedure; not observed in this release. |
| `unmatched` | 0 | Neither exact ID nor the documented alias rule yields a candidate. `case_id` is blank. Reserved by the mapping procedure; not observed in this release. |

## `dctaint_analysis.xlsx`: `Legacy analysis`

| Field | Type and meaning |
| --- | --- |
| `annotation_id` | Foreign key to the corresponding authorization annotation. |
| `logged` | Original logging-related annotation from the legacy workbook. |
| `logged metadata` | Original annotation about metadata logging. |
| `detected` | Original detection-related label. |

These three value fields retain their original values, including blanks and `-`. The historical workbook does not fully document the instrumentation, workloads, runtime environment, analyzer version, or decision procedure underlying these entries. They must not be relabeled as measured execution results. This is particularly important for non-Java systems, which are outside the paper's seven-system Java evaluation. Use `annotation_id` to retrieve the original API, study grouping, and worksheet provenance. This release neither infers missing analyzer configurations nor reruns these cases.

## `dctaint_analysis.xlsx`: `Reported evaluation`

| Field | Type and meaning |
| --- | --- |
| `group_id` | Unique stable identifier for a reported paper identifier/category/outcome group. |
| `system` | System name used by the paper's seven-system evaluation. |
| `reported_id` | Identifier exactly as reported in the paper table. |
| `category` | Reported category: `input_validation`, `information_flow`, or `MPC`. This does not overwrite the corpus category. |
| `reported_detection` | `detected` or `not_detected`, transcribed from the paper. |
| `bug_count` | Number of instances represented by the group, as reported in the paper. |
| `candidate_case_ids` | Study records linked to the group by the documented identifier mapping. Membership does not allocate an individual detection outcome. |
| `mapping_status` | Group-level match/ambiguity classification; see mapping rules below. |
| `notes` | Identifier-resolution details, count/label disagreements, and limits on assigning group outcomes to individual corpus rows. |
| `paper_source` | Manuscript table reference and the reported identifier used for provenance. |
| `evaluated_commit` | Actual revision used in the historical experiment, if documented. All cells are blank in this release because the inspected inputs do not establish it; the corpus's current vulnerable SHA is not substituted. |

The 68 groups total 96 reported instances: 57 MPC, 12 input-validation, and 27 information-flow instances; 87 reported detections and 9 reported misses. The counts are transcribed from the manuscript, not produced by running the newly anchored versions. The mapping finds 95 distinct directly associated corpus rows and one additional Kafka candidate. A match by ID does not establish the actual evaluated commit, independent file/signature/line location, or a complete per-instance outcome assignment.

### Mapping rules and ambiguities

Only documented identifier aliases and source evidence may be used to establish a join. A field with multiple possible matches remains ambiguous. Do not match repeated rows by their order. Do not fill a missing `case_id` merely because one candidate seems plausible.

Paper prefixes CS/CLOUDSTACK, MR/MAPREDUCE, and ZK/ZOOKEEPER are recognized aliases. The unprefixed paper identifier CVE-2014-0229 is linked through its unique CVE suffix in the supplied corpus. KAFKA-5868 versus KAFKA-PULL-5868 remains a candidate because issue and pull-request identifiers occupy different namespaces.

Both HBASE-19400 outcome groups retain the same ten candidate corpus rows. Their `bug_count` values describe the reported group counts; they do not assign detected/not-detected labels to individual members. Repeated YARN-8455 method names do not establish independent code sites. Conflicts between HBase legacy detection annotations and paper outcomes, and between HDFS original classifications and paper categories, remain in `notes` instead of being overwritten.

| Evaluation mapping status | Groups | Interpretation |
| --- | ---: | --- |
| `group_matched` | 60 | The reported identifier resolves to the candidate study rows under the documented identifier rules, with no additional mapping conflict flagged. This remains a group association. |
| `candidate_identifier_alias` | 1 | The Kafka issue/PR namespace mismatch remains a candidate rather than a confirmed identifier equivalence. |
| `group_matched_nonunique_api_annotation` | 1 | The identifier maps, but repeated API annotations prevent independent per-instance code-location attribution. |
| `group_matched_legacy_outcome_conflict` | 2 | The reported outcome conflicts with historical detection annotations. Both sources are retained in their respective sheets. |
| `group_membership_known_individual_outcomes_unresolved` | 2 | Candidate membership is known, but individual members cannot be allocated to the paper's detected/missed groups. Used for the two HBASE-19400 groups. |
| `group_matched_classification_conflict` | 2 | The reported category differs from the original corpus/authorization classification; the release preserves both. |

## Known issues and source-anchor limits

**ZOOKEEPER-2642.** The original row labels it as an authorization issue, but its linked patch and final commits repair client API compatibility rather than server authorization. The related ZOOKEEPER-2014 security fix is not silently substituted. Both SHA cells remain blank, and the original identity remains available for correction with additional evidence. Exclude this unresolved record from confirmed-positive benchmark evaluations until its identity is corrected.

**CVE-2020-8554.** The Kubernetes record has an affected-version anchor. Optional admission and networking mitigations were found, but no fixed SHA meeting the audit's documented default-complete criterion was established as of the audit date. `fixed_commit` remains blank. This is not a claim that no mitigation code exists. Future upstream milestones must not be treated as already released fixes.

**CVE-2014-0085.** The current CVE/vendor attribution is JBoss Fuse/Fabric application password storage, not a generic ZooKeeper defect. The original `ZOOKEEPER-CVE-2014-0085` ID and study grouping remain for provenance. `repository_url` points to Fabric, and the SHA pair maps official Fuse 6.0.0 and 6.1.0 sources to different maintenance histories. The affected source subtree is byte-identical to its Git snapshot. The fixed snapshot matches all Java and non-build source/resources, with documented POM/dependency/SAP manifest productization differences. The vendor advisory establishes the fixed-release designation. The observed ZKENC/Base64 behavior is encoding/obfuscation, not cryptographic encryption. The original generic ZooKeeper reproduction has not been validated as this Fuse-specific CVE.

**Private-source release anchors.** The MongoDB Enterprise CVE-2024-6384 pair comes from official affected/fixed release manifest hashes. It does not provide a publicly inspectable Enterprise patch or public ancestry proof. Public export hashes must not be substituted for the recorded private-source identity.

**Configuration and mitigation scope.** Some rows cover dependency upgrades, downstream packaging changes, optional protections, limited mitigations, later bypasses, or inferred CVE attribution. Consumers must filter on these status qualifications for their intended experiment. SHA presence alone is not a sufficient eligibility rule for an automated benchmark.

## Availability and reproduction boundary

Original workbooks remain archival inputs at their existing paths. The canonical workbooks provide separated data and traceability. They preserve unresolved labels and duplicates rather than claiming a newly deduplicated population.

Analyzer configuration files and runtime logs are not included in this package. Source references and API/metadata annotations are not substitutes for operational analyzer settings. Build commands, deployment settings, workloads, executed PoCs, and per-run results are not established for all 497 records by this schema or by commit enrichment. Reusing a case may require additional implementation and environment work. This documentation change is not a new experiment and does not establish that every retained vulnerability is reproducible.
