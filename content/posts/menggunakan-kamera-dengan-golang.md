---
title: "Menggunakan Kamera Dengan Golang"
description: "Golang Camera, Mengambil foto dengan golang, mengambil video dengan golang"
date: 2019-07-30T00:15:29+07:00
author: codenoid
categories:
  - "howto"
tags:
  - "tutorial"
  - "kamera"
  - "video"
---

![Gopher](/posts/menggunakan-kamera-dengan-golang/main.jpg)

### Persiapan

1. Memiliki Kamera (internal/external)

### Cek, apakah kamera sudah terdeteksi

cek, apakah kamera sudah terdeteksi dengan `ls /dev/video*`, coba cabut dan colok lagi sembari cek, apakah bertambah

saya menggunakan kamera external pada laptop yang sudah terdapat kamera, dan hasil ls saya : 

```
# ketika kamera external di colok
$ ls /dev/video*
/dev/video0  /dev/video1  /dev/video2  /dev/video3

# ketika kamera external di cabut
$ ls /dev/video*
/dev/video0  /dev/video1
```

nah, yang saya gunakan disini `/dev/video2` (kamera external saya)

### Dependency

Jalankan perintah `go get -u github.com/blackjack/webcam`

### Yang Akan Di Lakukan

Dimulai dari hal yang sangat basic, yakni melakukan pengambilan gambar / foto

### Kode dan Pembahasan

```
package main

import (
	"os"
	"bytes"
	"fmt"
	"image"
	"image/jpeg"
	"io/ioutil"

	"github.com/blackjack/webcam"
)

func main() {

	// pilih device kamera yang akan kamu gunakan
	cam, err := webcam.Open("/dev/video2")
	if err != nil {
		panic(err.Error())
	}
	defer cam.Close()

	// format gambar kamu, dukungan format dapat di cek di
	// cat /usr/include/linux/videodev2.h
	// atau get it from cam.GetSupportedFormats()
	// $ cat /usr/include/linux/videodev2.h | grep jpeg
	// struct v4l2_jpegcompression {
	//	    __u32 jpeg_markers;     /* Which markers should go into the JPEG
	// #define VIDIOC_G_JPEGCOMP        _IOR('V', 61, struct v4l2_jpegcompression)
	// #define VIDIOC_S_JPEGCOMP        _IOW('V', 62, struct v4l2_jpegcompression)
	// w = 800; h = 600;
	cam.SetImageFormat(62, 800, 600)

	fmap := cam.GetSupportedFormats()

	// coba print hasil fmap, jika pemilihan device salah
	// makan fmap akan kosong
	if len(fmap) == 0 {
		fmt.Println("empty camera format")
		return
	}

	// mulai streaming byte frame dari /dev/video2
	err = cam.StartStreaming()
	if err != nil {
		panic(err.Error())
	}

	// jika frame tidak update dalam 10 detik
	// maka akan error
	err = cam.WaitForFrame(10)

	switch err.(type) {
	case nil:
	case *webcam.Timeout:
		fmt.Fprint(os.Stderr, err.Error())
	default:
		panic(err.Error())
	}

	// dapatkan frame image dari /dev/video2
	frame, err := cam.ReadFrame()
	if len(frame) != 0 {
		// convert frame menjadi image
		// int(800), int(600) harus sama seperti definisi
		// SetImageFormat di atas
		var img image.Image
		
		// rubah YUV menjadi RGB
		yuyv := image.NewYCbCr(image.Rect(0, 0, int(800), int(600)), image.YCbCrSubsampleRatio422)
		for i := range yuyv.Cb {
			// https://github.com/blackjack/webcam/issues/2#issuecomment-141982187
			ii := i * 4
			yuyv.Y[i*2] = frame[ii]
			yuyv.Y[i*2+1] = frame[ii+2]
			yuyv.Cb[i] = frame[ii+1]
			yuyv.Cr[i] = frame[ii+3]
		}
		img = yuyv

		// rubah format image menjadi jpeg 
		// seperti yang sudah di definisikan diatas
		buf := &bytes.Buffer{}
		if err := jpeg.Encode(buf, img, nil); err != nil {
			fmt.Println(err)
			return
		}
		// tulis/save menjadi file
		ioutil.WriteFile("./test.jpg", buf.Bytes(), 0644)
	} else if err != nil {
		panic(err.Error())
	}
}
```

mulai dari `WaitForFrame` bisa kamu taruh dalam infinity loop untuk melakukan streaming-video

## THAT'S IT

selanjutnya kita menggabungkan kode diatas dengan computer vision (detect face, tulisan) tau deh nanti, enak nya gimana, see you

jangan takut buat rubah rubah code !, biar tau !!!