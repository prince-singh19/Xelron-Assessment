# Part 3: Prompt Preparation

### 3.1.1 Repository Context
Beets is an open-source music library manager written in Python, designed for users who maintain large and carefully organized digital music collections. Its main purpose is to help users clean, standardize, and manage metadata across thousands of audio files. Beets works primarily as a command-line tool and focuses on importing music files, reading their metadata, matching them with authoritative databases such as MusicBrainz, and storing the results in a structured SQLite database. Along with metadata management, Beets also supports file renaming, directory organization, and powerful querying features.

The intended users of Beets are music collectors, archivists, and technically inclined users who need more control than what traditional media players offer. These users often deal with music files collected from many different sources, which can result in inconsistent naming, missing tags, or duplicate entries. Beets addresses this problem domain by enforcing consistency and providing automation during the import process.

A key challenge in music library management is identifying duplicate tracks or albums correctly. Files may share similar names but represent different releases, editions, or versions. Beets handles this by comparing metadata during import and deciding whether a file should be skipped, merged, or added. Its modular architecture allows core logic to remain stable while offering extensibility through plugins and configuration options, making it suitable for both simple and advanced use cases.

### 3.1.2 Pull Request Description
Pull Request #4199 focuses on improving how Beets detects duplicate albums and tracks during the import process. Before this change, duplicate detection relied on fixed metadata fields. Albums were considered duplicates if both `albumartist` and `album` matched, while individual tracks were compared using `artist` and `title`. Although this worked for basic libraries, it caused incorrect duplicate detection for users with more complex collections, such as multiple releases of the same album or tracks with identical titles but different versions.

This PR introduces a new configuration option called `duplicate_keys`, which allows users to define which metadata fields should be used when identifying duplicates. Separate keys can be configured for albums and items, giving users fine-grained control over the import behavior.

The previous behavior was rigid and could not be adapted without modifying the source code. With this PR, duplicate detection becomes configuration-driven. Internally, the importer now builds compound database queries using the specified fields instead of relying on hardcoded comparisons. This change is necessary to reduce false positives during import and to support advanced library organization strategies without breaking existing workflows.

### 3.1.3 Acceptance Criteria
* ✓ When importing an album, the system should identify duplicates based only on the fields defined in `import.duplicate_keys.album`.
* ✓ When importing a single track, duplicate detection should use the fields defined in `import.duplicate_keys.item`.
* ✓ If users do not modify the configuration, the system should behave exactly as before.
* ✓ When any configured field differs, the importer should treat the album or item as a new entry.
* ✓ The implementation should correctly handle computed fields by generating temporary `Album` or `Item` objects before querying.
* ✓ Unit tests should confirm that changing `duplicate_keys` alters duplicate detection behavior as expected.

### 3.1.4 Edge Cases
* **Case 1: Missing Metadata Fields**  
  If a configured duplicate key is missing from an imported file, the system should handle it gracefully without crashing or misidentifying duplicates.

* **Case 2: Overly Broad Keys**  
  If users configure very few fields (e.g., only `album`), the importer should still function correctly, even if it results in more duplicates being detected.

* **Case 3: Computed or Derived Fields**  
  The system should correctly evaluate fields that are computed internally, ensuring consistency between imported data and existing library entries.

### 3.1.5 Initial Prompt
"You are a Python developer contributing to the Beets music library manager. Your task is to improve the duplicate detection mechanism used during the import process by making it configurable through user-defined metadata fields.

**Repository Context:**  
Beets imports music files, reads their metadata, and stores them in an SQLite database. During import, the system checks whether an album or track already exists in the library and resolves duplicates based on metadata comparison.

**Objective:**  
Implement support for configurable duplicate detection using a new configuration option called `duplicate_keys`. This option should allow users to define which metadata fields are used to identify duplicate albums and items.

**Tasks to Implement:**  
1. Add a `duplicate_keys` configuration entry under the import section, with separate values for albums and items.
2. Update the importer logic so that duplicate detection uses these configured fields instead of hardcoded ones.
3. Introduce a helper method that builds a compound database query using all configured fields.
4. Ensure temporary `Album` and `Item` objects are used to compute derived fields before querying.
5. Preserve existing behavior when users do not modify the configuration.

**Acceptance Criteria:**  
The implementation must satisfy all acceptance criteria listed above, including backward compatibility and correct handling of missing fields.

**Edge Cases:**  
Pay special attention to missing metadata, minimal key configurations, and computed fields.

**Testing Requirements:**  
Update or add unit tests to confirm that:
- Default behavior remains unchanged.
- Custom `duplicate_keys` settings alter duplicate detection.
- No false positives occur when metadata differs.

Ensure the solution is clean, readable, and consistent with Beets’ existing importer architecture."

---
### Integrity Declaration
I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words.
