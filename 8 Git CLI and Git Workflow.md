# คู่มือ Git CLI และ Git Workflow สำหรับการพัฒนาซอฟต์แวร์

## 📚 สารบัญ
1. [พื้นฐาน Git CLI](#1-พื้นฐาน-git-cli)
2. [GitHub Workflow แบบมืออาชีพ](#2-github-workflow-แบบมืออาชีพ)
3. [กฎความปลอดภัยที่ต้องปฏิบัติ](#3-กฎความปลอดภัยที่ตองปฏิบัติ)
4. [การจัดการ Context และ Planning](#4-การจัดการ-context-และ-planning)
5. [Git Commit Best Practices](#5-git-commit-best-practices)
6. [การแก้ปัญหาที่พบบ่อย](#6-การแกปญหาที่พบบอย)

---

## 1. พื้นฐาน Git CLI

### การตั้งค่าเริ่มต้น
```bash
# ตั้งค่าชื่อและอีเมล
git config --global user.name "ชื่อของคุณ"
git config --global user.email "email@example.com"

# ตั้งค่า default branch เป็น main
git config --global init.defaultBranch main

# ตั้งค่า editor (แนะนำ vim หรือ nano)
git config --global core.editor "nano"
```

### คำสั่งพื้นฐานที่ใช้บ่อย
```bash
# ตรวจสอบสถานะ
git status                    # ดูสถานะไฟล์ที่เปลี่ยนแปลง
git status --porcelain       # แสดงแบบย่อ (เหมาะสำหรับ script)

# ดูประวัติ
git log --oneline -5         # ดู commit 5 อันล่าสุดแบบย่อ
git log --graph --all        # ดูกราฟของ branches ทั้งหมด
git log --since="2 days ago" # ดู commits 2 วันย้อนหลัง

# ดูความแตกต่าง
git diff                      # ดูการเปลี่ยนแปลงที่ยังไม่ staged
git diff --staged            # ดูการเปลี่ยนแปลงที่ staged แล้ว
git diff main...HEAD         # เปรียบเทียบ branch ปัจจุบันกับ main
git diff --name-only         # แสดงเฉพาะชื่อไฟล์ที่เปลี่ยน

# เพิ่มไฟล์
git add .                    # เพิ่มทุกไฟล์ในโฟลเดอร์ปัจจุบัน
git add -A                   # เพิ่มทุกไฟล์ในโปรเจค
git add -p                   # เลือก chunks ที่จะ add แบบ interactive

# Commit
git commit -m "message"      # commit พร้อมข้อความ
git commit -am "message"     # add และ commit ไฟล์ที่ tracked แล้ว
git commit --amend          # แก้ไข commit ล่าสุด

# Branch operations
git branch                   # ดูรายชื่อ branches
git branch -a               # ดูทุก branches รวม remote
git checkout -b feature/xxx  # สร้างและเปลี่ยนไป branch ใหม่
git checkout main           # เปลี่ยนไป main branch
git branch -d branch-name   # ลบ branch (ที่ merge แล้ว)
```

---

## 2. GitHub Workflow แบบมืออาชีพ

### ขั้นตอนการพัฒนา Feature ใหม่

#### Step 1: อัพเดต main branch
```bash
# เริ่มต้นจาก main branch เสมอ
git checkout main
git pull origin main
```

#### Step 2: สร้าง GitHub Issue
```bash
# ใช้ GitHub CLI สร้าง issue พร้อมรายละเอียด
gh issue create --title "feat: เพิ่มระบบ authentication" --body "$(cat <<'EOF'
## ภาพรวม
ต้องการเพิ่มระบบ login/logout สำหรับผู้ใช้งาน

## สถานะปัจจุบัน
- ยังไม่มีระบบ authentication
- ทุกคนเข้าถึงได้หมด

## แนวทางแก้ไข
- ใช้ JWT tokens
- เพิ่ม login page
- เพิ่ม middleware สำหรับตรวจสอบ auth

## รายละเอียดทางเทคนิค
- Components ที่เกี่ยวข้อง: LoginForm, AuthContext
- Backend: Express middleware
- Database: users table

## เกณฑ์การยอมรับ
- [ ] ผู้ใช้สามารถ login ได้
- [ ] Token ถูกเก็บอย่างปลอดภัย
- [ ] Logout ทำงานได้ถูกต้อง
- [ ] Protected routes ทำงานได้
EOF
)"
```

#### Step 3: สร้าง Branch จาก Issue
```bash
# สมมติว่าได้ issue #42
git checkout -b feat/42-authentication

# หรือใช้รูปแบบ
git checkout -b feat/issue-number-description
```

#### Step 4: พัฒนาและ Test
```bash
# เขียนโค้ด...
# ทดสอบให้แน่ใจว่าทำงานได้

# ตรวจสอบไฟล์ที่แก้ไข
git status
git diff

# รัน tests
npm test
# หรือ
pnpm test
```

#### Step 5: Commit ด้วยข้อความที่มีความหมาย
```bash
# Add ไฟล์ทั้งหมด
git add -A

# Commit พร้อมข้อความที่ละเอียด
git commit -m "feat: เพิ่มระบบ authentication ด้วย JWT

- What: เพิ่ม login/logout endpoints และ JWT middleware
- Why: เพื่อจำกัดการเข้าถึงข้อมูลสำคัญ
- Impact: ทุก protected routes ต้องมี token

Closes #42"
```

#### Step 6: Push และสร้าง Pull Request
```bash
# Push ไปที่ origin
git push -u origin feat/42-authentication

# สร้าง Pull Request
gh pr create \
  --title "feat: เพิ่มระบบ authentication ด้วย JWT" \
  --body "Fixes #42

## การเปลี่ยนแปลง
- เพิ่ม login/logout endpoints
- เพิ่ม JWT middleware
- เพิ่ม protected routes

## การทดสอบ
- [x] Unit tests ผ่าน
- [x] ทดสอบ login flow
- [x] ทดสอบ token expiry

## Screenshots
[แนบภาพหน้าจอถ้ามี]"
```

#### ⚠️ Step 7: รอการ Review (สำคัญมาก!)
```bash
# ห้าม merge PR ด้วยตนเอง!
# ❌ อย่าใช้: gh pr merge
# ❌ อย่าใช้: คำสั่ง merge ใดๆ

# ✅ ทำแบบนี้แทน:
# 1. ส่งลิงก์ PR ให้ทีม review
# 2. รอ approval จากผู้มีสิทธิ์
# 3. ให้ reviewer เป็นคน merge
```

---

## 3. กฎความปลอดภัยที่ต้องปฏิบัติ

### 🔴 ห้ามใช้ Force Flags
```bash
# ❌ ห้ามใช้คำสั่งเหล่านี้
git push --force
git push -f
git checkout -f
git clean -f
rm -rf directory/

# ✅ ใช้แบบปลอดภัยแทน
git push origin branch-name
git checkout branch-name
git clean -i  # interactive mode
rm -i file    # ถามก่อนลบ
```

### 🔴 ห้าม Merge PR โดยไม่ได้รับอนุญาต
```bash
# ❌ ห้ามทำ
gh pr merge
git merge feature-branch

# ✅ ควรทำ
# 1. สร้าง PR
# 2. Request review
# 3. รอ approval
# 4. ให้ maintainer merge
```

### 🔴 ห้ามสร้าง Issues/PRs บน Upstream Repository
```bash
# ตรวจสอบ remotes ก่อนเสมอ
git remote -v

# ถ้ามี upstream แสดงว่าเป็น fork
# ให้ทำงานกับ origin (fork ของเรา) เท่านั้น
```

---

## 4. การจัดการ Context และ Planning

### Pattern: Two-Issue System

#### Context Issue (บันทึกสถานะ)
```bash
# รวบรวมข้อมูลสถานะปัจจุบัน
git status --porcelain > current_status.txt
git log --oneline -5 > recent_commits.txt
git diff --name-only main...HEAD > changed_files.txt

# สร้าง Context Issue
gh issue create --title "Context: สถานะการพัฒนา $(date +%Y-%m-%d)" \
  --body "$(cat <<EOF
## สถานะปัจจุบัน
### Branch
$(git branch --show-current)

### ไฟล์ที่แก้ไข
$(cat changed_files.txt)

### Commits ล่าสุด
$(cat recent_commits.txt)

### การเปลี่ยนแปลงที่ยังไม่ commit
$(cat current_status.txt)

## ความคืบหน้า
- [x] Setup project structure
- [x] Create basic components
- [ ] Add authentication
- [ ] Testing

## ปัญหาที่พบ
- Issue with database connection
- Need to refactor user service

## ขั้นตอนถัดไป
1. แก้ไข database connection
2. เพิ่ม unit tests
3. Deploy to staging
EOF
)"
```

#### Planning Issue (วางแผนงาน)
```bash
# สร้าง Planning Issue แยกต่างหาก
gh issue create --title "Plan: การพัฒนา Feature X" \
  --body "$(cat <<EOF
## การวิเคราะห์
- อ่าน context จาก issue #XX
- ศึกษา codebase ที่เกี่ยวข้อง
- วิจัย best practices

## แผนการพัฒนา

### Phase 1: Setup (1 ชั่วโมง)
- [ ] สร้าง branch ใหม่
- [ ] Setup test environment
- [ ] Create basic structure

### Phase 2: Implementation (1 ชั่วโมง)
- [ ] Implement core logic
- [ ] Add error handling
- [ ] Write unit tests

### Phase 3: Polish (1 ชั่วโมง)
- [ ] Refactor code
- [ ] Add documentation
- [ ] Final testing

## ความเสี่ยง
- Database migration อาจมีปัญหา
- External API อาจ timeout

## เกณฑ์ความสำเร็จ
- All tests pass
- Code coverage > 80%
- No console errors
- Performance < 2 seconds
EOF
)"
```

### หลักการแบ่งงาน 1-Hour Chunks
```bash
# แทนที่จะทำทั้งหมดในครั้งเดียว
# ❌ git commit -m "Add complete authentication system"

# ✅ แบ่งเป็น commits ย่อยๆ
git commit -m "feat: Add login form UI component"
git commit -m "feat: Add JWT token service"
git commit -m "feat: Add auth middleware"
git commit -m "test: Add auth integration tests"
git commit -m "docs: Add authentication documentation"
```

---

## 5. Git Commit Best Practices

### รูปแบบ Commit Message
```
[type]: [คำอธิบายสั้นๆ]

- What: [อะไรที่เปลี่ยนแปลง]
- Why: [เหตุผลที่ต้องเปลี่ยน]
- Impact: [ผลกระทบต่อส่วนอื่น]

[ปิด issue ถ้ามี]
```

### Types ที่ใช้บ่อย
- `feat`: เพิ่ม feature ใหม่
- `fix`: แก้ไข bug
- `docs`: อัพเดตเอกสาร
- `style`: แก้ formatting (ไม่กระทบ logic)
- `refactor`: ปรับโครงสร้างโค้ด
- `test`: เพิ่มหรือแก้ไข tests
- `chore`: งานทั่วไป (update deps, configs)
- `perf`: ปรับปรุง performance

### ตัวอย่าง Commit Messages ที่ดี
```bash
git commit -m "feat: เพิ่มระบบแจ้งเตือนผ่านอีเมล

- What: เพิ่ม EmailService และ notification templates
- Why: ผู้ใช้ต้องการรับการแจ้งเตือนเมื่อมี order ใหม่
- Impact: ต้องตั้งค่า SMTP credentials ใน .env

Closes #15"

git commit -m "fix: แก้ไขปัญหา memory leak ใน ImageProcessor

- What: Clear buffer หลังประมวลผลรูปเสร็จ
- Why: Memory usage เพิ่มขึ้นเรื่อยๆ หลังอัพโหลดรูปหลายๆ รูป
- Impact: ลด memory usage ลง 60%

Fixes #23"
```

---

## 6. การแก้ปัญหาที่พบบ่อย

### Merge Conflicts
```bash
# เมื่อเกิด conflict
git status  # ดูไฟล์ที่ conflict

# แก้ไขไฟล์ที่ conflict manually
# มองหา <<<<<<< HEAD

# หลังแก้เสร็จ
git add [conflicted-files]
git commit -m "fix: resolve merge conflicts"
```

### ย้อน Commit กลับ (อย่างปลอดภัย)
```bash
# ดู commit ที่จะย้อน
git log --oneline -5

# Revert (สร้าง commit ใหม่ที่ยกเลิกการเปลี่ยนแปลง)
git revert [commit-hash]

# ❌ อย่าใช้ reset --hard ใน shared branches
# git reset --hard HEAD~1  # อันตราย!
```

### เปลี่ยนแปลงใน Wrong Branch
```bash
# ถ้ายังไม่ commit
git stash                      # เก็บการเปลี่ยนแปลงไว้ชั่วคราว
git checkout correct-branch    # ไปที่ branch ที่ถูกต้อง
git stash pop                  # เอาการเปลี่ยนแปลงกลับมา

# ถ้า commit ไปแล้ว
git log --oneline -1          # จด commit hash
git checkout correct-branch
git cherry-pick [commit-hash] # ย้าย commit ไปอีก branch
git checkout wrong-branch
git reset --soft HEAD~1       # ลบ commit จาก branch เดิม
```

### ดู Remote Branches
```bash
# อัพเดตข้อมูล remote
git fetch --all

# ดู remote branches ทั้งหมด
git branch -r

# Track remote branch
git checkout -b local-name origin/remote-branch-name
```

### สร้าง Alias เพื่อความสะดวก
```bash
# เพิ่มใน ~/.gitconfig หรือใช้คำสั่ง
git config --global alias.st "status"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.cm "commit -m"
git config --global alias.last "log -1 HEAD"
git config --global alias.unstage "reset HEAD --"
git config --global alias.visual "log --graph --pretty=oneline --abbrev-commit --all"

# ใช้งาน
git st        # แทน git status
git cm "msg"  # แทน git commit -m "msg"
git visual    # ดูกราฟ commits
```

---

## 📋 Retrospective Template

หลังจบ session การทำงาน ควรสร้างไฟล์ retrospective:

```markdown
# Session Retrospective
**วันที่**: YYYY-MM-DD
**เวลาเริ่ม**: HH:MM GMT+7
**เวลาจบ**: HH:MM GMT+7
**ระยะเวลา**: ~X นาที
**Focus หลัก**: [อธิบายสั้นๆ]
**Issue**: #XXX
**PR**: #XXX

## สรุป Session
[สรุป 2-3 ประโยคว่าทำอะไรสำเร็จ]

## Timeline
- HH:MM - เริ่ม session, review issue #XXX
- HH:MM - พัฒนา feature Y
- HH:MM - พบปัญหา Z
- HH:MM - Deploy สำเร็จ

## รายละเอียดทางเทคนิค
### ไฟล์ที่แก้ไข
$(git diff --name-only main...HEAD)

### การเปลี่ยนแปลงสำคัญ
- Component X: เพิ่ม feature Y
- Service Z: แก้ไข performance issue

## สิ่งที่ทำได้ดี
- แก้ปัญหา X ได้เร็วกว่าที่คิด
- Code structure ชัดเจน
- Tests ครอบคลุม

## สิ่งที่ควรปรับปรุง
- ควรวางแผนให้ละเอียดกว่านี้
- Documentation ยังไม่ครบ

## บทเรียนที่ได้
- Pattern X ใช้ได้ดีกับปัญหาแบบ Y
- Tool Z ช่วยประหยัดเวลาได้มาก

## ขั้นตอนถัดไป
- [ ] เพิ่ม integration tests
- [ ] Update documentation
- [ ] Deploy to production
```

---

## 🎯 สรุป Key Takeaways

1. **Safety First**: ไม่ใช้ force flags, ไม่ merge PR เอง
2. **Two-Issue Pattern**: แยก context กับ planning
3. **1-Hour Chunks**: แบ่งงานเป็นชิ้นเล็กๆ ทำให้เสร็จได้
4. **Clear Commits**: ข้อความ commit ต้องอธิบายชัดเจน
5. **Document Everything**: บันทึก retrospective ทุก session
6. **Test Before Push**: ทดสอบให้มั่นใจก่อน push
7. **Collaborative Review**: รอ review และ approval เสมอ

การทำงานกับ Git ที่ดีไม่ใช่แค่การใช้คำสั่ง แต่เป็นการสร้างวินัยและ workflow ที่ช่วยให้ทีมทำงานร่วมกันได้อย่างมีประสิทธิภาพและปลอดภัย
