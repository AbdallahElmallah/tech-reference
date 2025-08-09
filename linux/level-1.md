# Level 1: Foundation - Using, SSH, File Management

## Prerequisites
- Basic computer literacy and familiarity with operating systems
- Understanding of files, folders, and basic networking concepts
- Access to a Linux system (virtual machine, cloud instance, or WSL)
- Terminal emulator installed (Windows Terminal, PuTTY, or built-in terminal)

## Problem Statement
As a backend engineer, you need to:
- **Navigate Linux systems** confidently using command-line interface
- **Connect to remote servers** securely using SSH
- **Manage files and directories** efficiently without graphical interface
- **Understand basic system operations** for development and deployment
- **Work with text files** and basic system information
- **Manage running processes** and system resources

These skills form the foundation for all Linux-based development, deployment, and system administration tasks.

---

## Key Concepts

### 1. Linux Fundamentals

#### Understanding Linux Distributions
```bash
# Check your Linux distribution
# الغرض: معرفة نوع وإصدار نظام Linux المستخدم
cat /etc/os-release
# الشرح: cat يعرض محتوى الملف، /etc/os-release يحتوي على معلومات النظام
# مثال النتيجة: Ubuntu 22.04.3 LTS

# Check kernel version
# الغرض: معرفة إصدار نواة النظام والمعلومات التقنية
uname -a
# الشرح: uname يعرض معلومات النظام، -a يعني "all" لعرض كل المعلومات
# مثال النتيجة: Linux ubuntu 5.15.0-91-generic x86_64

# Check system information
# الغرض: عرض معلومات شاملة عن النظام بشكل منظم
hostnamectl
# الشرح: أداة systemd لعرض معلومات الهوست والنظام
# يعرض: اسم الجهاز، نوع النظام، الإصدار، المعمارية

# Check current user
# الغرض: معرفة اسم المستخدم الحالي
whoami
# الشرح: يعرض اسم المستخدم الذي تعمل به حالياً
# مثال النتيجة: ubuntu أو root

# Check current directory
# الغرض: معرفة المجلد الحالي الذي تتواجد فيه
pwd
# الشرح: pwd = Print Working Directory
# مثال النتيجة: /home/ubuntu أو /root
```

#### Basic Command Structure
```bash
# Command structure: command [options] [arguments]
# الهيكل الأساسي لأي أمر في Linux
ls -la /home
#  ^  ^   ^
#  |  |   └── argument (المعامل: المجلد المراد عرضه)
#  |  └────── option (الخيار: -la للعرض المفصل مع الملفات المخفية)
#  └───────── command (الأمر: ls لعرض محتويات المجلد)

# Getting help - الحصول على المساعدة
# الغرض: فهم كيفية استخدام أي أمر وخياراته
man ls          # دليل المستخدم الكامل للأمر ls
# الشرح: man = manual، يفتح صفحة شرح مفصلة (اضغط q للخروج)

ls --help       # مساعدة سريعة للأمر ls
# الشرح: يعرض ملخص سريع للخيارات المتاحة

info ls         # وثائق معلوماتية للأمر ls
# الشرح: نظام وثائق GNU، أكثر تفصيلاً من help

# Command history - تاريخ الأوامر
# الغرض: إعادة استخدام الأوامر السابقة بسرعة
history         # عرض قائمة الأوامر السابقة مع أرقامها
!!              # تكرار آخر أمر تم تنفيذه
!n              # تكرار الأمر رقم n من التاريخ
!string         # تكرار آخر أمر يبدأ بـ string
# مثال: !ls سيكرر آخر أمر يبدأ بـ ls
```

### 2. File System Navigation

