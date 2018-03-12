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

ขั้นตอนต่อไปคือการติดตั้ง Webpack ให้เปิด Terminal ขึ้นมา เข้าไปยังแฟ้มของ Project แล้วรัน

{% highlight sh %}
npm install webpack webpack-cli --save-dev
{% endhighlight %}

`--save-dev` เป็นการบอกว่า `webpack` และ `webpack-cli` เป็นแค่ Module ที่ใช้ตอนพัฒนาเท่านั้น ไม่ได้ Deploy ไปกับเว็บ

หลังจากติดตั้งเสร็จแล้ว จะพบว่า

- ไฟล์ `package.json` มี Property ใหม่เพิ่มขึ้นมาชื่อ `devDependencies` ซึ่งมีข้อมูล Version ของแต่ละ Dependency ที่ Project ต้องการ หากตอนติดตั้งเราไม่ได้ระบุ Version ตัว `npm` จะเลือก
  Version ล่าสุดให้อัตโนมัติ สำหรับ Format ของ Version หากต้องการรู้รายละเอียดเพิ่มเติมสามารถอ่านได้จาก [ที่นี่](https://docs.npmjs.com/misc/semver)
- มีไฟล์ `package-lock.json` เพิ่มขึ้นมา ซึ่งไฟล์นี้เป็นตัวบอก Version ของแต่ละ Module ที่จะใช้งานจริง เหตุผลที่ต้องมีไฟล์นี้เนื่องจาก Version ของ Dependency ที่ระบุใน `package.json`
  นั้นสามารถกำหนดแบบคร่าว ๆ ได้ กล่าวคื่อไม่จำเป็นต้องกำหนดแบบเจาะจงว่าต้องเป็น Version นั้นเท่านั้น เช่น `^4.1.0` หมายถึง Version ไหนก็ได้ที่ไม่น้อยกว่า `4.1.0` และไม่ใช่ `5.X.X` หรือมากกว่า
  หากเราใช้ Version control เราควรจะ Commit ไฟล์นี้ขึ้นไปด้วยเพื่อให้ทุกคนในทีมใช้ Modules ที่เป็น Version เดียวกันทั้งหมด
- มีแฟ้ม `node_modules` เพิ่มขึ้นมา ซึ่งแฟ้มนี้จะเก็บ Dependencies ทั้งหมดของ Project

# ทดลองรัน Webpack
ให้สร้างไฟล์ JavaScript ง่าย ๆ ขึ้นมาหนึ่งไฟล์ เช่น

{% highlight js %}
window.alert('Hello, world!');
{% endhighlight %}

เสร็จแล้วรัน `./node_modules/.bin/webpack INPUT --output OUTPUT` ตัวอย่าง

{% highlight sh %}
./node_modules/.bin/webpack assets/src/app.js --output wwwroot/assets/js/app.js
{% endhighlight %}

หากไม่มี Error อะไร ไฟล์ `INPUT` จะถูก Minified ออกไปที่ `OUTPUT` (ยังไม่มีการ Transpile ซึ่งจะอยู่ในหัวข้อถัดไป) หากเราเปิดดูไฟล์ `OUTPUT` จะพบว่ามี Code บางส่วนเพิ่มเข้ามา ซึ่งเป็น Code ที่ Webpack
เอาไว้รองรับ Feature ของตัวมันเอง ซึ่งตอนนี้เรายังไม่ได้ใช้ เราสามารถเรียกใช้ไฟล์ `OUTPUT` ในไฟล์ HTML ได้ทันที แต่ Browser ต้องรองรับ Code ที่เราเขียนด้วย เพราะยังไม่มีการ Transpile เกิดขึ้น

# ทดลอง Transpile Class ของ ES6 ให้ออกมาเป็น Code ที่ IE 11 ใช้งานได้
ตัว Webpack นั้นไม่สามารถ Transpile Code ได้ด้วยตัวมันเอง เราต้องใช้ Feature หนึ่งของ Webpack ที่ชื่อว่า Loaders ซึ่ง Loader แต่ละตัวนั้นเราต้องติดตั้งเพิ่ม Loader ที่เราจะใช้ในการ Transpile JavaScript
Version ใหม่ ๆ ให้ออกมาเป็น Version ที่ IE 11 ใช้งานได้คือ [babel-loader](https://webpack.js.org/loaders/babel-loader/) ซึ่งตัวมันเป็นแค่ Wrapper เพื่อเรียกใช้งาน [Babel](https://babeljs.io/)
อีกต่อหนึ่ง

วิธีการติดตั้ง `babel-loader` ให้รัน

{% highlight sh %}
npm install babel-loader babel-core babel-preset-env --save-dev
{% endhighlight %}

เสร็จแล้ว เราต้องสร้าง [config](https://webpack.js.org/configuration/) เพื่อบอก Webpack ว่าให้ประมวลผลไฟล์ JavaScript ด้วย `babel-loader`

การสร้าง config ของ Webpack เพื่อใช้ `babel-loader` ให้เราสร้างไฟล์ JavaScript ขึ้นมา แล้วใส่ Code ข้างล่างลงไป

{% highlight js %}
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['babel-preset-env']
          }
        }
      }
    ]
  }
}
{% endhighlight %}

