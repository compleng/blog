---
layout: post
title: Protostar Stack 0 Exploit Writeup
date: 2020-09-15 11:37:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: protostar-stack-0.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Exploit,Writeup]
---
Merhaba, ilk yazımda Protostar makinesinde yer alan Stack0 binary’si için writeup yazmaya karar verdim. Protostar makinesini kurduktan sonra user: root ve password: godmode bilgileriyle giriş yapıp /opt/protostar/bin klasöründen binary’lere ulaşabilirsiniz. Stack0 binary’sini indirdikten sonra ilk olarak binary’i çalıştırıp çıktısına baktım.

![Resim1]({{site.baseurl}}/assets/img/ss1.png)

Ardından IDA’da binary’i incelediğimde gets() fonksiyonunun kullanıldığını ardından v5 değişkeni 0 dışında bir değerse “you have changed the 'modified' variable” , 0 ise de “Try again?” basıldığını gördüm. Bizim değişkenin üzerine yazarak ekrana “you have changed the 'modified' variable” yazısını basmamız gerekiyor. gets() fonksiyonunun kullanılması Buffer Overflow zafiyetine neden olduğu için de bu zafiyeti kullanacağım.

![Resim2]({{site.baseurl}}/assets/img/ss2.png)

Checksec ile binary’i kontrol ettiğimde hiçbir güvenlik önleminin açık olmadığını gördüm.

![Resim3]({{site.baseurl}}/assets/img/ss3.png)

Binary’i gdb’de incelemeye başladım.

![Resim4]({{site.baseurl}}/assets/img/ss4.png)

Gdb stack0 komutu ile gdb’nin içinde stack0 binary’sini açtım. b main komutu ile programın main foksiyonunda durmasını istedim. r komutu ile de programı çalıştırdım. 

![Resim5]({{site.baseurl}}/assets/img/ss5.png)

Disas main komutu ile main fonksiyonunun içeriğini ekrana bastırdım. Call 0x804830c instruction’ı gets ile kullanıcıdan input almamızı sağlıyor. Benim de büyük boyutta bir input vererek IDA’da gördüğümüz değişkenin üzerine yazmam gerekiyor. Bunun için pattern create 100 diyerek 100 karakterli bir input oluşturup bunu kopyalıyorum.

![Resim6]({{site.baseurl}}/assets/img/ss6.png)

Ardından ni komutunu kullanarak call gets@plt’ye kadar geliyorum ve kopyaladığım input’u veriyorum. Yine ni komutuyla test eax, eax instruction’ına  geliyorum. Burada $eax’de qaaa değerinin olduğunu görüyorum.

![Resim7]({{site.baseurl}}/assets/img/ss7.png)

![Resim8]({{site.baseurl}}/assets/img/ss8.png)

Bu değerin olması, bu değerden sonra değişkenin üzerine yazabileceğimiz anlamına geliyor. O yüzden kaçıncı karakterde bu değerin olduğunu bulmak için pattern search qaaa yazıyorum.

Buradan 61. veya 64. karakterde değişkene yazabileceğimi görüyorum. İkisini de denediğimde doğru cevabın 64 olduğunu görüyorum. Ardından  “you have changed the 'modified' variable” stringinin atandığı yeri bulmam gerekiyor. Instruction’ları incelediğimde 0x8048419’da atama işleminin yapıldığını görüyorum.

![Resim9]({{site.baseurl}}/assets/img/ss10.png)

Yani 64 tane junk data girip ardından  0x8048419 adresini vermem gerekiyor. 
Daha sonra bu bilgiyle exploitimi yazıyorum.

{% highlight python %}
from pwn import *
p=process("./stack0")
payload = "A"*64 + p32(0x8048419) 
p.sendline(payload)
p.interactive()
{% endhighlight %}

Exploiti yazıp çalıştırdıktan sonra ise “you have changed the 'modified' variable” stringinin ekrana basıldığını görüyorum. Böylece amacıma ulaşmış oluyorum.

![Resim10]({{site.baseurl}}/assets/img/ss9.png)

