# Changelog

All notable changes to this project will be documented in this file.

**Version Number Format:** `major.minor.patch`  
**Release Date Format:** `yyyy-mm-dd`  

**Types of Changes:**
- **Added** for new features.
- **Changed** for changes in existing functionality.
- **Deprecated** for soon-to-be removed features.
- **Removed** for now removed features.
- **Fixed** for any bug fixes.
- **Security** in case of vulnerabilities.
##


## [Unreleased]

`-`


## [0.4.0] - 2020-11-09

### Added
- Added support for text and code descriptions for `generate_config_info` command using `--text_description` and `--code_description` options.


## [0.3.0] - 2020-11-06

### Added
- Added support for printing entire nodes instead of just tags for `extract_tag` command with `-n` and `--nodes_print_command` options.  
- Added support to add `tasker_config_utils` script signature at end of output file for `generate_config_info` command with the `-s` option.  

### Changed
- Changed `generate_config_info` default behaviour of for task help anchor labels.The labels must be in markdown from now on and will not be put in a code block unless `-c` is passed.  

### Removed
- Removed separate `Comments` and `Control` section from output of `generate_config_info` command. The help anchor labels should preferably format any sections themselves in markdown.  


## [0.2.2] - 2020-09-10

### Added
- Added separate `Control` section to output of `generate_config_info` command.  


## [0.2.1] - 2020-05-20

### Changed
- Changed timestamp to UTC for `generate_config_info` command.  


## [0.2.0] - 2020-04-13

### Added
- Added sha256sum to output of `generate_config_info` command.  


## [0.1.0] - 2020-01-13

`-`
##


[unreleased]: https://github.com/Taskomater/tasker_config_utils/compare/v0.4.0...HEAD
[0.3.0]: https://github.com/Taskomater/tasker_config_utils/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/Taskomater/tasker_config_utils/compare/v0.2.2...v0.3.0
[0.2.2]: https://github.com/Taskomater/tasker_config_utils/compare/v0.2.1...v0.2.2
[0.2.1]: https://github.com/Taskomater/tasker_config_utils/compare/v0.2.0...v0.2.1
[0.2.0]: https://github.com/Taskomater/tasker_config_utils/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/Taskomater/tasker_config_utils/releases/tag/v0.1.0