#### Directory Structure
```bash
# Linux file system hierarchy - هيكل نظام الملفات في Linux
# الغرض: فهم تنظيم الملفات والمجلدات في النظام
/               # المجلد الجذر - نقطة البداية لكل شيء
├── bin/        # الملفات التنفيذية الأساسية للمستخدمين
├── boot/       # ملفات إقلاع النظام
├── dev/        # ملفات الأجهزة (devices)
├── etc/        # ملفات إعدادات النظام
├── home/       # مجلدات المستخدمين الشخصية
├── lib/        # المكتبات المشتركة الأساسية
├── opt/        # البرامج الاختيارية
├── proc/       # معلومات العمليات الجارية
├── root/       # المجلد الشخصي لمستخدم root
├── tmp/        # الملفات المؤقتة
├── usr/        # برامج وبيانات المستخدمين
└── var/        # البيانات المتغيرة (سجلات، قواعد بيانات)

# Navigation commands - أوامر التنقل
# الغرض: التحرك بين المجلدات ومعرفة الموقع الحالي
pwd             # طباعة مسار المجلد الحالي
# الشرح: pwd = Print Working Directory

ls              # عرض محتويات المجلد
# الشرح: يعرض أسماء الملفات والمجلدات في المجلد الحالي

ls -la          # عرض مفصل مع الملفات المخفية
# الشرح: -l للتفاصيل، -a للملفات المخفية (تبدأ بنقطة)

ls -lh          # عرض مفصل مع أحجام مقروءة
# الشرح: -h يعرض الأحجام بوحدات مفهومة (KB, MB, GB)

cd /path        # الانتقال إلى مسار مطلق
# الشرح: المسار المطلق يبدأ من الجذر /

cd ../          # الصعود مجلد واحد للأعلى
# الشرح: .. يشير إلى المجلد الأب

cd ~            # الانتقال إلى المجلد الرئيسي
# الشرح: ~ رمز للمجلد الشخصي للمستخدم

cd -            # العودة إلى المجلد السابق
# الشرح: - يحفظ آخر مجلد ويعود إليه
```

#### File and Directory Operations
```bash
# Creating directories
mkdir myproject
mkdir -p projects/backend/api    # Create nested directories

# Creating files
touch file.txt                   # Create empty file
echo "Hello World" > file.txt    # Create file with content
echo "New line" >> file.txt      # Append to file

# Copying files and directories
cp file.txt backup.txt           # Copy file
cp -r directory/ backup/         # Copy directory recursively
cp *.txt backup/                 # Copy all .txt files

# Moving and renaming
mv file.txt newname.txt          # Rename file
mv file.txt /path/to/directory/  # Move file
mv directory/ /new/location/     # Move directory

# Removing files and directories
rm file.txt                      # Remove file
rm -i file.txt                   # Remove with confirmation
rm -rf directory/                # Remove directory recursively (dangerous!)
rmdir empty_directory/           # Remove empty directory

# Finding files
find /path -name "*.txt"         # Find files by name
find /path -type f -size +1M     # Find files larger than 1MB
locate filename                  # Quick file search (if available)
which command                    # Find command location
```

### 3. SSH (Secure Shell)

#### Basic SSH Connection
```bash
# Basic SSH connection - الاتصال الأساسي بـ SSH
# الغرض: الوصول إلى جهاز آخر عبر الشبكة بشكل آمن
ssh username@hostname
# الشرح: ssh = Secure Shell، username اسم المستخدم، hostname عنوان الخادم
# مثال: ssh admin@myserver.com

ssh username@192.168.1.100
# الشرح: الاتصال باستخدام عنوان IP مباشرة

ssh -p 2222 username@hostname    # الاتصال بمنفذ مخصص
# الشرح: -p لتحديد رقم المنفذ (Port) إذا كان غير المنفذ الافتراضي 22

# SSH with key authentication - المصادقة بالمفاتيح
# الغرض: اتصال آمن بدون كلمة مرور باستخدام مفاتيح التشفير
ssh -i ~/.ssh/private_key username@hostname
# الشرح: -i لتحديد ملف المفتاح الخاص، أكثر أماناً من كلمة المرور

# SSH with verbose output (for troubleshooting) - إخراج مفصل لاستكشاف الأخطاء
# الغرض: عرض تفاصيل عملية الاتصال لحل المشاكل
ssh -v username@hostname
# الشرح: -v = verbose، يعرض خطوات الاتصال والأخطاء بالتفصيل

# SSH and execute command - تنفيذ أمر عبر SSH
# الغرض: تنفيذ أمر واحد على الخادم البعيد والعودة
ssh username@hostname 'ls -la'
# الشرح: ينفذ الأمر ls -la على الخادم البعيد ويعرض النتيجة

ssh username@hostname 'sudo systemctl status nginx'
# الشرح: فحص حالة خدمة nginx على الخادم البعيد
```

