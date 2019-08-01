---
title: "Komparasi Syntax Antara PHP Dengan Golang"
description: "Komparasi sintax antara php dan golang, pindah dari php ke golang"
date: 2019-08-01T00:13:29+07:00
author: codenoid
categories:
  - "programming-language"
tags:
  - "proglang"
  - "php"
  - "golang"
---

![Gopher vs PHP](/posts/komparasi-syntax-antara-php-dengan-golang/gopher-php.png)

Menggunakan OS Ubuntu, disini aq akan melakukan komparasi syntax antara golang dengan php

### Installasi

Golang : 

```
cd ~
wget https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz
tar -xf go1.12.7.linux-amd64.tar.gz

export GOROOT="/home/$USER/go"
export GOPATH="/home/$USER/go/packages"
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
# puts export line to your ~/.bashrc or any bashprofile file
# please change $USER with your static username value
```
PHP : 

```
sudo apt install php7.2-cli / *
```

### Hello World !

Golang : 

```
package main

import "fmt"

func main() {
	fmt.Println("Hello World!")
}
```

PHP : 

```
<?php

echo "Hello World!";
```

### Eksekusi Program

Golang : 

```
go run main.go # langsung run tanpa build executable file
go build main.go # build executable ./main file
```

PHP : 

```
php filename.php
php -S 0.0.0.0:2002 # dukungan web server
```

### Variable, Mutability and Typing System

Golang : 

```
package main

import (
	"fmt"
)

func main() {
	// var variable = "henlo"
	variable := "Henlo" # atau definisi dengan :=
	variable = "hello"
	variable = 2 # cause error, because initially defined as string, positif untuk data consistency
	fmt.Println(variable)
}
```

PHP : 

```
<?php
$variable = "henlo";
$variable = "hello";
$variable = 2;

echo $variable;
```

### If, Else, Else If

Golang : 

```
if umur >= 18 {
    fmt.Println("udah legal buat nonton")
} else if umur < 14 {
    fmt.Println("jangan megang hp dulu")
} else {
    fmt.Println("sabar ya")
}
```

PHP : 

```
if ($umur >= 18) {
    echo "udah legal buat nonton";
} elseif ($umur < 14) {
    echo "jangan megang hp dulu";
} else {
    echo "sabar ya";
} 
```

### Operator

Golang : 

```
package main

import (
	"fmt"
)

func main() {
	MyAge := 18 + 2 - 2
	fmt.Println(MyAge)
}
```

PHP : 

```
<?php

$MyAge = 18 + 2 - 2;
echo $MyAge;
```

### Array &/ Slice

Golang : 

```
package main

import (
	"fmt"
)

func main() {
	letters := []string{"a", "b", "c", "d"}
	fmt.Println(letters)
}
```

PHP : 

```
<?php

$letters = array("a", "b", "c", "d");
echo var_dump($letters);
```

### Iterate Array &/ Slice

Golang : 

```
package main

import (
	"fmt"
)

func main() {
	letters := []string{"a", "b", "c", "d"}
	for letter := range letters {
		fmt.Println(letter)
	}
}
```

PHP : 

```
<?php

$letters = array("a", "b", "c", "d");
foreach($letters as $letter) {
  echo $letter;
}
```