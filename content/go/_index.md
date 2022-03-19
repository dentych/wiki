---
title: Go
weight: 4
---

### Range problem
If you range over a slice and append to another slice, sometimes all the values are equal.
This is an example of how to make that problem appear.

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