#### SSH Key Management
```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
ssh-keygen -t ed25519 -C "your.email@example.com"  # More secure

# Copy public key to remote server
ssh-copy-id username@hostname

# Manual key copying
cat ~/.ssh/id_rsa.pub | ssh username@hostname 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'

# SSH agent for key management
ssh-agent bash
ssh-add ~/.ssh/id_rsa
ssh-add -l                       # List loaded keys
```

#### SSH Configuration
```bash
# SSH client configuration (~/.ssh/config)
cat << 'EOF' > ~/.ssh/config
Host myserver
    HostName 192.168.1.100
    User myuser
    Port 22
    IdentityFile ~/.ssh/id_rsa
    ServerAliveInterval 60

Host *.example.com
    User admin
    IdentityFile ~/.ssh/work_key
    ForwardAgent yes
EOF

# Now you can connect simply with:
ssh myserver

# SSH tunneling (port forwarding)
ssh -L 8080:localhost:80 username@hostname    # Local port forwarding
ssh -R 8080:localhost:80 username@hostname    # Remote port forwarding
ssh -D 1080 username@hostname                 # SOCKS proxy
```

### 4. Text File Operations

#### Viewing File Contents
```bash
# Display file contents - عرض محتويات الملفات
# الغرض: قراءة وعرض محتوى الملفات النصية بطرق مختلفة
cat file.txt                     # عرض الملف كاملاً
# الشرح: cat = concatenate، يعرض كامل محتوى الملف مرة واحدة
# مناسب للملفات الصغيرة

less file.txt                    # عرض الملف صفحة بصفحة (q للخروج)
# الشرح: يعرض الملف بشكل تفاعلي، يمكن التنقل بالأسهم
# مفاتيح مفيدة: Space (صفحة تالية)، b (صفحة سابقة)، q (خروج)

more file.txt                    # مثل less لكن بخيارات أقل
# الشرح: إصدار أقدم من less، Space للصفحة التالية

head file.txt                    # عرض أول 10 أسطر
# الشرح: head = رأس، مفيد لمعاينة بداية الملف

head -n 20 file.txt              # عرض أول 20 سطر
# الشرح: -n لتحديد عدد الأسطر المطلوبة

tail file.txt                    # عرض آخر 10 أسطر
# الشرح: tail = ذيل، مفيد لرؤية نهاية الملف أو آخر السجلات

tail -n 20 file.txt              # عرض آخر 20 سطر
# الشرح: -n لتحديد عدد الأسطر من النهاية

tail -f /var/log/syslog          # متابعة تغييرات الملف (مفيد للسجلات)
# الشرح: -f = follow، يعرض الإضافات الجديدة فور حدوثها
# مثالي لمتابعة ملفات السجلات (logs) المباشرة

# File information - معلومات الملفات
# الغرض: الحصول على تفاصيل وإحصائيات الملفات
file filename                    # تحديد نوع الملف
# الشرح: يحلل محتوى الملف ويحدد نوعه (نص، صورة، تنفيذي، إلخ)

wc file.txt                      # عدد الكلمات والأسطر والأحرف
# الشرح: wc = word count، يعرض عدد الأسطر، الكلمات، الأحرف

wc -l file.txt                   # عدد الأسطر فقط
# الشرح: -l = lines، مفيد لمعرفة حجم الملف بالأسطر

du -h file.txt                   # حجم الملف
# الشرح: du = disk usage، -h للعرض بوحدات مقروءة (KB, MB, GB)

ls -lh file.txt                  # تفاصيل الملف
# الشرح: يعرض الصلاحيات، الحجم، تاريخ التعديل، المالك
```

