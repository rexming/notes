# 需求，拿官方demo来说，按照岁数从小到大，而且当岁数相同，John排在Bob前面

```
type ByAge []Person

func (a ByAge) Len() int      { return len(a) }
func (a ByAge) Swap(i, j int) { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool {
	if a[i].Age == a[j].Age {
		if a[i].Name == "Bob" && a[j].Name == "John" {
			return false
		}
		return true
	} else {
		return a[i].Age < a[j].Age
	}
}

func main() {
	
	people := []Person{
		{"Bob", 31},
		{"John", 31},
		{"Michael", 17},
		{"Jenny", 26},
	}
	
	fmt.Println(people)
	sort.Sort(ByAge(people))
	fmt.Println(people)
	
}
上面是正确输出
但是当Less()这样写时，却无法得到正确排序结果，表现在Bob始终在John前面
func (a ByAge) Less(i, j int) bool {
	if a[i].Age == a[j].Age && a[i].Name == "Bob" && a[j].Name == "John" {
		return false
		
	} else {
		return a[i].Age < a[j].Age
	}
}
```