---
layout: post
title:  "การพัฒนาเว็บยุคใหม่ ภาค 2 - Multiple Entry Points"
date:   2018-03-12 19:51:35 +0700
categories: programming
---
บทความนี้เป็นภาคต่อของ [การพัฒนาเว็บยุคใหม่ ภาค 1 - Webpack]({{ site.baseurl }}{% post_url 2018-03-06-modern-web-development-part-1 %}) ซึ่งเราจะพูดถึงการ Transpile JavaScript มากกว่า
1 ไฟล์

จากบทความภาคที่แล้ว เราได้ลอง Transpile JavaScript แค่ไฟล์เดียว ซึ่งเวลาใช้งานจริง ใน 1 เว็บไซต์ไม่ได้มี JavaScript แค่ไฟล์เดียวแน่นอน (ยกเว้นว่าจะเป็นเว็บเล็ก ๆ)

# Entry Point คืออะไร

Entry Point คือไฟล์ JavaScript หลักที่ Webpack จะเริ่มต้นประมวลผล ซึ่งก็คือไฟล์ JavaScript ที่เราสั่งให้ Webpack ทำการ Transpile ในบทความที่แล้ว ปรกติแล้วเว็บในแต่ละหน้าหากมีการใช้ JavaScript มักจะมีอยู่
3 ไฟล์ คือ

1. ไฟล์แรกสำหรับ Logic เฉพาะหน้านั้น ๆ
2. ไฟล์สองสำหรับ Layout/Template ของหน้านั้น ๆ
3. ไฟล์สามสำหรับแชร์ Code ที่ใช้บ่อยมากกว่าหนึ่งหน้า

หากเราต้องการมากกว่า 1 Entry Point เรามีอยู่ 2 ทางเลือก คือ

1. รัน Webpack หนึ่งครั้งต่อหนึ่ง Entry Point
2. เขียน Script สำหรับ List รายการ Entry Point ทั้งหมดแล้วส่งเข้า Webpack

ปัญหาของวิธีแรกคือ ไม่ยืดหยุ่น เนื่องจากเราต้องมากำหนดชื่อไฟล์แต่ละไฟล์เอง ดั่งนั้น วิธีที่สองเป็นทางเลือกที่ดีที่สุด

# สร้าง Script สำหรับรัน Webpack

จริง ๆ แล้ว Webpack ไม่ได้มีแค่คำสั่งที่เราเคยใช้ก่อนหน้านั้น แต่ Webpack มี Module ของ Node.js ให้เราเรียกใช้งานได้ด้วย เราจะใช้ Module ตัวนี้เพื่อเขียน Script สำหรับ List รายการ Entry Points ทั้งหมดที่เรามี
แล้วส่งให้ Webpack

ก่อนอื่น เราต้องออกแบบโครงสร้างของแฟ้มใน Project ก่อนเพื่อให้ง่ายต่อการจัดการ ตัวอย่างโครงสร้างที่ผมใช้

~~~
.
|-- ...
|-- assets
|   |-- src
|   |   |-- entry1.js
|   |   |-- entry2.js
|   |   `-- ...
|   `-- webpack.base.conf.js
|-- ...
|-- wwwroot
|   `-- assets
|       `-- js
|           |-- entry1.js
|           |-- entry2.js
|           `-- ....
`-- package.json
~~~

แฟ้ม `wwwroot/assets` คือแฟ้มที่เราจะเก็บ Output ของ Webpack ด้วยโครงสร้างนี้ เราสามารถ List รายการ Entry Point ทั้งหมดจากแฟ้ม `assets/src` แค่แฟ้มเดียวได้เลย และยังสามารถลบแฟ้ม `wwwroot/assets`
ทั้งแฟ้มก่อนที่จะรัน Webpack เพื่อลบไฟล์ทั้งหมดจากการรันครั้งที่แล้วได้ด้วย

เสร็จแล้ว ให้สร้างไฟล์ JavaScript อันใหม่ขึ้นมา ไฟล์นี้จะเป็นตัวเก็บ Config ต่าง ๆ ของ Script ที่เรากำลังจะสร้าง ซึ่งควรจะอยู่ที่เดียวกันกับ Script นั้น ตัวอย่าง

{% highlight js %}
'use strict'

const path = require('path')

const projectPath = path.dirname(__dirname)
const sourcePath = path.join(__dirname, 'src')
const outputPath = path.join(projectPath, 'wwwroot', 'assets')

module.exports = {
  projectPath,
  sourcePath,
  outputPath
}
{% endhighlight %}

ต่อมา เราต้องแก้ไฟล์ Config ของ Webpack เพื่อกำหนดรายการ Entry Points ตัวอย่าง

{% highlight js linenos %}
'use strict'

const path = require('path')
const glob = require('glob')
const config = require('./config')

function getEntryPoints() {
  return new Promise((resolve, reject) => {
    glob(path.join(config.sourcePath, '*.js'), (err, matches) => {
      if (err) {
        reject(err)
      } else {
        resolve(matches)
      }
    })
  })
}

module.exports = async function () {
  let entryPoints = (await getEntryPoints()).reduce((obj, file) => {
    obj[path.parse(file).name] = file
    return obj
  }, {})

  return {
    entry: entryPoints,
    output: {
      path: config.outputPath,
      filename: path.join('js', '[name].js')
    },
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
}
{% endhighlight %}

บรรทัดที่ 4 เป็นการเรียกใช้ Module [glob](https://www.npmjs.com/package/glob) ซึ่งเราต้องติดตั้งเพิ่ม ส่วนบรรทัดที่ 5 เป็นการเรียกใช้ Config ที่เราพึ่งสร้างเมื่อกี้ (ผมตั้งชื่อว่า `config.js` ซึ่งอยู่ในแฟ้มเดียวกันกับ
Config ของ Webpack)

ขั้นตอนสุดท้าย สร้าง Script สำหรับรัน Webpack ตัวอย่าง

{% highlight js linenos %}
'use strict'

const rimraf = require('rimraf')
const webpack = require('webpack')
const config = require('./config')
const webpackConfig = require('./webpack.base.conf')

rimraf(config.outputPath, async err => {
  if (err) {
    throw err
  }

  webpack(await webpackConfig(), (err, stats) => {
    if (err) {
      throw err
    }

    let result = stats.toString({colors: true})
    console.log(result)
  })
})

{% endhighlight %}

บรรทัดที่ 6 เป็นการเรียกใช้ Config ของ Webpack ที่เราพึ่งแก้ไขไปเมื่อกี้ ส่วนบรรทัดที่ 8 เป็นการสั่งลบแฟ้มที่เก็บ Output ทั้งแฟ้มก่อนการรัน Webpack

การรัน Script ที่เราเขียนก็เหมือนกับการรัน Node.js ทั่วไป ตัวอย่าง

{% highlight sh %}
node assets/build.js
{% endhighlight %}

เช่นเคย เราสามารถใส่คำสั่งนี้ไว้ใน `package.json` เพื่อย่นเวลาการพิมพ์คำสั่งได้

# บทส่งท้าย

พอเราสามารถ Transpile ได้มากกว่า 1 ไฟล์ เราก็สามารถที่จะเริ่มใช้ Webpack อย่างจริงจังใน Project ได้แล้ว ในภาคต่อไปของบทความ จะเป็นการ Import ไฟล์อื่น ๆ เข้ามาในไฟล์ JavaScript เพื่อใช้งานไฟล์นั้น ๆ
ซึ่งไม่จำกัดว่าจะต้องเป็นไฟล์ JavaScript เช่น เราสามารถ Import ไฟล์ PNG หรือ CSS เข้ามาได้