#### Basic Text Editing
```bash
# Nano editor (beginner-friendly) - محرر نانو (سهل للمبتدئين)
# الغرض: تحرير الملفات النصية بطريقة بسيطة ومباشرة
nano file.txt
# الشرح: محرر نصوص بسيط يعرض الاختصارات في الأسفل
# Ctrl+X للخروج، Ctrl+O للحفظ، Ctrl+K لقص السطر
# Ctrl+U للصق، Ctrl+W للبحث

# Vim basics (more powerful) - أساسيات فيم (أكثر قوة)
# الغرض: تحرير متقدم للنصوص بكفاءة عالية
vim file.txt
# الشرح: محرر قوي لكن يحتاج تعلم، له أوضاع مختلفة:
# اضغط 'i' لوضع الإدراج (insert mode)
# اضغط 'Esc' لوضع الأوامر (command mode)
# ':w' للحفظ، ':q' للخروج، ':wq' للحفظ والخروج
# ':q!' للخروج بدون حفظ

# Quick text manipulation - معالجة سريعة للنصوص
# الغرض: إنشاء وتعديل الملفات بأوامر سريعة
echo "Hello World" > file.txt    # استبدال محتوى الملف
# الشرح: > يعيد توجيه الإخراج ويستبدل محتوى الملف

echo "New line" >> file.txt      # إضافة إلى نهاية الملف
# الشرح: >> يضيف النص إلى نهاية الملف دون حذف المحتوى الموجود

cat file1.txt file2.txt > combined.txt  # دمج الملفات
# الشرح: cat يعرض محتوى الملفين، > يحفظ النتيجة في ملف جديد
```

### 5. Process Management

#### Viewing Processes
```bash
# List running processes - عرض العمليات الجارية
# الغرض: مراقبة ومتابعة البرامج والخدمات التي تعمل على النظام
ps                               # عمليات المستخدم الحالي
# الشرح: ps = process status، يعرض العمليات الأساسية للمستخدم

ps aux                           # كل العمليات مع التفاصيل
# الشرح: a = all users، u = user format، x = processes without terminal
# يعرض: المستخدم، PID، استهلاك CPU، الذاكرة، الأمر

ps aux | grep nginx              # البحث عن عملية محددة
# الشرح: | pipe يمرر النتيجة لـ grep للبحث عن nginx

# Interactive process viewer - عارض العمليات التفاعلي
# الغرض: مراقبة العمليات في الوقت الفعلي
top                              # عارض العمليات المباشر
# الشرح: يعرض العمليات مرتبة حسب استهلاك CPU، يتحدث كل ثانية
# مفاتيح: q (خروج)، k (قتل عملية)، M (ترتيب حسب الذاكرة)

htop                             # نسخة محسنة (إذا كان مثبت)
# الشرح: واجهة ملونة وتفاعلية أكثر من top

# Process tree - شجرة العمليات
# الغرض: عرض العلاقات الهرمية بين العمليات
pstree                           # عرض هرمية العمليات
# الشرح: يعرض العمليات في شكل شجرة توضح العلاقات الأبوية

pstree -p                        # عرض مع أرقام العمليات
# الشرح: -p يضيف PID لكل عملية في الشجرة

# Jobs management - إدارة المهام
# الغرض: التحكم في المهام التي تعمل في الخلفية
jobs                             # عرض المهام النشطة
# الشرح: يعرض قائمة بالمهام في الخلفية مع حالتها

fg                               # إحضار مهمة للمقدمة
# الشرح: fg = foreground، يجعل آخر مهمة تعمل في المقدمة

bg                               # إرسال مهمة للخلفية
# الشرح: bg = background، يجعل المهمة المتوقفة تعمل في الخلفية

nohup command &                  # تشغيل أمر محمي من قطع الاتصال
# الشرح: nohup = no hang up، يحمي من الإنهاء عند قطع SSH
```

