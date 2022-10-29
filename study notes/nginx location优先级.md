# Nginx Location 路径匹配优先级

Nginx 服务器需要对不同的请求 uri 执行不同的操作，基本的配置语法为：

```nginx
location [=|~|~*|^~]  uri  { 
	...
}
```

## 一、精确匹配

语法示例：
```nginx
location = /static/img/file.jpg {
	...
}
```

## 二、前缀匹配

### 1、普通前缀匹配

语法示例：
```nginx
location /static/img/ {
	...
}
```

### 2、优先前缀匹配

语法示例：
```nginx
location ^~/static/img/ {
	...
}
```

## 三、正则匹配

### 1、区分大小写

语法示例：
```nginx
location ~ /static/img/.*\.jpg$ {
	...
}
```

### 2、不区分大小写

语法示例：
```nginx
location ~* /static/img/.*\.jpg$ {
	...
}
```

### 3、区分大小写取反

语法示例：
```nginx
location !~ /static/img/.*\.jpg$ {
	...
}
```

### 4、不区分大小写取反

语法示例：
```nginx
location !~* /static/img/.*\.jpg$ {
	...
}
```

## 四、优先级

对于请求： http://example.com/static/img/logo.jpg

### 1、命中精确匹配

```nginx
location = /static/img/logo.jpg {
	...
}
```

** 则优先精确匹配，并终止匹配。

### 2、命中多个前缀匹配

```nginx
location /static/ {
	...
}
location /static/img/ {
	...
}
```

** 则记住最长的前缀匹配，即上例中的 /static/img/，并继续匹配

### 3、最长的前缀匹配是优先前缀匹配

```nginx
location /static/ {
	...
}
location ^~ /static/img/ {
	...
}
```

** 则命中此最长的优先前缀匹配，并终止匹配

### 4、命中多个正则匹配

```nginx
location /static/ {
	...
}
location /static/img/ {
	...
}
location ~* /static/ {
	...
}
location ~* /static/img/ {
	...
}
```

** 则忘记上述 2 中的最长前缀匹配，使用第一个命中的正则匹配，即上例中的 location ~* /static/ ，并终止匹配（命中多个正则匹配，优先使用配置文件中出现次序的第一个），否则，命中上述 2 中记住的最长前缀匹配