## JSON encode扩展
基于`json`标签扩展encode规则，实现struct动态编码为不同JSON结构`字段的取舍`

- golang struct转json提供了字段名、字段筛选`-`、空值忽略`omitempty`等标签，使用中对于选择性忽略部分字段的支持较弱
- 可以定义新struct或者用map，实现自己想要的不同json结构
- 在这里尝试扩展json tag规则的方式来支持同一struct解析为不同的json结构
- 目前仅做了`动态的字段取舍`的规则，在此基础上可以进一步扩展，如多种字段名定义等

### `json`标签格式
```go
filter:[{preKey1}.{subKey1};{preKey2}.{subKey2}]

`json:"f,filter:preKey1.subKey1;preKey2;preKey2"`
`json:"f,filter:*.subKey"`
`json:"f,filter:preKey.*"`
`json:"f,filter:*.*"`
```

## 示范
#### struct
```go
type A struct {
	F  string `json:"f,filter:*"`
	F1 string `json:"f_1,filter:a1"`
	F2 string `json:"f_2,filter:a2"`

	B  B `json:"b,filter:*.*"`
	B1 B `json:"b_1,filter:*.b1"`
	B2 B `json:"b_2,filter:a2.b2"`
	B3 B `json:"b_3,filter:a1.*;a2.a2"`
}

type B struct {
	F  string `json:"f,filter:*"`
	F1 string `json:"f_1,filter:b1"`
	F2 string `json:"f_2,filter:b2"`

	C C `json:"c,filter:*.*"`
	C1 C `json:"c_1,filter:b1.*"`
	C2 C `json:"c_3,filter:*.b1"`
}

type C struct {
	F  string `json:"f,filter:*"`
	F1 string `json:"f_1,filter:b1"`
	F2 string `json:"f_2,filter:b2"`
}
```

#### [""，不过滤](#json)
```go
json.MarshalFilterIndent(i, "", "", "\t")
```

#### ["*"，过滤，A全部](#json-1)
```go
json.MarshalFilterIndent(i, "*", "", "\t")
```

#### ["a1"，过滤，只要a1](#a1json)
```go
json.MarshalFilterIndent(i, "a1", "", "\t")
```

#### ["a2"，过滤，只要a1](#a2json)
```go
json.MarshalFilterIndent(i, "a2", "", "\t")
```

#### [echo-web Demo](https://github.com/hb-go/echo-web/blob/master/router/api/json.go)
```
http://{API_HOST}/json/encode
http://{API_HOST}/json/encode?filter=*
http://{API_HOST}/json/encode?filter=a1
http://{API_HOST}/json/encode?filter=a2
```

##### ""json
```json
{
	"f": "",
	"f_1": "",
	"f_2": "",
	"b": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": "",
			"f_2": ""
		}
	},
	"b_1": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": "",
			"f_2": ""
		}
	},
	"b_2": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": "",
			"f_2": ""
		}
	},
	"b_3": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": "",
			"f_2": ""
		}
	}
}
```
##### "*"json
```json
{
	"f": "",
	"f_1": "",
	"f_2": "",
	"b": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	},
	"b_1": {
		"f": "",
		"f_1": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	},
	"b_2": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	},
	"b_3": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	}
}
```
##### "a1"json
```json
{
	"f": "",
	"f_1": "",
	"b": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	},
	"b_1": {
		"f": "",
		"f_1": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	},
	"b_3": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	}
}
```
##### "a2"json
```json
{
	"f": "",
	"f_2": "",
	"b": {
		"f": "",
		"f_1": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	},
	"b_1": {
		"f": "",
		"f_1": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_1": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	},
	"b_2": {
		"f": "",
		"f_2": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	},
	"b_3": {
		"f": "",
		"c": {
			"f": "",
			"f_1": "",
			"f_2": ""
		},
		"c_3": {
			"f": "",
			"f_1": ""
		}
	}
}
```