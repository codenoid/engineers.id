---
title: "Membuat Global KeyLogger Dengan Golang"
description: "Golang Global KeyLogger, keylogger, deteksi pencet keyboard"
date: 2019-07-30T09:13:29+07:00
author: codenoid
categories:
  - "howto"
tags:
  - "tutorial"
  - "keyboard"
  - "keylogger"
  - "howto"
---

![Screenshot](/posts/membuat-global-keylogger-dengan-golang/screenshot.png)

### My Device

1. Linux Ubuntu 14.04

### Cek Device

Pastikan terdapat input keyboard, dapat di cek di `ls /dev/input/event*`, entah itu keyboard bluetooth maupun kabel, yang penting bisa di pakai

### Dependency

Jalankan `go get -u github.com/MarinX/keylogger`

### Yang Akan Kita Lakukan

1. Basic Key Logging
2. Kombinasi Key pada Keyboard, seperti CTRL+V, dsj

### Pembahasan Dan Code

```
package main

import (
	"fmt"
	"strings"

	"github.com/MarinX/keylogger"
)

func main() {
	keyboard := keylogger.FindKeyboardDevice()
	// check if we found a path to keyboard
	if len(keyboard) <= 0 {
		fmt.Println("No keyboard found...you will need to provide manual input path")
		return
	}

	// yang digunakan biasanya built-in keyboard (jika laptop)
	fmt.Println("Found a keyboard at", keyboard)
	// init keylogger with keyboard
	k, err := keylogger.New(keyboard)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer k.Close()

	// stream global key logger
	events := k.Read()

	// prev akan kita gunakan untuk membantu melakukan
	// emulasi key combination, seperti CTRL+V
	prev := ""

	// range of events
	for e := range events {
		switch e.Type {
		// EvKey digunakan untuk mendeskripsikan aksi yang dilakukan oleh keyboards, buttons, dsj.
		// lihat https://github.com/MarinX/keylogger/blob/master/input_event.go untuk informasi lebih lanjut
		case keylogger.EvKey:

			// jika keyboard di tekan
			if e.KeyPress() {

				// untuk combination key, rubah semua menjadi UPPERCASE
				// anda tetap mendapatkan original input dari e.KeyString()
				pressed := strings.ToUpper(e.KeyString())

				// jika previous button masih kosong, atau belum di pencet
				// set previous button dengan yang saat ini sedang di tekan
				if prev == "" {
					prev = pressed
				}

				// jika previous keyboard adalah L_CTRL
				// makan masuk ke level selanjut nya
				if prev == "L_CTRL" {
					// jika user melakukan kombinasi
					// CTRL+V makan akan melakukan print PASTE !
					if pressed == "V" {
						fmt.Println("PASTE !")
					}
				}

				// jadikan key sekarang sebagai previous press di bagian akhir
				prev = e.KeyString()
				fmt.Println("[event] press key ", e.KeyString())
			}

			// jika tombol keyboard di lepas (setelah di tekan)
			if e.KeyRelease() {
				fmt.Println("[event] release key ", e.KeyString())
			}

			break
		}
	}
}

```

### EXPANSI

Yang saya pikirkan, Anda dapat menggunakan ini untuk:

1. Shortcut
2. Screen Lock seperti [xtrlock](http://manpages.ubuntu.com/manpages/xenial/man1/xtrlock.1x.html)
3. Sniffing Key
4. Membuat aplikasi seperti teamviewer, mengirimkan keypress nya
5. masih banyak lagi

### MARI BERKONTRIBUSI !

Mari ikut menulis di website ini dengan mengirimkan pull request ke [https://github.com/codenoid/engineers.id](https://github.com/codenoid/engineers.id)