# JNI

### 数据类型



| Java 类型           | JNI 类型      | C/C++ 类型                         | JVM描述符       |
| ------------------- | ------------- | ---------------------------------- | --------------- |
| boolean             | jboolean      | unsigned char (无符号 8 位整型)    | Z               |
| byte                | jbyte         | char (有符号 8 位整型)             | B               |
| char                | jchar         | unsingned short (无符号 16 位整型) | C               |
| short               | jshort        | short (有符号 16 位整型)           | S               |
| int                 | jint          | int (有符号 32 位整型)             | I               |
| long                | jlong         | long (有符号 64 位整型)            | J               |
| float               | jfloat        | float (有符号 32 位浮点型)         | F               |
| double              | jdouble       | double (有符号 64 位双精度型)      | D               |
| void                |               |                                    | V               |
| 其它引用类型        |               |                                    | L + 全类名 + ； |
| type[]              |               |                                    | [               |
| method type         |               |                                    | (参数)返回值    |
| Java.lang.Class     | jclass        |                                    |                 |
| Java.lang.Throwable | jthrowable    |                                    |                 |
| Java.lang.String    | jstring       |                                    |                 |
| Other object        | jobject       |                                    |                 |
| Java.lang.Object[]  | jobjectArray  |                                    |                 |
| boolean[]           | jbooleanArray |                                    |                 |
| byte[]              | jbyteArray    |                                    |                 |
| char[]              | jcharArray    |                                    |                 |
| short[]             | jshortArray   |                                    |                 |
| int[]               | jintArray     |                                    |                 |
| long[]              | jlongArray    |                                    |                 |
| float[]             | jfloatArray   |                                    |                 |
| double[]            | jdoubleArray  |                                    |                 |
| Other arrays        | jarray        |                                    |                 |

### 环境配置

- NDK：这套工具集允许为 Android 使用 C 和 C++ 代码。

- CMake：一款外部构建工具，可与 Gradle 搭配使用来构建原生库。如果只计划使用 ndk-build，则不需要此组件。

- LLDB：debug 调式。

  > File —— Setting —— SDK Tools 里勾选安装NDK、CＭake、LLDB选项