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