ซึ่งไฟล์ JavaScript ตัวนี้จะถูกรันใน Node.js นั่นหมายความว่า เราสามารถเรียกใช้ Features ต่าง ๆ ของ Node.js ได้

เสร็จแล้วเราต้องสร้าง config ของ Babel เพื่อบอกว่าเราต้องการ Code ที่ใช้งานกับ Browser ไหนได้บ้าง ให้สร้างไฟล์ `.babelrc` ไว้ที่ Root ของ Project แล้วใส่ JSON ข้างล่างลงไป

{% highlight json %}
{
  "presets": [
    ["env", {
      "targets": {
        "browsers": ["last 2 versions", "not ie <= 10"]
      }
    }]
  ]
}
{% endhighlight %}

`last 2 versions` หมายถึง Browser ทุกตัวย้อนไป 2 Versions ส่วน `not ie <= 10` หมายถึง ตัด IE ตั้งแต่ Version 10 ลงมาออกไป รายละเอียด Syntax สามารถดูได้จาก [ที่นี่](https://github.com/ai/browserslist)

จากนั้น ให้แก้ใขไฟล์ JavaScript ที่เราลองเมื่อกี้ให้เป็น Code นี้แทน

{% highlight js %}
class Foo {
  constructor(msg) {
    this.msg = msg
  }

  alert() {
    window.alert(this.msg)
  }
}

let foo = new Foo('Hello, world!')
foo.alert()
{% endhighlight %}

เสร็จแล้ว รัน Webpack ด้วย config ที่เราสร้างเมื่อกี้

{% highlight sh %}
./node_modules/.bin/webpack assets/src/app.js --config assets/webpack.base.conf.js --output wwwroot/assets/js/app.js
{% endhighlight %}

เมื่อเราเปิดดู Output จะพบว่า Class ที่เราเขียนถูกแปลงเป็น Code อีกรูปแบบหนึ่งที่สามรถใช้งานใน Browser ที่เรากำหนดใน `.babelrc`

# ย่นเวลาการพิมพ์คำสั่งเพื่อรัน Webpack
เราสามารถตั้งค่าให้ `npm` รันคำสั่งที่เรากำหนดได้เมื่อเรารัน `npm run xyz` ซึ่ง `xyz` ได้ คือชื่อคำสั่งที่เรากำหนดใน `package.json`

ตัวอย่างการรันคำสั่ง Webpack ข้างบนเมื่อเรารัน `npm run webpack`

{% highlight json %}
{
  "name": "modern-web-development",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "webpack": "webpack assets/src/app.js --config assets/webpack.base.conf.js --output wwwroot/assets/js/app.js"
  },
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.4",
    "babel-preset-env": "^1.6.1",
    "webpack": "^4.1.0",
    "webpack-cli": "^2.0.10"
  }
}
{% endhighlight %}

สังเกตว่าคำสั่ง `webpack` ไม่ต้องใช้ `./node_modules/.bin/webpack` แล้ว เนื่องจาก `npm` จะเพิ่มพาธ `node_modules/.bin` เข้าไปใน Environment ให้เราอัตโนมัติ

# บทส่งท้าย
จากบทความนี้จะเห็นว่าเราสามารถใช้ Webpack และ Babel เพื่อช่วยให้เราสามารถใช้งาน Version ใหม่ ๆ ของ JavaScript ได้ ซึ่งช่วยเพิ่ม Productivity รวมถึงทำให้ดูแล Code ได้ง่ายขึ้นด้วย เนื่องจาก Version ใหม่ ๆ
มักจะมี Feature ที่ทำให้เราสามารถแยก Code ออกเป็นส่วน ๆ ได้ดีขึ้น และมี Syntax ที่ทำให้เขียน Code ได้สั้นลง Feature ของ Webpack ที่เราใช้ในบทความนี้เป็นเพียงส่วนน้อยเท่านั้น ซึ่งถ้าใช้งาน Features อื่น ๆ เพิ่มเติม
จะทำให้การพัฒนา Front-end ของเว็บไซต์เป็นระบบและง่ายขึ้นมาก ซึ่งจะช่วยให้เราสามารถสร้างเว็บไซต์ที่มีความซับซ้อนสูงได้
