# yeb

[简体中文](README_CN.md) | [English](README.md)

> Golang Web Development Framework
>
>> Note: This project is built for learning purposes and is not recommended for use in production environments.

## Features

- Prefix tree routing
- Group control & middleware
- Template rendering & static file handling
- Error handling
- Logging

## Example

```go
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
	"time"

	"github.com/yaragos/yeb"
)

type student struct {
	Name string
	Age  int8
}

func main() {
	// Create a yeb engine
	r := yeb.Default()

	// Route matching
	r.GET("/v0/*path", func(c *yeb.Context) {
		c.JSON(http.StatusOK, yeb.H{"path": c.Param("path")})
	})

	// Define route groups
	v1 := r.Group("/v1")
	{
		v1.GET("/", func(c *yeb.Context) {
			c.String(http.StatusOK, "hello, this is the index page of group v1")
		})

		v1.GET("/hello", func(c *yeb.Context) {
			// expect /hello?name=yara
			c.String(http.StatusOK, "hello %s, you're at %s\n", c.Query("name"), c.Path)
		})
	}

	v2 := r.Group("/v2")
	// Use middleware
	v2.Use(middlewareV2())
	{
		v2.GET("/", func(c *yeb.Context) {
			c.String(http.StatusOK, "hello, this is the index page of group v2")
		})

		v2.GET("/hello/:name", func(c *yeb.Context) {
			// expect /hello/yara
			c.String(http.StatusOK, "hello %s, you're at %s\n", c.Param("name"), c.Path)
		})

		v2.POST("/login", func(c *yeb.Context) {
			c.JSON(http.StatusOK, yeb.H{
				"username": c.PostForm("username"),
				"password": c.PostForm("password"),
			})
		})
	}

	// Template rendering
	r.SetFuncMap(template.FuncMap{
		"FormatAsDate": FormatAsDate,
	})
	r.LoadHTMLGlob("templates/*")
	r.Static("/assets", "./static")
	stu1 := &student{Name: "Alice", Age: 18}
	stu2 := &student{Name: "Bob", Age: 20}
	r.GET("/students", func(c *yeb.Context) {
		c.HTML(http.StatusOK, "arr.tmpl", yeb.H{
			"title":  "yeb",
			"stuArr": [2]*student{stu1, stu2},
		})
	})
	// index out of range for testing Recovery()
	r.GET("/panic", func(c *yeb.Context) {
		names := []string{"yara"}
		c.String(http.StatusOK, names[100])
	})

	r.Run(":8080")
}

func middlewareV2() yeb.HandlerFunc {
	return func(c *yeb.Context) {
		log.Printf("middlewareV2 before")
		log.Printf("path=%s, method=%s", c.Path, c.Method)
		c.Next()
		log.Printf("middlewareV2 after")
	}
}

func FormatAsDate(t time.Time) string {
	year, month, day := t.Date()
	return fmt.Sprintf("%d-%02d-%02d", year, month, day)
}
```

## Thanks

- [7天用Go从零实现Web框架Gee教程 | 极客兔兔](https://geektutu.com/post/gee.html)

- [GitHub - gin-gonic/gin](https://github.com/gin-gonic/gin)
