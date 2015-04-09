# hadoop-2.6.0-windows

**TLDR: mine binaries are published; unless you trust it read on what were causing build errors**

To use hadoop (at least its client from MS Windows machine) one is directed to https://wiki.apache.org/hadoop/Hadoop2OnWindows (kind of official documentation). The problem is lack of native libraries for Windows at Apache website. For hadoop 2.2.0 such binaries are https://github.com/srccodes/hadoop-common-2.2.0-bin and they don't work with Hadoop 2.6.0. So my solution was to compile it myself. I have followed BUILDING.txt instructions and guess what, show-stopping build errors. 

## Pre-requisites:
1. Fresh Windows 7 VM **without** any MS Visual Studion
  * **Important**: Uninstall MS VS 2010 Redistributables both for x86 and 64-bit. Otherwise Microsoft's installer will fail with obscure error (read Samples.html bla-bla)
2. install oracle java: get and run jdk-7u75-windows-x64.exe
 * no src, no public jre, into C:\jdk7
 * edit system environment variables, add JAVA_HOME with value C:\jdk7
3. install maven: apache-maven-3.0.3-bin.zip
 * cd C:\
 * "C:\Program Files\7-Zip\7z.exe" x C:\Users\you\Uploads\apache-maven-3.0.3-bin.zip
4. install protoc: protoc-2.5.0-win32.zip
 * mkdir C:\UTILS
 * cd C:\UTILS
 * "C:\Program Files\7-Zip\7z.exe" x C:\Users\you\Uploads\protoc-2.5.0-win32.zip
5. expand Hadoop sources into short path
 * mkdir C:\h260w7
 * cd C:\h260w7
 * "C:\Program Files\7-Zip\7z.exe" x C:\Users\you\Uploads\hadoop-2.6.0-src.tar.gz
 * "C:\Program Files\7-Zip\7z.exe" x hadoop-2.6.0-src.tar
 * move C:\h260w7\hadoop-2.6.0-src\* ..
6. Get and run winsdk_web.exe (http://www.microsoft.com/en-us/download/details.aspx?id=8279)
 * specify C:\VS71
 * unselect samples
 * **Important**: .NET tools are mandatory, will be used.
 * **Important**: Install all this at once. Repeated attempt to add missing component will have **disabled** wrong installation path (C:\Program Files\...) and will overwrite C:\VS71 so your install will be essentially broken.
 * *Remark: seems Microsoft QA team for MSVS product was lacking, and developers too*
7. Get and run Cygwin installer (http://cygwin.org/setup-x86.exe)
 * specify C:\cygwin and download into C:\cygwin-dl
 * just installing default base is enough
8. edit environment variable PATH
 * append to the end: ;C:\jdk7\bin;C:\apache-maven-3.0.3\bin;C:\cygwin\bin;C:\UTILS
9. Find in start menu and run "Windows SDK 7.1 Command Prompt"
 * cd C:\h260w7
 * set Platform=x64
10. Finally
 * mvn package -Pdist,native-win -DskipTests -Dtar -Dmaven.javadoc.skip=true

## My experience
```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
... bla-blah ...
"C:\h260w7\hadoop-common-project\hadoop-common\src\main\winutils\winutils.vcxproj" (default target) (4) ->
(Link target) ->
  LINK : fatal error LNK1123: failure during conversion to COFF: file invalid or corrupt [C:\h260w7\hadoop-common-project\hadoop-common\src\main\winutils\winutils.vcxproj]

```

The reason: "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/winutils_msg.res" is not COFF. Why link.exe attempted and why failed? That's the question to MS C gurus.

## My solution
```
C:\h260w7> "C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\bin\amd64\link.exe" /ERRORREPORT:QUEUE ^
  /OUT:"C:\h260w7\hadoop-common-project\hadoop-common\target/bin/winutils.exe" ^
  /INCREMENTAL:NO /NOLOGO kernel32.lib user32.lib gdi32.lib winspool.lib comdlg32.lib advapi32.lib shell32.lib ^
  ole32.lib oleaut32.lib uuid.lib odbc32.lib odbccp32.lib  "C:\h260w7\hadoop-common-project\hadoop-common\target\bin\libwinutils.lib" ^
  /MANIFEST /ManifestFile:"C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/winutils.exe.intermediate.manifest" ^
  /MANIFESTUAC:"level='asInvoker' uiAccess='false'" ^
  /DEBUG /PDB:"C:\h260w7\hadoop-common-project\hadoop-common\target\bin\winutils.pdb" ^
  /SUBSYSTEM:CONSOLE /OPT:REF /OPT:ICF /LTCG /TLBID:1 /DYNAMICBASE /NXCOMPAT ^
  /IMPLIB:"C:\h260w7\hadoop-common-project\hadoop-common\target/bin/winutils.lib" ^
  /MACHINE:X64 ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/hadoopwinutilsvc_s.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/service.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/readlink.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/symlink.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/systeminfo.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/chmod.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/chown.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/groups.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/hardlink.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/task.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/ls.obj" ^
 "C:\h260w7\hadoop-common-project\hadoop-common\target/winutils/main.obj"
```

then 

```
C:\h260w7> msbuild C:\h260w7\hadoop-common-project\hadoop-common\src\main\native\native.sln
```

Get results from C:\h260w7\hadoop-common-project\hadoop-common\target\bin\
