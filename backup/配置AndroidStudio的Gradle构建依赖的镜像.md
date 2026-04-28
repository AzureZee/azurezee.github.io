# 1.项目级配置
### 1.1 创建一个`gradle-mirror.ps1`文件,粘贴以下代码:
```pwsh
param(
    # 项目根目录路径，默认当前目录
    [string]$projectPath = "."
)

# wrapper 配置文件路径
$wrapperFile = Join-Path $projectPath "gradle\wrapper\gradle-wrapper.properties"

if (-Not (Test-Path $wrapperFile)) {
    Write-Host "未找到 $wrapperFile"
    exit 1
}

# 读取文件并替换
(Get-Content $wrapperFile) `
    -replace "https\\://services\.gradle\.org/distributions", "https\://mirrors.cloud.tencent.com/gradle" `
    -replace "-bin\.zip", "-all.zip" `
    | Set-Content $wrapperFile

Write-Host "已更新 $wrapperFile"
```
### 1.2 Android Studio 外部工具(External Tools)

- 打开设置, 搜索框输入`External Tools`, 找到外部工具(External Tools)选项
- 点击右侧的 + 号新增一个工具
- 依次填写配置, `Name: Gradle Mirror` | `Program: powershell.exe`
- Arguments:`-ExecutionPolicy Bypass -File "C:\myscripts\gradle-mirror.ps1" "$ProjectFileDir$" `
- Working directory:`$ProjectFileDir$`
- 点确定保存
### 1.3 使用方法

- 在项目目录下或随便哪个文件里右键,选择刚刚创建的`Gradle Mirror`就能自动替换下载地址了

# 2.全局配置
因为我要使用kotlin,所以创建`~/.gradle/init.gradle.kts`文件,粘贴以下内容:
[从这里抄的](https://www.cnblogs.com/Chary/articles/18657340)
```kotlin
@file:Suppress("UnstableApiUsage")

import org.gradle.util.GradleVersion

val urlMappingsTencent = mapOf(
    "https://repo.maven.apache.org/maven2" to "https://mirrors.tencent.com/nexus/repository/maven-public/",
    "https://dl.google.com/dl/android/maven2" to "https://mirrors.tencent.com/nexus/repository/maven-public/",
    "https://plugins.gradle.org/m2" to "https://mirrors.tencent.com/nexus/repository/gradle-plugins/"
)

val urlMappingsAliyun = mapOf(
    "https://repo.maven.apache.org/maven2" to "https://maven.aliyun.com/repository/central/",
    "https://dl.google.com/dl/android/maven2" to "https://maven.aliyun.com/repository/google/",
    "https://plugins.gradle.org/m2" to "https://maven.aliyun.com/repository/gradle-plugin/",
)

/**
 * 启用镜像地址映射
 */
fun RepositoryHandler.enableMirror() {
    all {
        if (this is MavenArtifactRepository) {
            val originalUrl = this.url.toString().removeSuffix("/")
            urlMappingsTencent[originalUrl]?.let {
                logger.lifecycle("Repository $name [$url] is mirrored to $it")
                setUrl(it)
            }
        }
    }
}

gradle.allprojects {
    buildscript {
        repositories.enableMirror()
    }
    repositories.enableMirror()
}

gradle.beforeSettings {
    pluginManagement.repositories.enableMirror()
    // 6.8 及更高版本执行 DependencyResolutionManagement 配置
    val versionR68 = GradleVersion.version("6.8")
    if (GradleVersion.current() >= versionR68) {
        val getDrm = settings.javaClass.getDeclaredMethod("getDependencyResolutionManagement")
        val drm = getDrm.invoke(settings)
        val getRepos = drm.javaClass.getDeclaredMethod("getRepositories")
        val repos = getRepos.invoke(drm) as RepositoryHandler
        repos.enableMirror()
        println("Gradle ${gradle.gradleVersion} DependencyResolutionManagement Configured $settings")
    } else {
        println("Gradle ${gradle.gradleVersion} DependencyResolutionManagement Ignored $settings")
    }
}
```