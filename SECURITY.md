# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 2.0.x   | :white_check_mark: |
| 1.0.x   | :x:                |

## Security Considerations

Claudarity is designed with security and privacy in mind:

### Local-First Architecture

- **All data stays local**: No cloud sync, no external servers
- **User-only access**: Database and files have 600 permissions
- **No telemetry**: No data collection or tracking
- **No external dependencies**: Core functionality works offline

### Data Protection

- **SQLite database**: Stored locally at `~/.claude/claudarity.db`
- **Markdown cache**: Stored locally (configurable location)
- **Logs**: Stored locally at `~/.claude/logs/`
- **No credentials stored**: Uses system keychain for API keys

### Known Security Considerations

1. **Hardcoded Paths**: Some scripts have hardcoded paths (being fixed in v3.0.0)
2. **Background Processes**: Hooks run in background with user permissions
3. **SQL Injection**: Parameterized queries used, but review recommended
4. **File Permissions**: Ensure proper permissions on database and config files

## Reporting a Vulnerability

We take security seriously. If you discover a security vulnerability, please follow these steps:

### 1. Do Not Open a Public Issue

Please **do not** open a public GitHub issue for security vulnerabilities.

### 2. Report Privately

Report security vulnerabilities by:

1. **GitHub Security Advisory** (preferred):
   - Go to the Security tab
   - Click "Report a vulnerability"
   - Fill out the form

2. **Email** (alternative):
   - Contact the maintainer directly
   - Include "SECURITY" in the subject line
   - Provide detailed information (see below)

### 3. What to Include

Please include the following information:

- **Description**: Clear description of the vulnerability
- **Impact**: What could an attacker do?
- **Reproduction**: Step-by-step instructions to reproduce
- **Environment**: OS, shell, versions
- **Proof of Concept**: Code or commands demonstrating the issue
- **Suggested Fix**: If you have ideas on how to fix it

### 4. Response Timeline

- **Initial Response**: Within 48 hours
- **Status Update**: Within 7 days
- **Fix Timeline**: Depends on severity
  - Critical: Within 7 days
  - High: Within 30 days
  - Medium: Within 90 days
  - Low: Next release

### 5. Disclosure Policy

- We will work with you to understand and fix the issue
- We will credit you in the security advisory (unless you prefer anonymity)
- We will coordinate disclosure timing with you
- We will publish a security advisory after the fix is released

## Security Best Practices for Users

### Installation

1. **Verify Source**: Clone from official repository only
2. **Check Permissions**: Ensure scripts are not world-writable
3. **Review Code**: Review scripts before running (especially hooks)
4. **Use Templates**: Start with template configs, don't copy from untrusted sources

### Configuration

1. **File Permissions**: Set proper permissions on sensitive files
   ```bash
   chmod 600 ~/.claude/claudarity.db
   chmod 600 ~/.claude/config/settings.json
   chmod 700 ~/.claude/
   ```

2. **API Keys**: Use environment variables or system keychain
   ```bash
   # Don't hardcode in scripts
   export ANTHROPIC_API_KEY="your-key"
   ```

3. **Backup Encryption**: Encrypt backups if storing off-machine
   ```bash
   # Example: Encrypt database backup
   gpg -c ~/.claude/claudarity.db
   ```

### Usage

1. **Review Feedback**: Check what feedback is being logged
2. **Sanitize Logs**: Remove sensitive data before sharing logs
3. **Regular Updates**: Keep Claudarity updated
4. **Monitor Processes**: Check for unexpected background processes

### Database Security

1. **Access Control**: Database should only be readable by your user
2. **Backup Security**: Encrypt backups containing sensitive data
3. **No Sensitive Data**: Avoid storing passwords or secrets in feedback

### Script Security

1. **Review Hooks**: Understand what hooks do before enabling
2. **Validate Input**: Be cautious with user input in custom scripts
3. **Path Traversal**: Validate file paths in custom scripts
4. **Command Injection**: Avoid eval and unquoted variables

## Security Audit Checklist

If you're auditing Claudarity for security:

### Code Review

- [ ] Check for SQL injection vulnerabilities
- [ ] Check for command injection vulnerabilities
- [ ] Check for path traversal vulnerabilities
- [ ] Review file permission handling
- [ ] Review input validation
- [ ] Check for hardcoded credentials
- [ ] Review error handling (information disclosure)

### Configuration Review

- [ ] Check file permissions on database
- [ ] Check file permissions on config files
- [ ] Check file permissions on logs
- [ ] Verify no sensitive data in git
- [ ] Check .gitignore coverage

### Runtime Review

- [ ] Monitor background processes
- [ ] Check network connections (should be none for core features)
- [ ] Review log files for sensitive data
- [ ] Check database for sensitive data
- [ ] Verify proper cleanup of temporary files

## Known Issues

See [CLEANUP_GUIDE.md](docs/CLEANUP_GUIDE.md) for known issues and planned fixes.

### Current Known Issues

1. **Hardcoded Paths**: Some scripts have hardcoded paths (v3.0.0 will fix)
2. **Limited Input Validation**: Some scripts need better input validation
3. **Background Process Errors**: Can fail silently (improving in v3.0.0)

## Security Updates

Security updates will be:
- Published in CHANGELOG.md
- Announced in GitHub releases
- Tagged with "security" label
- Documented in security advisories

## Contact

For security concerns:
- GitHub Security Advisory (preferred)
- GitHub Issues (for non-sensitive security improvements)

## Acknowledgments

We appreciate security researchers who responsibly disclose vulnerabilities.

Security contributors will be acknowledged in:
- Security advisory
- CHANGELOG.md
- GitHub release notes

Thank you for helping keep Claudarity secure!
