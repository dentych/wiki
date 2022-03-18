---
title: Go
weight: 4
---

The range problem:

```go
package main

import "fmt"

type Person struct {
	Name *string
}

func main() {
	people := []Person{
		{Name: ptr("Jan")},
		{Name: ptr("Waddahell")},
		{Name: ptr("Mie")},
	}

	var otherPeople []*Person
	for _, v := range people {
		otherPeople = append(otherPeople, &v)
	}

	for _, v := range otherPeople {
		fmt.Println(*v.Name)
	}
}

func ptr(str string) *string {
	return &str
}
```