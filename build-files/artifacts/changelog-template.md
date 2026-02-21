# Changelog Template (User Story)

This template is used by py-developer to generate `CHANGELOG-US-XXX.md` in each user story folder during Step J.

---

## Template Structure

```markdown
# Changelog: US-XXX [Story Title]

## [Unreleased] - YYYY-MM-DD

### Added
- New feature or capability added
- Files created:
  - `path/to/file.py` - Purpose/description
  - `path/to/test.py` - Test suite description

### Changed
- Modifications to existing functionality
- Files modified:
  - `path/to/file.py` - Changes made

### Fixed
- Bug fixes or corrections
- Issues resolved

### Tests
- Unit tests: X tests (X% coverage)
- Integration tests: Y tests
- E2E tests: Z tests
- Live verification: N tests

### Documentation
- Documentation files created/updated
```

---

## Usage

py-developer generates this changelog during **Step J** with:
- All changes made in the specific user story
- Files created and modified with descriptions
- Test statistics (count and coverage)
- Following Keep a Changelog format

This user story changelog is then summarized into a single line entry in `docs/releases/CHANGELOG.md`.
