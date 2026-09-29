# `index` 在不同位置的含义

## 三个 `index` 各司其职

| 位置 | 写法 | 含义 |
|---|---|---|
| `urls.py` | `name="index"` | 路由的名字 |
| `urls.py` | `views.index` | 视图**函数** `def index(request)` |
| `templates/` | `{% url 'index' %}` | 引用路由名字，生成 URL |

## 完整链路

```
模板里写 {% url 'index' %}
        ↓
Django 去 urls.py 找 name="index" 的路由
        ↓
找到 path("", views.index, name="index")
        ↓
提取对应的 URL 路径 ""
        ↓
加上前缀（如果有 include）
        ↓
生成最终 URL
```

## 关键理解

- `{% url 'index' %}` 只关心路由的名字 → 生成 URL 路径
- 浏览器访问这个 URL 后，Django 根据 URL 找到对应的视图函数 `views.index` 去执行
- 视图函数再决定渲染哪个模板（如 `index.html`）返回

**`{% url 'index' %}` 跟 `index.html` 文件没有直接关系。** 它只关心路由名字，不关心视图里渲染什么模板。即使视图函数改成渲染别的模板，`{% url 'index' %}` 照常工作。

## 路由定义示例

```python
path("", views.index, name="index")  # 空地址指向 index 视图
```
