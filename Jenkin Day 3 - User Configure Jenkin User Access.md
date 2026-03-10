# Jenkins Configuration Study Material
### User Management, Security & Authorization

---

## 1. Creating a Jenkins User

**Path:** Manage Jenkins → Users → Create User

| Field | Description |
|-------|-------------|
| Username | Login name (e.g. `anita`) |
| Password | User's password |
| Full Name | Display name (e.g. `Anita`) |
| Email | Optional |

- Users are stored in Jenkins' own database by default
- User config files live at: `/var/lib/jenkins/users/<username>/config.xml`

---

## 2. Security Realm vs Authorization Strategy

These are two separate settings under **Manage Jenkins → Security**

| Setting | What it controls |
|---------|-----------------|
| **Security Realm** | *Who can log in?* (Authentication) |
| **Authorization Strategy** | *What can they do?* (Permissions) |

**Security Realm options:**
- Jenkins' own user database ← most common for labs
- LDAP
- GitHub OAuth
- Active Directory

---

## 3. Authorization Strategies

**Path:** Manage Jenkins → Security → Authorization

| Strategy | Use Case |
|----------|----------|
| Anyone can do anything | Dev/test only — no security |
| Logged-in users can do anything | Simple setups |
| Matrix-based security | Global fine-grained permissions |
| **Project-based Matrix Authorization** | Global + per-job permissions ← what you used |

---

## 4. Project-based Matrix Authorization Strategy

This is the most flexible strategy. Permissions are set at **two levels**:

1. **Global level** — applies across all of Jenkins
2. **Job level** — applies to individual jobs only

### Installing the Plugin First
**Manage Jenkins → Plugins → Available plugins → Search: "Matrix Authorization Strategy" → Install**

After install: select **"Restart Jenkins when installation is complete and no jobs are running"** — wait for login page before proceeding.

---

## 5. Global Permission Matrix

**Path:** Manage Jenkins → Security → Authorization → Project-based Matrix Authorization Strategy

### Common Permission Categories

| Category | Permission | Meaning |
|----------|-----------|---------|
| Overall | **Administer** | Full control of Jenkins |
| Overall | **Read** | Can log in and view Jenkins |
| Job | **Read** | Can see the job |
| Job | **Build** | Can trigger builds |
| Job | **Configure** | Can change job settings |
| Job | **Delete** | Can delete the job |
| Agent | **Build** | Can run builds on agents |
| SCM | **Tag** | Can tag in source control |

### Typical Setup (What You Configured)

| User | Overall: Administer | Overall: Read | Notes |
|------|-------------------|--------------|-------|
| admin | ✅ | ✅ (auto) | Full control |
| anita | ❌ | ✅ | Read-only access |
| Anonymous | ❌ | ❌ | No access |

> ⚠️ Always keep at least one admin with **Administer** before saving — otherwise you lock yourself out!

---

## 6. Job-level Permissions

**Path:** Job → Configure → Enable project-based security → Add user

This overrides or extends global permissions for that specific job.

### What You Configured for anita

| Permission | Granted? |
|-----------|---------|
| Job: Read | ✅ Yes |
| Job: Build | ❌ No |
| Job: Configure | ❌ No |
| Job: Delete | ❌ No |
| Agent: (all) | ❌ No |
| SCM: (all) | ❌ No |
| Run: (all) | ❌ No |

---

## 7. Anonymous User

- **Anonymous** = any visitor who is NOT logged in
- In production, always remove all Anonymous permissions
- This forces users to log in before seeing anything in Jenkins

---

## 8. Summary — What You Did Step by Step

```
1. Logged in as admin (admin / Adm!n321)
       ↓
2. Installed "Matrix Authorization Strategy" plugin
   → Restarted Jenkins
       ↓
3. Created user: anita
   → Password: 8FmzjvFU6S
   → Full Name: Anita
       ↓
4. Set Authorization Strategy:
   → Project-based Matrix Authorization Strategy
   → admin     = Overall: Administer
   → anita     = Overall: Read
   → anonymous = no permissions
       ↓
5. Opened existing job → Configure
   → Enabled project-based security
   → Added anita with Job: Read only
```

---

## 9. Key Takeaways

- **Matrix Authorization** gives you full control over who can do what
- **Project-based** means you can set different permissions per job
- **Anonymous** users should always be locked down in real environments
- Always test by logging in as the new user to verify permissions work correctly
- The **admin** must always retain **Administer** — never uncheck it before saving

---

*Jenkins Configuration Notes | Nautilus Team*