#### Controlling Processes
```bash
# Running commands in background - تشغيل الأوامر في الخلفية
# الغرض: تشغيل المهام دون تعطيل استخدام Terminal
command &                        # تشغيل في الخلفية
# الشرح: & في النهاية يجعل الأمر يعمل في الخلفية فوراً

Ctrl+Z                          # تعليق العملية الحالية
# الشرح: يوقف العملية مؤقتاً ويعطيك Terminal مرة أخرى

bg                              # متابعة العملية المعلقة في الخلفية
# الشرح: يجعل العملية المتوقفة تعمل في الخلفية

fg                              # إحضار عملية الخلفية للمقدمة
# الشرح: يجعل عملية الخلفية تعمل في المقدمة مرة أخرى

# Killing processes - إنهاء العمليات
# الغرض: إيقاف العمليات التي لا تستجيب أو غير مرغوبة
kill PID                        # إنهاء عملية برقم المعرف
# الشرح: يرسل إشارة TERM للعملية لإنهائها بلطف
# احصل على PID من ps أو top

kill -9 PID                     # إجبار إنهاء العملية
# الشرح: -9 يرسل إشارة KILL، إنهاء فوري وقسري
# استخدم فقط إذا لم تستجب للـ kill العادي

killall process_name            # قتل كل العمليات بالاسم
# الشرح: يقتل كل العمليات التي تحمل نفس الاسم
# مثال: killall firefox

pkill -f pattern                # قتل العمليات المطابقة للنمط
# الشرح: -f يبحث في كامل سطر الأمر، pattern نمط البحث
# مثال: pkill -f "python script.py"

# Process monitoring - مراقبة العمليات
# الغرض: متابعة حالة العمليات والنظام بشكل مستمر
watch 'ps aux | grep nginx'     # مراقبة إخراج الأمر
# الشرح: watch يكرر الأمر كل ثانيتين ويعرض النتيجة
# مفيد لمراقبة التغييرات المستمرة

watch -n 5 'df -h'             # مراقبة استخدام القرص كل 5 ثوان
# الشرح: -n 5 يحدد فترة التحديث بـ 5 ثوان
# مفيد لمراقبة مساحة القرص أثناء العمليات الكبيرة
```

---

## Hands-on Examples

### Example 1: Setting Up Development Environment
```bash
# Create project structure - إنشاء هيكل المشروع
# الغرض: تنظيم ملفات المشروع في مجلدات منطقية
mkdir -p ~/projects/myapp/{src,tests,docs,config}
# الشرح: mkdir ينشئ مجلدات، -p ينشئ المجلدات الأب إذا لم تكن موجودة
# {} يوسع إلى أربع مجلدات منفصلة، ~ يشير للمجلد الرئيسي

cd ~/projects/myapp
# الشرح: الانتقال إلى مجلد المشروع الجديد

# Create basic files - إنشاء الملفات الأساسية
# الغرض: إنشاء ملف README لوصف المشروع
touch README.md
# الشرح: touch ينشئ ملف فارغ إذا لم يكن موجوداً

echo "# My Application" > README.md
# الشرح: echo يطبع النص، > يحفظه في الملف (يستبدل المحتوى)

echo "TODO: Add project description" >> README.md
# الشرح: >> يضيف النص إلى نهاية الملف دون حذف المحتوى الموجود

# Create source files - إنشاء ملفات الكود المصدري
# الغرض: إنشاء ملف Python أساسي قابل للتنفيذ
touch src/main.py
# الشرح: إنشاء ملف Python فارغ في مجلد src

echo "#!/usr/bin/env python3" > src/main.py
# الشرح: shebang line يحدد المفسر المطلوب لتنفيذ الملف

echo "print('Hello, World!')" >> src/main.py
# الشرح: إضافة كود Python بسيط يطبع رسالة

# Make script executable - جعل الملف قابل للتنفيذ
# الغرض: إعطاء صلاحية التنفيذ للملف
chmod +x src/main.py
# الشرح: chmod يغير صلاحيات الملف، +x يضيف صلاحية التنفيذ
# الآن يمكن تشغيل الملف مباشرة: ./src/main.py

# View project structure - عرض هيكل المشروع
# الغرض: التحقق من أن كل شيء تم إنشاؤه بشكل صحيح
tree .  # or use ls -la if tree is not available
# الشرح: tree يعرض هيكل المجلدات في شكل شجرة، . يشير للمجلد الحالي

find . -type f -name "*.py"
# الشرح: find يبحث عن الملفات، -type f للملفات فقط، -name للبحث بالاسم
# "*.py" يبحث عن كل الملفات التي تنتهي بـ .py

# Check file contents - فحص محتويات الملفات
# الغرض: التأكد من أن المحتوى تم إنشاؤه بشكل صحيح
cat src/main.py
# الشرح: cat يعرض كامل محتوى الملف

wc -l src/main.py
# الشرح: wc -l يعد عدد الأسطر في الملف
```

