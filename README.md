# DevOps Interview Prep – Q&A Worksheet  
**Date:** October 28, 2025  
**Focus:** Linux, Git, GitHub, Maven, Shell Scripting, Jenkins  

---

## 1. **Linux: Package Managers**
**Q:** What’s the difference between `apt` and `yum`?  
**Your Answer:**  
- `apt` → Debian-based (Ubuntu)  
- `yum` → Red Hat-based (CentOS, Amazon Linux)  
- Used to install apps from remote repo to local machine  

**Correction / Key Points:**  
- Correct!  
- `apt` (Debian/Ubuntu): `sudo apt update && sudo apt install nginx`  
- `yum` (RHEL/CentOS): `sudo yum install httpd`  
- **Note:** `dnf` replaced `yum` in newer Fedora/RHEL 8+  
- Both handle **dependency resolution** automatically  

---

## 2. **Git: Branch Management & Merge vs Rebase**
**Q:** How do you handle branch management in a team to avoid conflicts?  
**Your Answer:**  
- Create feature branch: `feature/login`  
- Work → commit → when done, merge into `main`  
- Use `git pull`, `git merge`, or `git rebase`  
- Rebase → linear history, merge → merge commit  

**Correction / Key Points:**  
- Good!  
- **Best Practice:**  
  - Rebase **locally** before PR → clean history  
  - **Never rebase public/shared branches**  
  - Use `git pull --rebase` to stay updated  
- Conflict resolution: `git status` → edit files → `git add` → continue  
- **Merge**: keeps full history  
- **Rebase**: rewrites commits → cleaner graph  

---

## 3. **Jenkins: CI/CD Pipeline with GitHub Webhook**
**Q:** Configure Jenkins to run tests on GitHub push via webhook, handle failures.  
**Your Answer:**  
- GitHub webhook → HTTP POST to Jenkins  
- Jenkins clones repo → builds with Maven (`pom.xml`)  
- Runs tests (Selenium) → deploys if pass  
- Master-slave setup  

**Correction / Key Points:**  
- Solid flow! Add:  
  - Webhook URL: `http://jenkins:8080/github-webhook/`  
  - Pipeline stages:  
    ```groovy
    pipeline {
      stages {
        stage('Build') { steps { sh 'mvn clean install' } }
        stage('Test') { steps { sh 'mvn test' } }
      }
      post {
        failure { emailext subject: 'Build Failed', body: 'Check Jenkins', to: 'team@company.com' }
      }
    }
    ```
  - Retry: `retry(3) { sh 'mvn test' }`  
  - Poll SCM or webhook → both work  

---

## 4. **Shell Script: Backup Jenkins Logs >7 Days**
**Q:** Write a secure shell script to archive Jenkins logs older than 7 days, log actions, no root.  
**Your Answer:**  
- Use `find`, `tar`, `xargs`, timestamp  
- Check disk space first  

**Final Script (Corrected):**  
```bash
#!/bin/bash
LOG_DIR="/var/lib/jenkins/logs"
BACKUP_DIR="/backup"
LOCK="/tmp/jenkins_backup.lock"
EMAIL="narendra@team.com"

# Prevent overlap
exec 200>$LOCK
flock -n 200 || { echo "Backup already running" | mail -s "Skipped" $EMAIL; exit 1; }

# Check disk
USED=$(df /var | awk 'NR==2 {print $5}' | tr -d '%')
if [ $USED -gt 90 ]; then
  echo "Disk full ($USED%)" | mail -s "CRITICAL: Disk Full" $EMAIL
  systemctl stop jenkins
  exit 1
fi

# Archive old logs
find $LOG_DIR -type f -mtime +7 -print0 | xargs -0 tar -czf $BACKUP_DIR/jenkins_logs_$(date +%Y%m%d_%H%M%S).tar.gz

# Notify
echo "Backup completed: $(date)" | mail -s "Jenkins Logs Archived" $EMAIL
```

**Cron (with lock):**  
```bash
0 2 * * * flock -n /tmp/jenkins_backup.lock /bin/bash /scripts/backup_logs.sh
```

---

## 5. **Git: Recover Branch After Force-Push**
**Q:** Teammate force-pushed. Your branch is broken. Recover without losing work.  
**Your Answer:**  
- Use `git reflog` → find old commit SHA  
- `git checkout -b recovered-branch <SHA>`  
- Rebase on `origin/main` → push with `--force-with-lease`  

**Correct Flow:**  
```bash
git reflog
# Find: abc1234 HEAD@{3}: commit: fixed login
git checkout abc1234
git checkout -b login-fix-v2
git fetch origin
git rebase origin/main
git push --force-with-lease origin login-fix-v2
```

**Key:** `reflog` is **local** — your safety net!  

---

## 6. **Git: Rename Branch (Local + Remote)**
**Q:** Rename branch without breaking team.  
**Your Answer:**  
- `git branch -m old-name new-name`  
- `git push origin :old-name` → delete remote  
- `git push -u origin new-name`  

**Correct & Safe:**  
```bash
git branch -m feature/login login-v2
git push origin :feature/login
git push -u origin login-v2
```
Teammates: `git fetch --prune && git checkout login-v2`

---

## 7. **Linux: Limit Process Memory (cgroups)**
**Q:** Limit Jenkins build to 2GB RAM using cgroups.  
**Your Answer:** Didn’t know  

**Answer:**  
```bash
systemd-run --scope -p MemoryMax=2G java -jar /opt/jenkins.war
```
Or in Jenkins pipeline (Kubernetes):  
```yaml
resources:
  limits:
    memory: "2Gi"
```

---

## 8. **Git: Remove Secret from History**
**Q:** API key leaked in commit. Remove from history **without force-push**.  
**Answer:** Use `git filter-repo` (recommended)  

```bash
# Install: pip install git-filter-repo
echo 'api_key=***REMOVED***' > secrets.txt
git filter-repo --replace-text secrets.txt --force
git push --force-with-lease
```
**Note:** Revoke key **first**!  

---

## 9. **Git: Create & Publish Release Tag**
**Q:** Tag a release and show it on GitHub Releases.  
**Answer:**  
```bash
git tag -a v1.2.0 -m "Production release"
git push origin v1.2.0
```
Then on GitHub:  
**Releases → Draft a new release → Choose tag → Publish**

Or with GitHub CLI:  
```bash
gh release create v1.2.0 --notes "Bug fixes & login feature"
```

---

## Summary Commands Cheat Sheet
| Task | Command |
|------|---------|
| Rebase local | `git pull --rebase origin main` |
| Safe force push | `git push --force-with-lease` |
| Delete remote branch | `git push origin :old-branch` |
| Disk % used | `df -h /var | awk 'NR==2 {print $5}'` |
| Lock script | `flock -n /tmp/lockfile script.sh` |
| Find old files | `find /path -mtime +7 -type f` |

---

**End of Session**  
**Instructor:** Ara (your not-puppy coach)  
**Next Step:** Quiz yourself tomorrow. Say `start` to resume.  

*Copy this into Markdown → PDF via browser print or [markdowntopdf.com](https://markdowntopdf.com)*
