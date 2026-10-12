# Android ART兼容补丁

## 问题

Pixel 3 Android12（SDK31，arm64）当前Demo使用Maven官方3.0.0时，10秒录制（包括-r重启）产生0个方法样本。该设备libart.so的WalkStack符号包含额外模板布尔参数，SDK仅查旧符号时抓栈初始化失败。内部空区间错误码8被strerror翻译成Exec format error，不能作为可执行文件格式错误判断。

## 方案

在本克隆当前源码上重新实现，没有复制旧版Java文件。上游基线提交为2ae621f2d84ad54b2811d675d86ba2815006adb3。

- StackVisitor.cpp：旧WalkStack符号优先，缺失时查Android12模板符号；回退成功和初始化失败分别记录诊断日志。
- HttpServer.java：导出完成标记改为volatile，保证收集线程对HTTP线程可见；等待循环即使超时为0也先检查已完成状态。
- 当前克隆已经在waitForDataReady动态读取等待超时，保持该实现，未引入Maven3.0.0旧版的构造期缓存字段。

## 关键决策

不修改版本号、Maven坐标、发布脚本或CLI协议，不添加人为方法样本。补丁只在原符号缺失时回退，不能据此断言所有Android12或此前成功APK都不兼容。

宿主Python中的128MiB系统缓冲、每秒flush、--full-sort和SDK日志采集属于观测工具，不放进SDK。用户后续发布远程依赖后，再替换MrcdDependencies的临时btrace-compat依赖；当前不引用未发布版本。

## 验证与适用边界

之前的临时兼容SDK在该设备Case19/Step2完成60秒正式录制：PID13662，Trace249568692 bytes，573537应用切片、97554调度片段，导出/分析完成、收尾错误为空，health无error/data_loss。证据位于MrcdDependencies/SVGAPlayer/experiment-results/20261012-151651-Case19。

这证明兼容方案可行，不代表本克隆的完整发布产物已验收。当前克隆需重新构建并验证ABI及远程发布版本；不复用旧APK结论为新版本打包或上传成功。

### 本次迁移验证

当前克隆Java及Native源码已通过源码构建验证：使用MrcdDependencies可用的AGP/Gradle环境，临时sourceSets与CMake入口指向本克隆，生成arm64 Debug AAR，BUILD SUCCESSFUL。只借用构建环境和无package属性的测试宿主Manifest，没有修改本仓库构建/发布文件，也没有安装该新AAR或进行真机验证。

原仓库:rhea-inhouse:assembleRelease在Gradle6.7.1阶段被本机全局初始化脚本的GradleVersion未解析错误阻断，尚未执行模块编译。日志保存在btrace-android/build/compatibility-check（由Gradle构建产物规则管理）；不能将源码构建成功表述为原发布任务成功。

发布前确认POM_GROUP_ID与POM_VERSION_NAME。本次保持原样；检查时POM_VERSION_NAME字段仍为3.0.0，不替用户设定远程3.1.x版本或上传。
