## lerna 使用

### 初始化lerna管理包的项目
```
lerna init
```
- 会创建packages目录
- git管理
- 创建lerna.json

### 直接创建自定义的包
- `lerna create @mose-ml-cli/core packages` => 在packages目录下,创建group组名为mose-ml-cli的名称为core的包


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
- 手动在core的package.json中增加依赖
    ```json
    "dependencies": {
        "@mose-ml-cli/utils": "^1.0.0"
    }
    ```

> lerna add **安装的依赖是已经发布的包**
> **lerna link是连接项目中使用的包**，对于已经上线的包通过add 安装后就不需要使用link安装了。

```
cd \@mose-ml-cli/
bogon:@mose-ml-cli sanfeng$ ls -l
total 0
lrwxr-xr-x  1 sanfeng  staff  14  7  3 22:08 utils -> ../../../utils
```

