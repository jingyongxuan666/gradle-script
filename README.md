# 起因

1. 每创建一个Project，就要配置一堆gradle的东西，需要有个模板配置
2. 每个gradle文件可以有单一职责，不希望一个gradle文件中有几百行，不能快速查找或定位需要位置，如资源混淆白名单混在build.gradle中
3. 不同module之间，依赖的第三方库版本号混乱，希望有个统一管理
4. 项目模块化后，可减少每个模块中gradle的模板代码
5. 后期可快速转为kts构建

# 优化思路

1. 使用国内仓库阿里云，加快编译速度。
2. `apply from: buildScript`统一配置plugin和project依赖的版本号
3. 利用gradle的apply from: 'xxx' 的所谓`继承`的特性，封装一些模板代码
4. 配置常用gradleTask，如依赖版本检查等

# 模块化module相关的定义

1. `app`为壳工程，直接`apply from: buildScriptApp`
2. `lib`为module，直接`apply from: buildScriptLib`

# 使用方式

1. 在项目根目录的`gradle.properties`文件中添加以下内容
   ```
   # gradle-script脚本地址
   build_script_url = http://gitlab.joyratel.com/api/v4/projects/58/repository/files/build-script.gradle/raw?ref=jsfc-0.0.1&private_token=2QgyTHT3FBpyYPqrQjA_
   
   # 加入maven仓库的地址和token
   maven_releases_url=http://gitlab.joyratel.com/api/v4/projects/46/packages/maven
   maven_releases_gitLabAccessToken=xxx

   maven_snapshots_url=http://gitlab.joyratel.com/api/v4/projects/47/packages/maven
   maven_snapshots_gitLabAccessToken=xxx

   maven_3rd_url=http://gitlab.joyratel.com/api/v4/projects/54/packages/maven
   maven_3rd_gitLabAccessToken=xxx
   ```

2. 修改project的build.gradle文件

   ```
   // Top-level build file where you can add configuration options common to all sub-projects/modules.
   buildscript {
        // 可选，可单一覆盖配置android相关版本
        ext {
            androidVersions = [
                    "compileSdk"  : 30,
                    "minSdk"      : 21,
                    "targetSdk"   : 26,
                    "gradlePlugin": "4.2.0",
                    "kotlin"      : "1.6.0",
            ]
        }
        // 以下为必选 
        // ① 导入配置文件
        apply from: build_script_url
        // ② 替换repo
        repositories config.repositories
        // ③ 替换Gradle插件依赖
        dependencies config.pluginDeps
   }
   
   // ④ 运行时仓库依赖
   allprojects {
       repositories config.repositories
       configurations.all {
           resolutionStrategy config.resolutionStrategy
           resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
       }
   }
   
   task clean(type: Delete) {
       delete rootProject.buildDir
   }
   
   // 用于jenkins打包读取，PackageTool脚本会读取该信息
   ext {
       APPLICATION_ID = "app.phoenixsky.xxx"
       APP_VERSION_CODE = 1
       APP_VERSION_NAME = '1.0.0'
   }
   ```

5. 移除setting.gradle里的`dependencyResolutionManagement`相关代码（if 7.0项目）

   ```
   // 移除
   dependencyResolutionManagement {
     repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
     repositories {
       google()
       mavenCentral()
       jcenter() // Warning: this repository is going to shut down soon
     }
   }
   ```


4. App模块可直接将build修改为以下内容

   ```
   apply from: buildScriptApp
   ```

   仅一句话app的build.gradle的配置就已经完毕，如需要添加定制各种业务逻辑和不同使用场景，可直接在当前的build.gradle文件继续添加。

   如下：

    1. 可继续添加`apply plugin`
    2. 如添加`android`部分，同名覆盖，非同名新增

   ```
   apply from: buildScriptApp
   // if you need but it's deprecated，viewbinding better
   apply plugin: 'kotlin-android-extensions'
   
   android {
   
       buildTypes{
           // debug版本的的混淆版本
           alpha {
               // 基于debug
               initWith debug
               // 用buildType的debug来兜底
               matchingFallbacks = ['debug']
   
               minifyEnabled true
               shrinkResources true
               zipAlignEnabled true
               proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
           }
       }
   }
   
   
   dependencies {
       androidTestImplementation 'xxx'
       implementation 'xxx'
   }
   ```


5. lib模块可直接将build修改为如下内容

   ```
   // 基础lib
   apply from: buildScriptLib
   
   android {
      xxx
   }
   
   dependencies {
       androidTestImplementation 'xxx'
       implementation 'xxx'
   }
   ```


   