### Example 2: Remote Server Management
```bash
# Connect to remote server - الاتصال بالخادم البعيد
# الغرض: الوصول إلى خادم بعيد لإدارته عن بُعد
ssh user@server.example.com
# الشرح: ssh يفتح اتصال آمن مشفر مع الخادم البعيد
# استبدل user باسم المستخدم و server.example.com بعنوان الخادم

# Check server information - فحص معلومات الخادم
# الغرض: جمع معلومات أساسية عن حالة الخادم
hostname
# الشرح: يعرض اسم الخادم المحدد في النظام

uptime
# الشرح: يعرض منذ متى الخادم يعمل ومتوسط الحمولة
# Load average: أرقام تمثل ضغط العمل (أقل من 1.0 جيد)

df -h                           # Disk usage - استخدام القرص
# الشرح: يعرض مساحة الأقراص المستخدمة والمتاحة بوحدات مقروءة
# تحقق من أن لا يوجد قرص ممتلئ (أكثر من 90%)

free -h                         # Memory usage - استخدام الذاكرة
# الشرح: يعرض استخدام الذاكرة بوحدات مقروءة (GB, MB)
# Available: الذاكرة المتاحة فعلياً للتطبيقات الجديدة

ps aux | head -10               # Top processes - أهم العمليات
# الشرح: ps aux يعرض كل العمليات، head -10 يحدد أول 10 عمليات
# يعرض استهلاك CPU والذاكرة لكل عملية

# Navigate to application directory - الانتقال لمجلد التطبيق
# الغرض: الوصول إلى ملفات التطبيق لفحصها أو تحديثها
cd /var/www/myapp
# الشرح: /var/www مجلد تطبيقات الويب التقليدي في Linux

pwd
# الشرح: تأكيد الموقع الحالي بعد الانتقال

ls -la
# الشرح: عرض محتويات مجلد التطبيق مع التفاصيل والملفات المخفية

# Check log files - فحص ملفات السجلات
# الغرض: مراقبة نشاط التطبيق والأخطاء
tail -f /var/log/nginx/access.log
# الشرح: متابعة سجل الوصول لـ nginx مباشرة
# مفيد لمراقبة الزيارات والطلبات الواردة
# Press Ctrl+C to stop following - اضغط Ctrl+C لإيقاف المتابعة

# Copy files from local to remote - نسخ الملفات من المحلي للبعيد
# الغرض: رفع الملفات والتحديثات إلى الخادم
# (run this from your local machine) - شغل هذا من جهازك المحلي
scp local_file.txt user@server:/path/to/destination/
# الشرح: scp = secure copy، ينسخ ملف واحد إلى الخادم البعيد

scp -r local_directory/ user@server:/path/to/destination/
# الشرح: -r للنسخ التكراري (recursive) لنسخ المجلدات وكل محتوياتها

# Copy files from remote to local - نسخ الملفات من البعيد للمحلي
# الغرض: تحميل الملفات والنسخ الاحتياطية من الخادم
scp user@server:/path/to/file.txt ./
# الشرح: نسخ ملف من الخادم البعيد إلى المجلد الحالي (.)

scp -r user@server:/path/to/directory/ ./
# الشرح: نسخ مجلد كامل من الخادم البعيد إلى المجلد الحالي
```

