# React Eslint 报错解决合集

## 提示错误：'React' must be in scope when using JSX react/react-in-jsx-scope

错误提示：在JSX作用域中使用JSX React/React时，“React”必须在作用域中。是因为React并未导入
此时要把：

```javascript
import { Component } from "react";
```

改为：
```javascript
import React,{ Component } from "react";
```
