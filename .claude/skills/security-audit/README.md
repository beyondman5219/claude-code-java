# Security Audit

**Load**: `view .claude/skills/security-audit/SKILL.md`

---

## 描述

基于 OWASP Top 10 和安全编码实践的 Java 安全清单。框架无关的核心，包含 Spring、Quarkus 和 Jakarta EE 的特定部分。

---

## 使用场景

- "Review this code for security issues"
- "Check for SQL injection vulnerabilities"
- "Is this authentication secure?"
- "Security audit before release"
- "OWASP compliance check"

---

## 涵盖主题

| Topic | 适用于 |
|-------|------------|
| **Input Validation** | 所有 Java（Bean Validation JSR 380） |
| **SQL Injection** | JPA、Hibernate、JDBC |
| **XSS Prevention** | Web 应用程序 |
| **CSRF Protection** | Spring、Quarkus |
| **Authentication** | 所有框架 |
| **Secrets Management** | 所有应用程序 |
| **Secure Deserialization** | 所有 Java |
| **Dependency Security** | Maven、Gradle |
| **Security Headers** | Web 应用程序 |

---

## OWASP Top 10 覆盖

| Risk | 已覆盖 |
|------|---------|
| A01 Broken Access Control | ✅ |
| A02 Cryptographic Failures | ✅ |
| A03 Injection | ✅ |
| A04 Insecure Design | ✅ |
| A05 Security Misconfiguration | ✅ |
| A06 Vulnerable Components | ✅ |
| A07 Authentication Failures | ✅ |
| A08 Data Integrity Failures | ✅ |
| A09 Logging Failures | ✅ |
| A10 SSRF | ✅ |

---

## 相关技能

- `java-code-review` - 通用审查
- `maven-dependency-audit` - 依赖扫描
- `logging-patterns` - 安全日志

---

## 参考资料

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Java Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Java_Security_Cheat_Sheet.html)
- [Spring Boot Security Best Practices (Snyk)](https://snyk.io/blog/spring-boot-security-best-practices/)
- [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/)