### Example 3: Log Analysis and Monitoring
```bash
# Navigate to log directory - الانتقال إلى مجلد السجلات
# الغرض: الوصول إلى مجلد السجلات الرئيسي في النظام
cd /var/log
# الشرح: /var/log هو المجلد القياسي لحفظ سجلات النظام والتطبيقات

# List log files - عرض ملفات السجلات
# الغرض: استكشاف ملفات السجلات المتاحة
ls -la
# الشرح: عرض كل الملفات مع التفاصيل والملفات المخفية

ls -lh *.log
# الشرح: عرض فقط الملفات التي تنتهي بـ .log مع أحجام مقروءة

# Monitor system logs - مراقبة سجلات النظام
# الغرض: متابعة أحداث النظام في الوقت الفعلي
tail -f /var/log/syslog
# الشرح: -f يتابع الإضافات الجديدة للملف فور حدوثها
# مفيد لمراقبة الأحداث المباشرة، Ctrl+C للإيقاف

# Search for specific patterns in logs - البحث عن أنماط محددة
# الغرض: العثور على رسائل خطأ أو أحداث معينة
grep "error" /var/log/syslog
# الشرح: grep يبحث عن كلمة "error" في ملف syslog

grep -i "failed" /var/log/auth.log  # Case-insensitive search
# الشرح: -i يجعل البحث غير حساس لحالة الأحرف
# auth.log يحتوي على سجلات المصادقة والدخول

grep "$(date '+%b %d')" /var/log/syslog  # Today's logs - سجلات اليوم
# الشرح: $(date '+%b %d') ينتج تاريخ اليوم بصيغة "Dec 25"

# Count occurrences - عد التكرارات
# الغرض: معرفة عدد مرات حدوث خطأ أو حدث معين
grep -c "error" /var/log/syslog
# الشرح: -c يعد عدد الأسطر المطابقة بدلاً من عرضها

grep "error" /var/log/syslog | wc -l
# الشرح: طريقة بديلة، grep يجد الأسطر، wc -l يعدها

# Search in multiple files - البحث في ملفات متعددة
# الغرض: البحث في كل ملفات السجلات عن مشكلة معينة
grep -r "connection refused" /var/log/
# الشرح: -r للبحث التكراري في كل الملفات داخل المجلد

find /var/log -name "*.log" -exec grep -l "error" {} \;
# الشرح: find يبحث عن ملفات .log، -exec ينفذ grep على كل ملف

# View compressed log files - عرض ملفات السجلات المضغوطة
# الغرض: قراءة السجلات القديمة المضغوطة دون فك الضغط
zcat /var/log/syslog.1.gz | grep "error"
# الشرح: zcat يقرأ الملفات المضغوطة (.gz) مباشرة

zgrep "error" /var/log/syslog.*.gz
# الشرح: zgrep يبحث مباشرة في الملفات المضغوطة

# Monitor multiple files - مراقبة ملفات سجلات متعددة
# الغرض: متابعة عدة سجلات في نفس الوقت
tail -f /var/log/syslog /var/log/auth.log
# الشرح: tail -f يتابع ملفين في نفس الوقت

# Real-time process monitoring - مراقبة العمليات المباشرة
# الغرض: متابعة العمليات والموارد في الوقت الفعلي
watch 'ps aux | grep python'
# الشرح: watch يكرر الأمر كل ثانيتين، مفيد لمراقبة عمليات Python

watch -n 2 'free -h'            # Update every 2 seconds
# الشرح: -n 2 يحدد فترة التحديث بثانيتين، free -h يعرض استخدام الذاكرة

# Check disk usage - فحص استخدام القرص
# الغرض: مراقبة مساحة التخزين وحجم ملفات السجلات
du -sh /var/log/*               # Size of log files
# الشرح: du -sh يعرض حجم كل ملف/مجلد في /var/log بوحدات مقروءة

df -h                           # Filesystem usage
# الشرح: df -h يعرض استخدام كل نظام ملفات مع النسب المئوية

# Filter and format log output - تصفية وتنسيق إخراج السجلات
# الغرض: استخراج معلومات محددة وتنسيقها
grep "ssh" /var/log/auth.log | awk '{print $1, $2, $3, $9, $11}'
# الشرح: grep يجد أسطر SSH، awk يستخرج أعمدة محددة (التاريخ والمعلومات)

grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
# الشرح: تحليل محاولات الدخول الفاشلة وترتيبها حسب التكرار
```

---

## Best Practices

### Command Line Efficiency
1. **Use Tab Completion**: Press Tab to auto-complete commands and paths
2. **Command History**: Use Up/Down arrows to navigate command history
3. **Aliases**: Create shortcuts for frequently used commands
4. **Wildcards**: Use `*`, `?`, and `[]` for pattern matching
5. **Pipes**: Combine commands with `|` for powerful operations

