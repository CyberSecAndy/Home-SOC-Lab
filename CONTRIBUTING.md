# Contributing to Home SOC Lab

Thank you for your interest in contributing! This document provides guidelines for contributing.

## How to Contribute

### 1. Report Issues

If you find a bug or have a suggestion:

1. Check existing issues first
2. Create a new issue with:
   - Clear title and description
   - Steps to reproduce (for bugs)
   - Expected vs actual behavior
   - Your environment (OS, VirtualBox version, etc.)

### 2. Submit Attack Scenarios

New attack scenarios are welcome!

**Requirements**:
- MITRE ATT&CK technique mapping
- Step-by-step instructions
- Expected logs/events
- Detection rules (Sigma, Splunk, Elastic)
- Screenshots of execution and logs
- Difficulty level (Beginner/Intermediate/Advanced)

### 3. Add Detection Rules

Help improve detection coverage!

**Requirements**:
- Rule title and description
- MITRE ATT&CK mapping
- Multiple formats:
  - Sigma (universal)
  - Splunk SPL
  - Elastic KQL
- False positive analysis
- Example detection screenshots

### 4. Improve Documentation

- Fix typos or unclear explanations
- Add clarifications
- Improve troubleshooting guides
- Add new tips or best practices

## Submission Process

1. **Fork** the repository
2. **Create a branch**: `git checkout -b feature/your-feature`
3. **Make changes** following guidelines below
4. **Test thoroughly**
5. **Commit** with clear messages
6. **Push** to your fork
7. **Create Pull Request** with description

## Guidelines

### Code/Script Quality
- Comment your code
- Use clear variable names
- Follow PEP 8 (Python)
- Test before submitting
- Include error handling

### Documentation
- Use clear, professional language
- Include examples
- Add images/screenshots where helpful
- Link to relevant resources
- Keep formatting consistent

### Attack Scenarios
- Test end-to-end
- Include all necessary steps
- Document expected output
- Note any dependencies
- Include troubleshooting section

### Detection Rules
- Test against actual logs
- Include false positive analysis
- Add example detections
- Document tuning parameters
- Include performance considerations

## Code of Conduct

- Be respectful and professional
- Provide constructive feedback
- Help others learn
- Focus on improving the project

## Questions?

Create an issue or discussion for questions. I'm happy to help!

Thank you for contributing! 🙏