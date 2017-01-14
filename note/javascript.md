# 易错点

### 防止空指针
```javascript
if (a && a.b && a.b.c) {
	var result = a.b.c.d;
}
```

