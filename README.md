# resize_photo

* Purpose: This code snippet is used to resize the image.
* Usage:
1. Put image at the root folder
2. Edit input/output file's name 
3. Run this command line to resize image "py .\resize.py"


Nếu upgrade Terasoluna từ 5.7.0 → 5.8.0 (hoặc version mới nào), nhiều dependencies sẽ bị upgrade:

Terasoluna 5.7.0.RELEASE          →  Terasoluna 5.8.0.RELEASE
├── Spring 5.3.2                  →  Spring 5.4.0
├── Spring Security 5.4.2         →  Spring Security 5.5.0
├── Jackson 2.13.1                →  Jackson 2.14.0
├── MyBatis 3.5.6                 →  MyBatis 3.5.10
└── ... (hàng chục thư viện khác)


Hiện tại, process upgrade là:

1. Tải Terasoluna 5.8.0 từ đâu đó
   ↓
2. Xem trong đó chứa dependencies nào (version nào)
   ↓
3. Download lần lượt từng JAR:
   ✓ spring-core-5.4.0.jar
   ✓ spring-beans-5.4.0.jar
   ✓ jackson-core-2.14.0.jar
   ... (100+ files)
   ↓
4. Xóa toàn bộ thư viện cũ:
   rm /ServerLib/lib/terasoluna5/*.jar
   ↓
5. Copy JAR mới vào:
   cp spring-core-5.4.0.jar /ServerLib/lib/terasoluna5/
   ... (copy 100+ lần)
   ↓
6. Update file config:
   - all.userlibraries (thêm 100+ archive entries)
   - .classpath (nếu version thay đổi)
   ↓
7. Test compile & run
   ↓
8. Commit toàn bộ vào Git
   git add ...
   git commit "Upgrade Terasoluna 5.7.0 → 5.8.0"

   Vấn đề:

🐌 Quá trình thủ công và mệt mỏi
📦 Repository sẽ rất nặng (100MB+ JAR files)
🔄 Dễ nhầm lẫn version
⏱️ Mất rất nhiều thời gian


Giải pháp:

Option 1: Migrate sang Maven (Best)

<!-- pom.xml -->
<dependency>
    <groupId>org.terasoluna.gfw</groupId>
    <artifactId>terasoluna-gfw-web-spring5-mvc-parent</artifactId>
    <version>5.8.0.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>

$ mvn clean install
# Maven tự động download 100+ JARs từ Maven Central!
# Xong!

Ưu điểm:

✓ Tự động download dependencies
✓ Quản lý version dễ (chỉ cần update version number)
✓ Repository nhẹ (không commit JAR)
✓ CI/CD tự động handle


Option 2: Tạo script tự động (Quick fix)
#!/bin/bash
# download_terasoluna.sh

VERSION="5.8.0.RELEASE"
TERASOLUNA_URL="https://repo1.maven.org/maven2/org/terasoluna/gfw"


# Download từng dependency
download_jar() {
    local GROUP=$1
    local ARTIFACT=$2
    local VERSION=$3
    curl -o "$ARTIFACT-$VERSION.jar" \
        "$TERASOLUNA_URL/$GROUP/$ARTIFACT/$VERSION/$ARTIFACT-$VERSION.jar"
}

# Download Spring
download_jar "org/springframework" "spring-core" "5.4.0"
download_jar "org/springframework" "spring-beans" "5.4.0"
...

