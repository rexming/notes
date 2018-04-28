# golang模版
go主要提供text/template 和html/template 解析库，基本接口都是相同的，主要是html/template 主要提供转义功能，用于html , 防止前端代码劫持。（比如<script>alert("Hi!");</script>如果不转义会在浏览器中直接被执行，这样很危险）

##  示例
简单的替换

```
demo.html:
<h1>Hello, {{.Name}}!</h1>

main.go:

func main() {
	t, err := template.ParseFiles(`src/demo.html`)
	if err != nil {
		panic(err)
	}
	if err!= nil {
		panic(err)
	}
	
	data := struct {
		Name string
	}{"Ming"}
	
	t.Execute(os.Stdout,data)
}

output:
      <h1>Hello, Ming!</h1>

```
复杂一点的示例：

```
demo.html:
{{.Title}}
{{.HTML}}
{{.SafeHTML}}
{{.}}


<a title="{{.Title}}">
<a title="{{.HTML}}">

<a href="{{.HTML}}">
<a href="?q={{.HTML}}">
<a href="{{.Path}}">
<a href="?q={{.Path}}">

<!-- Encoding even works on non-string values! -->
<script>
  var dog = {{.Dog}};
  var map = {{.Map}};
  doWork({{.Title}});
</script>

main.go:
type Test struct {
	HTML     string
	SafeHTML template.HTML
	Title    string
	Path     string
	Dog      Dog
	Map      map[string]string
}

type Dog struct {
	Name string
	Age  int
}

func main() {
	t, err := template.ParseFiles("context.gohtml")
	if err != nil {
		panic(err)
	}

	data := Test{
		HTML:     "<h1>A header!</h1>",
		SafeHTML: template.HTML("<h1>A Safe header</h1>"),
		Title:    "Backslash! An in depth look at the \"\\\" character.",
		Path:     "/dashboard/settings",
		Dog:      Dog{"Fido", 6},
		Map: map[string]string{
			"key":       "value",
			"other_key": "other_value",
		},
	}

	err = t.Execute(os.Stdout, data)
	if err != nil {
		panic(err)
	}
}

output:
1.Backslash! An in depth look at the &#34;\&#34; character.
&lt;h1&gt;A header!&lt;/h1&gt;
<h1>A Safe header</h1>
{&lt;h1&gt;A header!&lt;/h1&gt; &lt;h1&gt;A Safe header&lt;/h1&gt; Backslash! An in depth look at the &#34;\&#34; character. /dashboard/settings {Fido 6} map[key:value other_key:other_value]}

6 <a title="Backslash! An in depth look at the &#34;\&#34; character.">
7 <a title="&lt;h1&gt;A header!&lt;/h1&gt;">

9 <a href="%3ch1%3eA%20header!%3c/h1%3e">
10 <a href="?q=%3ch1%3eA%20header%21%3c%2fh1%3e">
11 <a href="/dashboard/settings">
12 <a href="?q=%2fdashboard%2fsettings">

<script>
  var dog = {"Name":"Fido","Age":6};
  var map = {"key":"value","other_key":"other_value"};
  doWork("Backslash! An in depth look at the \"\\\" character.");
</script>
    
```
分析输出的前几行，解析到模版的任何字符都需要先进行编码包括<与>,然后正确渲染，注意到template.HTML类型的之没有被编码，这种类型代码跳过编码（所以尽量避免这种情况，防止注入）

注意第6,7行与9，10的编码会有一点不同，其原因是在href 中，当正斜杠时作为路径不会被编码，当作为查询参数不会编码。

golang 模版有两个常用的传入参数的类型。一个是struct，还有一个是map[string]interface{},这两个类型会等价为json object

当嵌套时：比如结构体嵌套map,我们想直接取如上例子中的dog的Name,则在模版中这样取{{.Dog.Name}},便可以取到其值。而当Name 为非导出字段时，不能单独取
同理适用于map类型：
例：

```
var data2 = map[string]interface{}{"s": struct {
		Name string
		age int
	}{"zhang",24}}
	err = t.Execute(os.Stdout, data2)

{{.s}} //output： {zhang 24} 带了一个中括号
{{.s.Name}} //output：zhang
{{.s.age}} //output:panic  age is an unexported field of struct type interface {}
```

模版语法:
## if/else block
里面可以是bool类型或字符串，空字符串为false

```
demo.html
<h1>Hello, {{if .Name}} {{.Name}} {{else}} there {{end}}!</h1>

去除空格：
1.手动删除模版空格
2.适用-来消除多余空格
<h1>
  Hello,
  {{if .Name}}
    {{.Name}}
  {{- else}}
    there
  {{- end}}!
</h1>
````
## Range blocks

用来遍历数组类型

```
main.go
package main

import (
	"html/template"
	"net/http"
)

var testTemplate *template.Template

type Widget struct {
	Name  string
	Price int
}

type ViewData struct {
	Name    string
	Widgets []Widget
}

func main() {
	var err error
	testTemplate, err = template.ParseFiles("src/demo.html")
	if err != nil {
		panic(err)
	}
	
	http.HandleFunc("/", handler)
	http.ListenAndServe(":3000", nil)
}

func handler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "text/html")
	
	err := testTemplate.Execute(w, ViewData{
		Name: "John Smith",
		Widgets: []Widget{
			Widget{"Blue Widget", 12},
			Widget{"Red Widget", 12},
			Widget{"Green Widget", 12},
		},
	})
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

demo1.html
{{range .Widgets}}
  <div class="widget">
    <h3 class="name">{{.Name}}</h3>
    <span class="price">${{.Price}}</span>
  </div>
{{end}}

output:
Blue Widget
$12
Red Widget
$12
Green Widget
$12

-----------------------------
demo2.html
{{range .Widgets}}
  <div class="widget">
    <h3 class="name">{{.}}</h3>
    <span class="price">${{.}}</span>
  </div>
{{end}}

output:
{Blue Widget 12}
${Blue Widget 12}
{Red Widget 12}
${Red Widget 12}
{Green Widget 12}
${Green Widget 12}
```
特别的是当插入<pre>{{.}}</pre>。dot 还是代表go中的变量

```
{{range .Widgets}}
<div class="widget">
    <pre>{{.}}</pre>
    <h3 class="name">{{.Name}}</h3>
    <span class="price">${{.Price}}</span>
</div>
{{end}}
output:
{Blue Widget 12}
Blue Widget
$12
{Red Widget 12}
Red Widget
$12
{Green Widget 12}
Green Widget
$12
```
## 模版嵌套
定义和使用一个模版

```
定义
footer.html
{{define "footer"}}
...
{{end}}

使用
在index.html添加即可引用
{{template "footer"}}


```
footer称为子模版，index.html 称为父模版，子模版中可 获得父模版中变量。
{{template "footer" .}}/{{template "footer" .Attr}}
便获取了父模版的变量
代码实例

```
{{define "widget-header"}}
  <h3 class="name">{{.}}</h3>
{{end}}

{{range .Widgets}}
  <div class="widget">
    {{template "widget-header" .Name}}
    <span class="price">${{.Price}}</span>
  </div>
{{end}}
```
.Name 属性传给了widget-headher 模版中的dot
而且可以多层传递

```
{{define "widget"}}
<div class="widget">
{{template "widget-header" .Name}}
<span class="price">${{.Price}}</span>
</div>
{{end}}

{{define "widget-header"}}
z<h3 class="name">{{.}}</h3>
{{end}}

{{range .Widgets}}
  {{template "widget" .}}
{{end}}

output:
Blue Widget
$12
Red Widget
$12
Green Widget
$12

```
# 使用函数函数变量




