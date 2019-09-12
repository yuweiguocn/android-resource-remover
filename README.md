
# android-resource-remover
一个根据lint结果自动化清理无用资源的python脚本工具。


## 使用lint检查无用资源
通常我们可以使用lint检查工程内无用资源，但执行lint命令发现只检查了壳工程的无用资源，并没有检查子Module的无用资源。

针对这个问题可以使用下面的配置，对应用的依赖也执行检查：

lintOptions {
    checkReleaseBuilds false
    abortOnError false
    checkDependencies true //对依赖的资源也执行检查，注意也会对引用aar检查，如果是子module打开源码依赖即可
    check "UnusedResources" //只检查无用资源，提升执行速度
}
然后执行lint命令进行检查无用资源：
```
./gradlew lint
```

然后我们可以在build下的reports目录下找到lint的结果。


## 配置

首先将本工程clone到你的项目工程的根目录中，然后根据你项目使用反射获取的资源在resouceCleanConfig.json文件中进行配置：
```
{
	"projects": [
		"/app/",
		"/push/"
	],
	"pathIncludes": [
		".jpg",
		".png",
		".xml"
	],
	"pathExcludes": [
		"------common-config-start--------",
		"/layout/"
	]
}

```
- projects：表示要清理无用资源的路径需要包含的子Module工程名称，因为依赖检查也会对引用的所有aar进行检查，所以我们需要指定打开源码的子Module的名称。
- pathIncludes：表示要清理的无用资源的路径需要匹配的规则。
- pathExcludes：表示要清理的无用资源的路径不能包含的字符串，也就是白名单。

以上三个条件为并列条件，只有同时满足才会被清理。上述json配置的文件表示清理app和push Module下文件后缀为.jpg或.png或xml的资源，但不删除布局文件。

## 白名单

目前发现工程使用代码获取资源的方法有以下两种，代码中使用下列方法获取的图片名称需要添加配置文件白名单中。
```
//第一种
item.resId = mContext.getResources().getIdentifier(name, "drawable", mContext.getPackageName());
//第二种
public static Integer getResourceDrawableId(String resource) {
    if(TextUtils.isEmpty(resource)){
        return -1;
    }
    try {
        String name = resource.trim();
        Field field = R.drawable.class.getField(name);
        return field.getInt(null);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return -1;
}
```

建议对新增的资源在使用代码获取时对名称添加统一前缀`reflect_`。

## 自动化

> 此步骤请谨慎操作，删除的资源无法恢复，使用前请添加版本控制。

配置好后执行如下命令自动清理无用资源：
```
python android_clean_app.py --xml ../app/build/reports/lint-results.xml
```
参数`--xml`指定为你的lint结果文件的路径。


## 解决打包失败问题
lint的检测结果包含无用代码对资源的引用，如果只是将无用资源删除后可能会引起打包失败。
针对这个问题我们会自动添加一个unused_ids.xml文件保存删除的资源id，这可以保证不会由于删除无用资源引起的打包失败。


## Licence
Apache version 2.0
