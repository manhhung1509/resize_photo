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






=========================================

<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<eclipse-userlibraries version="2">
    <library name="TERASOLUNA５ライブラリ" systemlibrary="false">
        <archive path="/ServerLib/lib/terasoluna5/animal-sniffer-annotations-1.17.jar" source="/ServerLib/lib/terasoluna5/animal-sniffer-annotations-1.17-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/asm-7.3.1.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/aspectjrt-1.9.6.jar" source="/ServerLib/lib/terasoluna5/aspectjrt-1.9.6-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/aspectjweaver-1.9.6.jar" source="/ServerLib/lib/terasoluna5/aspectjweaver-1.9.6-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/checker-qual-2.5.2.jar" source="/ServerLib/lib/terasoluna5/checker-qual-2.5.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/classmate-1.5.1.jar" source="/ServerLib/lib/terasoluna5/classmate-1.5.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-beanutils-1.9.4.jar" source="/ServerLib/lib/terasoluna5/commons-beanutils-1.9.4-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-codec-1.11.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-collections-3.2.2.jar" source="/ServerLib/lib/terasoluna5/commons-collections-3.2.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-compress-1.20.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-dbcp2-2.8.0.jar" source="/ServerLib/lib/terasoluna5/commons-dbcp2-2.8.0-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-digester-2.1.jar" source="/ServerLib/lib/terasoluna5/commons-digester-2.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-io-2.6.jar" source="/ServerLib/lib/terasoluna5/commons-io-2.6-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-lang-2.6.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-lang3-3.11.jar" source="/ServerLib/lib/terasoluna5/commons-lang3-3.11-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-logging-1.2.jar" source="/ServerLib/lib/terasoluna5/commons-logging-1.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-pool2-2.9.0.jar" source="/ServerLib/lib/terasoluna5/commons-pool2-2.9.0-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/commons-validator-1.7.jar" source="/ServerLib/lib/terasoluna5/commons-validator-1.7-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/dozer-core-6.5.0.jar" source="/ServerLib/lib/terasoluna5/dozer-core-6.5.0-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/dozer-spring4-6.5.0.jar" source="/ServerLib/lib/terasoluna5/dozer-spring4-6.5.0-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/error_prone_annotations-2.2.0.jar" source="/ServerLib/lib/terasoluna5/error_prone_annotations-2.2.0-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/failureaccess-1.0.1.jar" source="/ServerLib/lib/terasoluna5/failureaccess-1.0.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/guava-27.0.1-jre.jar" source="/ServerLib/lib/terasoluna5/guava-27.0.1-jre-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/hibernate-validator-6.1.6.Final.jar" source="/ServerLib/lib/terasoluna5/hibernate-validator-6.1.6.Final-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/istack-commons-runtime-3.0.11.jar" source="/ServerLib/lib/terasoluna5/istack-commons-runtime-3.0.11-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/j2objc-annotations-1.1.jar" source="/ServerLib/lib/terasoluna5/j2objc-annotations-1.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jackson-annotations-2.13.1.jar" source="/ServerLib/lib/terasoluna5/jackson-annotations-2.13.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jackson-core-2.13.1.jar" source="/ServerLib/lib/terasoluna5/jackson-core-2.13.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jackson-databind-2.13.1.jar" source="/ServerLib/lib/terasoluna5/jackson-databind-2.13.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jackson-dataformat-xml-2.13.1.jar" source="/ServerLib/lib/terasoluna5/jackson-dataformat-xml-2.13.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jackson-datatype-joda-2.13.1.jar" source="/ServerLib/lib/terasoluna5/jackson-datatype-joda-2.13.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jackson-datatype-jsr310-2.13.1.jar" source="/ServerLib/lib/terasoluna5/jackson-datatype-jsr310-2.13.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jackson-module-jaxb-annotations-2.13.1.jar" source="/ServerLib/lib/terasoluna5/jackson-module-jaxb-annotations-2.13.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jakarta.activation-api-1.2.2.jar" source="/ServerLib/lib/terasoluna5/jakarta.activation-api-1.2.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jakarta.validation-api-2.0.2.jar" source="/ServerLib/lib/terasoluna5/jakarta.validation-api-2.0.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jakarta.xml.bind-api-2.3.3.jar" source="/ServerLib/lib/terasoluna5/jakarta.xml.bind-api-2.3.3-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/javax.annotation-api-1.3.2.jar" source="/ServerLib/lib/terasoluna5/javax.annotation-api-1.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/javax.inject-1.jar" source="/ServerLib/lib/terasoluna5/javax.inject-1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jaxb-impl-2.2.3-1.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jaxb-runtime-2.3.3.jar" source="/ServerLib/lib/terasoluna5/jaxb-runtime-2.3.3-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jboss-logging-3.4.1.Final.jar" source="/ServerLib/lib/terasoluna5/jboss-logging-3.4.1.Final-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jcl-over-slf4j-1.7.30.jar" source="/ServerLib/lib/terasoluna5/jcl-over-slf4j-1.7.30-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/joda-time-2.10.10.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/joda-time-jsptags-1.1.1.jar" source="/ServerLib/lib/terasoluna5/joda-time-jsptags-1.1.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/jsr305-3.0.2.jar" source="/ServerLib/lib/terasoluna5/jsr305-3.0.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/logback-classic-1.2.3.jar" source="/ServerLib/lib/terasoluna5/logback-classic-1.2.3-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/logback-core-1.2.3.jar" source="/ServerLib/lib/terasoluna5/logback-core-1.2.3-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/mybatis-3.5.6.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/mybatis-spring-2.0.6.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/objenesis-3.1.jar" source="/ServerLib/lib/terasoluna5/objenesis-3.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/oro-2.0.8.jar" source="/ServerLib/lib/terasoluna5/oro-2.0.8-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/ojdbc8-21.1.0.0.jar" source="/ServerLib/lib/terasoluna5/ojdbc8-21.1.0.0-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/ucp-19.8.0.0.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/plexus-velocity-1.2.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/slf4j-api-1.7.30.jar" source="/ServerLib/lib/terasoluna5/slf4j-api-1.7.30-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-aop-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-aop-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-aspects-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-aspects-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-beans-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-beans-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-context-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-context-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-context-support-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-context-support-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-core-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-core-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-data-commons-2.4.2.jar" source="/ServerLib/lib/terasoluna5/spring-data-commons-2.4.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-expression-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-expression-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-jcl-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-jcl-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-jdbc-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-jdbc-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-modules-validation-0.8.jar" source="/ServerLib/lib/terasoluna5/spring-modules-validation-0.8-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-orm-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-orm-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-security-acl-5.4.2.jar" source="/ServerLib/lib/terasoluna5/spring-security-acl-5.4.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-security-config-5.4.2.jar" source="/ServerLib/lib/terasoluna5/spring-security-config-5.4.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-security-core-5.4.2.jar" source="/ServerLib/lib/terasoluna5/spring-security-core-5.4.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-security-taglibs-5.4.2.jar" source="/ServerLib/lib/terasoluna5/spring-security-taglibs-5.4.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-security-web-5.4.2.jar" source="/ServerLib/lib/terasoluna5/spring-security-web-5.4.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-tx-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-tx-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-web-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-web-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/spring-webmvc-5.3.2.jar" source="/ServerLib/lib/terasoluna5/spring-webmvc-5.3.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/stax2-api-4.2.1.jar" source="/ServerLib/lib/terasoluna5/stax2-api-4.2.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/taglibs-standard-impl-1.2.5.jar" source="/ServerLib/lib/terasoluna5/taglibs-standard-impl-1.2.5-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/taglibs-standard-jstlel-1.2.5.jar" source="/ServerLib/lib/terasoluna5/taglibs-standard-jstlel-1.2.5-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/taglibs-standard-spec-1.2.5.jar" source="/ServerLib/lib/terasoluna5/taglibs-standard-spec-1.2.5-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/terasoluna-gfw-common-5.7.0.RELEASE.jar" source="/ServerLib/lib/terasoluna5/terasoluna-gfw-common-5.7.0.RELEASE-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/terasoluna-gfw-jodatime-5.7.0.RELEASE.jar" source="/ServerLib/lib/terasoluna5/terasoluna-gfw-jodatime-5.7.0.RELEASE-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/terasoluna-gfw-security-web-5.7.0.RELEASE.jar" source="/ServerLib/lib/terasoluna5/terasoluna-gfw-security-web-5.7.0.RELEASE-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/terasoluna-gfw-web-5.7.0.RELEASE.jar" source="/ServerLib/lib/terasoluna5/terasoluna-gfw-web-5.7.0.RELEASE-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/terasoluna-gfw-web-jsp-5.7.0.RELEASE.jar" source="/ServerLib/lib/terasoluna5/terasoluna-gfw-web-jsp-5.7.0.RELEASE-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-api-3.0.8.jar" source="/ServerLib/lib/terasoluna5/tiles-api-3.0.8-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-autotag-core-runtime-1.2.jar" source="/ServerLib/lib/terasoluna5/tiles-autotag-core-runtime-1.2-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-core-3.0.8.jar" source="/ServerLib/lib/terasoluna5/tiles-core-3.0.8-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-jsp-3.0.8.jar" source="/ServerLib/lib/terasoluna5/tiles-jsp-3.0.8-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-request-api-1.0.7.jar" source="/ServerLib/lib/terasoluna5/tiles-request-api-1.0.7-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-request-jsp-1.0.7.jar" source="/ServerLib/lib/terasoluna5/tiles-request-jsp-1.0.7-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-request-servlet-1.0.7.jar" source="/ServerLib/lib/terasoluna5/tiles-request-servlet-1.0.7-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-servlet-3.0.8.jar" source="/ServerLib/lib/terasoluna5/tiles-servlet-3.0.8-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tiles-template-3.0.8.jar" source="/ServerLib/lib/terasoluna5/tiles-template-3.0.8-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tomcat-el-api-9.0.41.jar" source="/ServerLib/lib/terasoluna5/tomcat-el-api-9.0.41-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tomcat-jsp-api-9.0.41.jar" source="/ServerLib/lib/terasoluna5/tomcat-jsp-api-9.0.41-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/tomcat-servlet-api-9.0.46.jar" source="/ServerLib/lib/terasoluna5/tomcat-servlet-api-9.0.46-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/txw2-2.3.3.jar" source="/ServerLib/lib/terasoluna5/txw2-2.3.3-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/validation-api-1.1.0.Final.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/velocity-1.7.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/velocity-tools-2.0.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/woodstox-core-6.2.1.jar" source="/ServerLib/lib/terasoluna5/woodstox-core-6.2.1-sources.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/xml-apis-1.3.04.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/KeiDensiKnssho-1.0.0.jar"/>
        <archive path="/ServerLib/lib/terasoluna5/LunaProvider-10.3.0.jar"/>
    </library>
    <library name="テストライブラリ" systemlibrary="false">
        <archive path="/ServerLib/lib/test/asm-5.0.4.jar"/>
        <archive path="/ServerLib/lib/test/asm-tree-3.3.1.jar"/>
        <archive path="/ServerLib/lib/test/cobertura.jar"/>
        <archive path="/ServerLib/lib/test/dbunit-2.5.1.jar" source="/ServerLib/lib/test/dbunit-2.5.1-sources.jar"/>
        <archive path="/ServerLib/lib/test/dom4j-1.6.1.jar"/>
        <archive path="/ServerLib/lib/test/easymockclassextension-3.2.jar"/>
        <archive path="/ServerLib/lib/test/goya-tmg-runtime-2.1.2.jar"/>
        <archive path="/ServerLib/lib/test/hamcrest-core-1.3.jar" source="/ServerLib/lib/test/hamcrest-core-1.3-sources.jar"/>
        <archive path="/ServerLib/lib/test/hamcrest-library-1.3.jar" source="/ServerLib/lib/test/hamcrest-library-1.3-sources.jar"/>
        <archive path="/ServerLib/lib/test/jmockit-1.36.3.jar"/>
        <archive path="/ServerLib/lib/test/jp.co.nttdata.terasoluna.ractes.reflect.jar"/>
        <archive path="/ServerLib/lib/test/junit-4.12.jar" source="/ServerLib/lib/test/junit-4.12-sources.jar"/>
        <archive path="/ServerLib/lib/test/junit-addons-1.4.jar"/>
        <archive path="/ServerLib/lib/test/mockito-core-1.10.19.jar"/>
        <archive path="/ServerLib/lib/test/objenesis-1.2.jar"/>
        <archive path="/ServerLib/lib/test/ooxml-schemas-1.1.jar"/>
        <archive path="/ServerLib/lib/test/poi-3.12-20150511.jar" source="/ServerLib/lib/test/poi-3.12-sources.jar"/>
        <archive path="/ServerLib/lib/test/poi-ooxml-3.12-20150511.jar" source="/ServerLib/lib/test/poi-ooxml-3.12-sources.jar"/>
        <archive path="/ServerLib/lib/test/poi-ooxml-schemas-3.12-20150511.jar"/>
        <archive path="/ServerLib/lib/test/poi-scratchpad-3.12-20150511.jar" source="/ServerLib/lib/test/poi-scratchpad-3.12-sources.jar"/>
        <archive path="/ServerLib/lib/test/slf4j-api-1.7.8.jar"/>
        <archive path="/ServerLib/lib/test/slf4j-log4j12-1.7.12.jar"/>
        <archive path="/ServerLib/lib/test/spring-test-3.1.3.RELEASE.jar" source="/ServerLib/lib/test/spring-test-3.1.3.RELEASE-sources.jar"/>
        <archive path="/ServerLib/lib/test/xmlbeans-2.6.0.jar"/>
        <archive path="/ServerLib/lib/test/xmlpull-1.1.3.1.jar"/>
        <archive path="/ServerLib/lib/test/xpp3_min-1.1.4c.jar"/>
        <archive path="/ServerLib/lib/test/xstream-1.4.9.jar"/>
    </library>
    <library name="拡張ライブラリ" systemlibrary="false">
        <archive path="/ServerLib/lib/extension/ant.jar"/>
        <archive path="/ServerLib/lib/extension/ini4j-0.5.1.jar"/>
        <archive path="/ServerLib/lib/extension/quartz-2.2.1.jar"/>
    </library>
