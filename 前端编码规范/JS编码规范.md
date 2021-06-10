| 版本      | 负责人 | 日期       | 内容     |
| --------- | ------ | ---------- | -------- |
| V0.1/草稿 | 李政29 | 2021-06-07 | 创建文档 |

# JS编码规范

eslint配置为：[eslint:recommended](https://cn.eslint.org/docs/rules/)。

## 自定义

```js
    // 启用的规则及其各自的错误级别：
    // 'off' | 0    - 关闭规则
    // 'warn' | 1   - 开启规则，使用警告级别的错误：warn (不会导致程序退出)
    // 'error' | 2  - 开启规则，使用错误级别的错误：error (当被触发的时候，程序会退出)
    rules: {
        'indent': [2, 4], // 强制使用4个空格
        'no-shadow-restricted-names': 0, // 禁止将标识符定义为受限的名字
        'no-prototype-builtins': 0 // 禁止直接调用 Object.prototypes 的内置属性
    }
```

