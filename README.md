## lerna 使用

### 初始化 lerna 管理包的项目

```
lerna init
```

- 会创建 packages 目录
- git 管理
- 创建 lerna.json

### 直接创建自定义的包

- `lerna create @mose-ml-cli/core packages` => 在 packages 目录下,创建 group 组名为 mose-ml-cli 的名称为 core 的包

### 安装依赖

- 会为 packages 下的每个包安装 axios

```
lerna add axios #
```

```
lerna notice cli v4.0.0
lerna info Adding axios in 2 packages
lerna info Bootstrapping 2 packages
lerna info Installing external dependencies
lerna info Symlinking packages and binaries
lerna success Bootstrapped 2 packages
```

### 清除 packages 中包的所有依赖项

- 删除每个 packages 下的包中的依赖 lerna clean, 就把 node_modules 目录删除了,但是没有删除每个包中 package.json 中的依赖列表, 如果不需要依赖,手工删除即可

```json
"dependencies": {
    "axios": "^0.21.1"
}
```

### 针对某个特定的包安装特定的依赖

- 比如 core 这个包, 安装 axios

```
lerna add axios packages/core
```

- 如下报错是因为, core 目录下的 package.json 中的 dependencies 已经存在了 axios,需要手工删除,

- 测试这个时发现,其他的包的依赖也必须删除,即 utils/package.json 中的 dependencies 中对应的依赖包也**必须删除**

```
bogon:mose-ml-cli sanfeng$ lerna add axios packages/core
info cli using local version of lerna
lerna notice cli v4.0.0
lerna WARN No packages found where axios can be added.
```

- 针对某个包安装依赖

```
lerna add mocha packages/core -D
```

### lerna bootstrap

- 如果 packages 中每个包中的 package.json 中有依赖,如果之前 lerna clean 了(即删除了 node_modules,但是没有清空 dependencies),会自动安装依赖的库(如 axios)

### lerna link

- 自动创建 packages 中所有包之间相互依赖的软连接,如果没有依赖是没有效果的
- 按照管理, 修改 utils/lib 下的文件名 从 utils.js => index.js
  - 修改 package.json 的 main 入口文件
  ```json
      "main": "lib/utils.js",
  ```
- 手动在 core 的 package.json 中增加依赖
  ```json
  "dependencies": {
      "@mose-ml-cli/utils": "^1.0.0"
  }
  ```

> lerna add **安装的依赖是已经发布的包** > **lerna link 是连接项目中使用的包**，对于已经上线的包通过 add 安装后就不需要使用 link 安装了。

```
cd \@mose-ml-cli/
bogon:@mose-ml-cli sanfeng$ ls -l
total 0
lrwxr-xr-x  1 sanfeng  staff  14  7  3 22:08 utils -> ../../../utils
```



### lerna exec 

**--scope: 指定哪个包的范围, **

**@mose-ml-cli/core 是某个包中package.json中name属性**





```
lerna exec --scope @mose-ml-cli/core -- rm -rf node_modules
```



### lerna run 执行npm中script中的命令

```
lerna run test  # 执行所有包的npm test的命令
lerna run test @mose-ml-cli/core  # 执行指定包的
```

package.json中的配置

```json
"scripts": {
    "test": "echo \"Error: run tests from core\""
},
```



### lerna发布上线

**总结: 先commit，然后检查升级版本号 lerna version, 更新代码到github上， 登录npm，最后lerna publish， **

1. **lerna version**

- 每次发布上线，都希望package.json中的version都需要增加一位 1.0.1

```
 通过使用lerna version 增加版本号
 需要将代码提交
λ lerna version
lerna notice cli v4.0.0
lerna info current version 1.0.0
lerna info Assuming all packages changed
? Select a new version (currently 1.0.0) (Use arrow keys)
> Patch (1.0.1)
  Minor (1.1.0)
  Major (2.0.0)
  Prepatch (1.0.1-alpha.0)
  Preminor (1.1.0-alpha.0)
  Premajor (2.0.0-alpha.0)
  Custom Prerelease
  Custom Version
```

因为1.0.0还没有创建，暂时先不创建1.0.1这个版本



2. **lerna changed**

和上个版本相比有哪些版本发生了变更

```
lerna changed
lerna notice cli v4.0.0
lerna info Assuming all packages changed
@mose-ml-cli/core
@mose-ml-cli/utils
lerna success found 2 packages ready to publish
```

3. **lerna diff**

```
λ lerna diff
lerna notice cli v4.0.0
diff --git a/packages/core/package.json b/packages/core/package.json
index ca9808b..01b2e6d 100644
--- a/packages/core/package.json
+++ b/packages/core/package.json
@@ -17,7 +17,7 @@
     "registry": "https://registry.npm.taobao.org"
   },
   "scripts": {
-    "test": "echo \"Error: run tests from root\" && exit 1"
+    "test": "echo \"Error: run tests from core\""
   },
   "dependencies": {
     "@mose-ml-cli/utils": "^1.0.0",
diff --git a/packages/utils/package.json b/packages/utils/package.json
index 1e1d488..fc61628 100644
```

4. **如果npm配置了淘宝源,就行修改即可**

```
npm config set register https://registry.npmjs.org/
```

5. **修改每个包的package.json中的config**

```
"publishConfig": {
	"registry": "https://registry.npmjs.org"
},
```

6. npm上, 组group是私有的属性，需要配置位公开的 

package.json

```json
"publishConfig": {
    "registry": "https://registry.npmjs.org",
    "access": "public"
},
```

7. 配置一下脚手架的命令，在某个包下的 package.json 

```
"bin": {
	"mose-ml-cli": "bin/index.js"
},
```