</eclipse-userlibraries>



============================
Context: Tôi đang làm một dự án Java legacy migration. Project hiện tại dùng Apache Ant và không được phép chuyển sang Maven hoặc Gradle. Vấn đề chính là khi upgrade TERASOLUNA từ 5.7.0.RELEASE lên 5.8.0.RELEASE hoặc version mới hơn, rất nhiều dependency như Spring, Spring Security, Jackson, MyBatis… sẽ thay đổi. Hiện tại project đang quản lý JAR thủ công trong `/ServerLib/lib/terasoluna5/` và Eclipse `all.userlibraries`, dẫn đến repo nặng, dễ sai version, dễ thiếu transitive dependency và khó maintain.

Yêu cầu: Đề xuất giải pháp vẫn giữ Ant nhưng tự động hóa dependency management. Hướng nên làm là dùng Apache Ivy tích hợp với Ant để resolve dependency từ Maven Central hoặc Nexus nội bộ, retrieve JAR vào `/ServerLib/lib/terasoluna5`, sync JAR cũ/mới, generate lại `all.userlibraries`, và tránh commit JAR vào Git. Cần chuẩn bị các file như `build.xml`, `ivy.xml`, `ivysettings.xml`, `build.properties`, script generate Eclipse user libraries, script verify libs, `.gitignore`, README và migration notes.


