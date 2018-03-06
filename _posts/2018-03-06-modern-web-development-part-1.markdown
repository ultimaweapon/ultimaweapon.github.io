---
layout: post
title:  "การพัฒนาเว็บยุคใหม่ ภาค 1 - Webpack"
date:   2018-03-06 19:21:35 +0700
categories: programming
---
# เนื้อหาบล็อกนี้เหมาะสำหรับใคร
- คนที่ไม่เคยใช้ Webpack และอยากก้าวทันเทคโนโลยีการพัฒนาเว็บไซต์ยุคใหม่ **เนื้อหาในบล็อกนี้จะ Focus เฉพาะฝั่ง Front-end เท่านั้น (HTML, CSS, JavaScript) ไม่มีเนื้อหาเกี่ยวกับฝั่ง Server-side เช่น
  ASP.NET MVC, Ruby on Rails, Laravel หรือ Django และอื่น ๆ**

# พื้นฐานที่ต้องมี
- HTML
- CSS
- JavaScript
- ภาษา Server-side หนึ่งภาษา (PHP/Java/Python/Ruby/C# บลา ๆ)

ทุกข้อไม่จำเป็นต้องระดับ Advanced แค่ใช้งานเป็นก็เพียงพอแล้ว

# เครื่องมือที่จำเป็น
- Node.js
- npm

# Webpack คืออะไร
Webpack เป็น Module ของ Node.js ตัวหนึ่งที่เอาไว้ Transpile JavaScript ให้ออกมาเป็น JavaScript (ในความเป็นจริงแล้ว Webpack ทำได้มากกว่านี้ แต่ช่วงแรกเราจะ Focus แค่นี้ก่อน) การ Transpile
นั้นจะคล้ายกับการ Compile แต่ต่างกันตรงที่การ Transpile เป็นการแปลง Source code ของภาษาหนึ่งให้ออกมาเป็นอีกภาษาหนึ่ง อ่านถึงตรงนี้บางคนอาจจะเริ่มงงแล้วว่าทำไมต้อง Transpile JavaScript ให้ออกมาเป็น
JavaScript ด้วย อย่างที่เรารู้กันดีว่าการรองรับ Features ใหม่ ๆ ของ JavaScript ในแต่ละ Web browser นั้นจะไม่เท่ากัน ทำให้เราโดนจำกัดการใช้งาน Feature ของ JavaScript ไว้ที่ Browser ต่ำสุดที่เราต้องการ
Supports ซึ่งปัญหานี้สามารถแก้ได้ด้วย Webpack โดยเราสามารถเขียน JavaScript Version ล่าสุดได้โดยที่ไม่ต้องกังวลการรองรับของฝั่ง Browser แล้วใช้ Webpack Transpile JavaScript ที่เราเขียนให้ออกมาเป็น
JavaScript ที่ Browser ทุกตัวสามารถเข้าใจได้

# Node.js เกี่ยวอะไรด้วย
อย่างที่อธิบายไปเมื้อกี้ว่า Webpack เป็น Module ตัวหนึ่งของ Node.js ซึ่งเราจะใช้ Node.js แค่ตอนพัฒนาเท่านั้น ไม่ได้ใช้บน Production

# ติดตั้ง Webpack
วิธีการติดตั้ง Webpack ที่ดีที่สุดคือติดตั้งเป็น Dependency ของ Project นั่นหมายความว่าเราต้องทำให้ Project เรากลายเป็น Module ของ Node.js เสียก่อน อาจจะฟังดูน่ากลัว แต่ไม่ยากและไม่ได้สร้างผลกระทบกับ
Project ขนาดนั้น ข้อดีของวิธีนี้คือแต่ละ Project จะสามารถใช้ Webpack Version ที่ไม่เหมือนกันได้ และคนในทีมทุกคนจะใช้ Webpack Version เดียวกันสำหรับ Project นั้น ๆ ทำให้ไม่มีปัญหาแปลก ๆ เช่น นาย A
รัน Webpack ไม่ผ่านเพราะนาย B Commit Code ที่เรียกใช้ Feature ใหม่ของ Webpack ที่นาย A ไม่มี

การทำให้ Project เป็น Module ของ Node.js นั้น ให้สร้างไฟล์ `package.json` ไว้ที่ Root ของ Project โดยมีข้อมูลดังนี้

{% highlight json %}
{
  "name": "project-name",
  "version": "1.0.0",
  "private": true
}
{% endhighlight %}

`name` คือชื่อของ Project นี้ ควรจะเป็นตัวเล็กทั้งหมด และใช้ `-` แทนการวรรค ส่วน `version` คือ Version ของ Project ต้องเป็นรูปแบบ `x.y.z` เท่านั้น โดย `x` คือ Major Version ส่วน `y` คือ
Minor Version และ `z` คือ Patch รายละเอียดสามารถอ่านได้จาก [ที่นี่](https://semver.org/#summary)

`private` เป็นการบอกให้ `npm` ปฏิเสธการ Upload ขึ้น [https://www.npmjs.com](https://www.npmjs.com/) เอาไว้ป้องกันเราเผลอ Publish Project เราออกสู่ภายนอก
