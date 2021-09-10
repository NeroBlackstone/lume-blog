---
title: Audio Signal Processing
description: coursera course notes
date: 2021-09-09
img: https://res.cloudinary.com/neroblackstone/image/upload/v1631177665/download_clgg6l.jpg
tags: []

---
## What is audio signal processing?

* Applications: storage, data compression, effects and transformations, synthesis, description.
* Intentional alteration of sound

## Some basic mathematics

### Sinusoidal functions (正弦函数)

`x [n]=Acos(ωnT +ϕ)=Acos(2 π f nT +ϕ)`

- A:amplitude (振幅)
- ω: angular frequency in radians/seconds
- f =ω/2 π:frequency in Hertz (cycles/seconds)
- ϕ:initial phase in radians
- n: time index
- T =1/ fs:sampling period in seconds(t=nT =n/ fs)