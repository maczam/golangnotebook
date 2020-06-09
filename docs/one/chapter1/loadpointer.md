# StorePointer&LoadPointer

在Go语言中，原子包提供较低级的原子内存，有助于实现同步算法。Go语言中的LoadPointer()函数用于自动加载*addr。这个函数是在原子包下定义的。在这里，您需要导入“同步/原子”和“不安全”包来使用这些函数。



## 语法

```go

  func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)
```



## 例子


```go

func main() {

	data := "hello word!"

	var dataPointer = (*unsafe.Pointer)(unsafe.Pointer(&data)) //定义指针
	fmt.Println(fmt.Sprintf("dataPointer>>%v,%s", dataPointer, *(*string)(unsafe.Pointer(dataPointer))))

	var datax string
	var dataxPointer = (*unsafe.Pointer)(unsafe.Pointer(&datax)) //定义指针
	atomic.StorePointer(dataxPointer, unsafe.Pointer(&data))     //将数据的指针，保存到新指针上
	fmt.Println(fmt.Sprintf("dataxPointer>>%v,%s", dataxPointer, datax))

	px1 := (*string)(atomic.LoadPointer(dataxPointer)) // 获取指针
	fmt.Println(fmt.Sprintf("px1>>%v,%s", unsafe.Pointer(px1), *(*string)(px1)))
	fmt.Println(px1 == &data)
}

```



## 输出

```
dataPointer>>0xc00000e1e0,hello word!
dataxPointer>>0xc00000e210,
px1>>0xc00000e1e0,hello word!
true
```