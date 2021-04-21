

### 1.创建
```
countryMap = make(map[string]string)
```

### 2.插入

```
/* map 插入 key-value  */
countryMap["ShengRI"] = "1988-04-09"
```

### 3.取值
```
/* 使用 key 输出 map 值 */
for country := range countryMap {
fmt.Println("Capital of",country,"is",countryMap[country])
}

```

### 4.key是否存在
```
/* 查看元素在集合中是否存在 */
captial, ok := countryMap["ShengRI"]
/* 如果 ok 是 true, 则存在，否则不存在 */
if(ok){
   fmt.Println("Capital of United States is", captial)
} else {
   fmt.Println("is not ok")
}
```