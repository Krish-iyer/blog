---
layout: post
title:  "GSoC'19 dav1d ARM NEON Optimization final evaluation report"
date:   2019-08-19 12:32:45 +0100
categories: [dav1d]
---
So here we come to an end of my GSoC'19 project under VideoLAN, dav1d ARM NEON Optimization. The project dealt with implementing functions in ARM assembly using SIMD architecture for both 32 and 64 bit processors. The target development board was odroid N-2 which has quad A-73 and dual A-53 cluster. I had installed ubuntu, accessed counters for benchmarking and a cross-compling tweak to cross compile 32 bit code on 64 bit OS.

So now I was supposed to compile dav1d code, read the C impmentation of functions and implement them in ARM NEON assembly. We started first with blend/blend_v/blend_h functions. AFAIK, I think these functions belongs to motion compensation and hence located in the file
```
src/mc_tmpl.c
```
corresponding test function is located in 
```
tests/checkasm/mc.c
```
checkasm is basically a tool which checks the functions output with the corresponding input bitstream for both c and asm function.
Now I need to implement an asm function in if 32 bit architecture
```
src/arm/32/mc.s 
```
or if 64 bit architecture
```
src/arm/64/mc.s 
```
Before even implementing the function, we need to hook the asm function because till now it was using C implementation and with availability of asm function we need it to priortize asm function over C. So we need to declare function and hook it to the object in
```
src/arm/mc_init_tmpl.c
```
for example 
```
decl_xyz_fn(dav1d_xyz_8bpc_neon);   # delcaring the function
c->xyz = dav1d_xyz_8bpc_neon;       # hooking to the c object
```
Now if your function is only supported for 32bit arch then you need add under a specific macro like
```
#if ARCH_ARM
    c->xyz = dav1d_xyz_8bpc_neon;
#endif
```
Now we are ready to implement our assembly function. Definitition of function should be under something like
```
function xyz_8bpc_neon, export=1
```
which will export the function in the format of  **dav1d_xyz_8bpc_neon**. 

So understand SIMD if you haven't already about it or didn't go through my previous blogs. The adavntage over common assembly is we are going to fetch data in keep it in registers and do operation in a single instruction.

So for example you have two 128bit register, you can either accomodation 16 8bit or 8 16bit or 4 32 bit data on either of the registers. For a given case lets consider we accomodated 4 32bit numbers on reg-A and another 4 32 bit numbers on reg-B, now you can add 4 32bit numbers in a single instruction and store it in another 128bit register. If you have to do this in common assembly language. You will fetch single 32 bit number and add it and then store, you will repeat this for 4 times.

So specific for widths of the data or the data about which we are sure that they would be contiguously arranged in the memory like array, we have different implementations it helps us to reduce of extra reg unncessarily and optimize it more to gain the similar functionality over fewer instruction cycles. Further to elaborate, one can imagine it like a matrix row x columns, only difference is you are not sure it rows are contiguously arranged in the memory i.e last element of any row shares the same border with starting element of the next row but in a row elements are arrange contiguously. So we have cases for different columns or row elements.



