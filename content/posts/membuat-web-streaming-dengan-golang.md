---
title: "Membuat Live Web Streaming Dengan Golang"
description: "Membuat Live Streaming dengan golang, web streaming, video streaming, live streaming"
date: 2019-07-30T09:13:29+07:00
author: codenoid
categories:
  - "howto"
tags:
  - "tutorial"
  - "video"
  - "camera"
  - "howto"
---

![Reporter TV](/posts/membuat-web-streaming-dengan-golang/reporter-tv.jpg)

### My Device

1. Linux Ubuntu 14.04

### Perkenalan (WAJIB BACA)

Melanjutkan [post saya yang kemaren](/posts/menggunakan-kamera-dengan-golang) (**JIKA BELUM MELIHAT, WAJIB MELIHAT ARTIKEL NYA**), saat ini mari kita expansi dengan membuat web streaming atau live streaming dengan menggunakan Golang dan kamera kita, di harapkan dengan ini bisa kalian expansi untuk membuat CCTV, LIVE-TV, DIBERI AUGMENTED REALITY (IMAGE PROCESSING), DSJ

### Pembahasan Dan Code

```
package main

import (
	"bytes"
	"fmt"
	"image"
	"image/jpeg"
	"log"
	"mime/multipart"
	"net/http"
	"net/textproto"
	"os"
	"strconv"

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

	var (
		// initialisasi channel li untuk menyimpan frame dari kamera kita
		// channel li juga akan di pakai oleh httpVideo
		li chan *bytes.Buffer = make(chan *bytes.Buffer)
	)

	// aktifkan route http untuk streaming
	go httpVideo(li)

	for {
		err = cam.WaitForFrame(10)

		switch err.(type) {
		case nil:
		case *webcam.Timeout:
			fmt.Fprint(os.Stderr, err.Error())
		default:
			panic(err.Error())
		}

		frame, err := cam.ReadFrame()
		if len(frame) != 0 {
			// Process frame

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

			li <- buf
		} else if err != nil {
			panic(err.Error())
		}
	}
}

func httpVideo(li chan *bytes.Buffer) {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		log.Println("connect from", r.RemoteAddr, r.URL)
		if r.URL.Path != "/" {
			http.NotFound(w, r)
			return
		}

		// remove gambar yang lama dari channel didalam httpVideo
		<-li
		const boundary = `frame`
		// https://blog.dubbelboer.com/2012/01/08/x-mixed-replace.html
		// Using this special Content-type you can replace the contents of a page
		w.Header().Set("Content-Type", `multipart/x-mixed-replace;boundary=`+boundary)
		multipartWriter := multipart.NewWriter(w)
		multipartWriter.SetBoundary(boundary)

		// lakukan for untuk melakukan update data untuk browser
		for {
			img := <-li
			image := img.Bytes()
			iw, err := multipartWriter.CreatePart(textproto.MIMEHeader{
				"Content-type":   []string{"image/jpeg"},
				"Content-length": []string{strconv.Itoa(len(image))},
			})
			if err != nil {
				log.Println(err)
				return
			}
			_, err = iw.Write(image)
			if err != nil {
				log.Println(err)
				return
			}
		}
	})

	log.Fatal(http.ListenAndServe(":3030", nil))
}
```

### CATATAN

1. Code ini hanya melakukan streaming dengan gambar, alias tanpa audio atau tanpa suara

### MARI BERKONTRIBUSI !

Mari ikut menulis di website ini dengan mengirimkan pull request ke [https://github.com/codenoid/engineers.id](https://github.com/codenoid/engineers.id)