### SSH Security
1. **Use Key Authentication**: Disable password authentication when possible
2. **Strong Keys**: Use RSA 4096-bit or Ed25519 keys
3. **SSH Config**: Organize connections with SSH config file
4. **Port Changes**: Use non-standard ports for SSH when possible
5. **Connection Timeouts**: Configure appropriate timeout values

### File Management
1. **Backup Important Files**: Always backup before making changes
2. **Use Descriptive Names**: Choose clear, descriptive filenames
3. **Organize Directory Structure**: Keep projects organized
4. **Check Before Deleting**: Use `ls` before `rm` to verify
5. **Use Version Control**: Track changes with Git

---

## Common Mistakes

1. **Using `rm -rf` carelessly**: Always double-check paths before deleting
2. **Not using SSH keys**: Relying only on passwords is less secure
3. **Ignoring file permissions**: Can cause application failures
4. **Not backing up**: Losing important data due to lack of backups
5. **Poor directory organization**: Making projects hard to navigate
6. **Not reading error messages**: Missing important troubleshooting information
7. **Using root unnecessarily**: Running commands as root when not needed
8. **Not using screen/tmux**: Losing work when SSH connection drops

---

## Simple Practice Projects

### Project 1: Personal File Organizer
**Objective**: Create a script to organize downloaded files

**Tasks**:
- Create directory structure for different file types
- Move files based on extensions (.pdf, .jpg, .txt, etc.)
- Generate a report of organized files
- Practice with find, mv, and basic scripting

### Project 2: Remote Server Setup
**Objective**: Set up and configure a remote server connection

**Tasks**:
- Create SSH key pair
- Configure SSH client settings
- Connect to a cloud instance (AWS, DigitalOcean, etc.)
- Set up basic security (change default ports, disable root login)
- Practice file transfer with scp

### Project 3: Log Monitoring Dashboard
**Objective**: Create a simple log monitoring solution

**Tasks**:
- Monitor multiple log files simultaneously
- Create alerts for specific error patterns
- Generate daily log summaries
- Practice with tail, grep, and basic text processing

---

## Related Levels
- **Level 2**: [Permissions, Logs, Commands, Git](./level-2.md) - System administration and version control
- **Level 3**: [Scripting, Installation](./level-3.md) - Automation and advanced system management
- **Related Topics**: Docker, System Administration, DevOps

---

## Q&A Section

**Q: What's the difference between absolute and relative paths?**
A: Absolute paths start from root (/) like `/home/user/file.txt`, while relative paths are relative to current directory like `../file.txt` or `./subdirectory/file.txt`. Use `pwd` to see your current location.

**Q: How do I safely delete files without losing important data?**
A: Always use `ls` to verify what you're about to delete, use `rm -i` for interactive deletion, avoid `rm -rf` unless absolutely necessary, and maintain regular backups. Consider moving files to a trash directory first.

**Q: Why can't I connect to my server via SSH?**
A: Common issues include: wrong hostname/IP, incorrect port, firewall blocking connection, SSH service not running, or authentication problems. Use `ssh -v` for verbose output to troubleshoot.

**Q: What's the difference between `cat`, `less`, and `more`?**
A: `cat` displays entire file at once (good for small files), `less` allows scrolling and searching (better for large files), `more` is similar to less but with fewer features. Use `less` for most file viewing tasks.

**Q: How do I run commands in the background?**
A: Add `&` at the end of command to run in background, use `Ctrl+Z` to suspend current process then `bg` to continue in background, or use `nohup command &` to run immune to hangups.

**Q: What should I do if I accidentally delete important files?**
A: Stop using the system immediately to prevent overwriting, use file recovery tools like `testdisk` or `photorec`, restore from backups if available. Prevention is better - always backup important data.

**Q: How do I transfer large files efficiently over SSH?**
A: Use `scp` with compression (`scp -C`), `rsync` for incremental transfers, or `tar` with SSH for multiple files: `tar czf - directory/ | ssh user@host 'tar xzf - -C /destination/'`.

**Q: What's the best way to learn vim?**
A: Start with `vimtutor` (built-in tutorial), learn basic modes (normal, insert, visual), master essential commands (i, :w, :q, dd, yy, p), practice daily, and gradually learn advanced features. Consider starting with nano if vim feels overwhelming.